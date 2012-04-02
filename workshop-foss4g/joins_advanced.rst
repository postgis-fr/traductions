.. _joins_advanced:

Partie 19 : Plus de jointures spatiales
=======================================

Dans la partie précédente nous avons vu les fonctions :command:`ST_Centroid(geometry)` et :command:`ST_Union([geometry])` ainsi que quelques exemples simples. Dans cette partie nous réaliseront des choses plus éllaborées.

.. _creatingtractstable:

Création de la table de traçage des recensements
------------------------------------------------

Dans le répertoire ``\data\`` des travaux pratiques, il y a un fichier qui contient des données attributaires, mais pas de géométries, ce fichier est nommé ``nyc_census_sociodata.sql``. La table contient des données sociaux-économiques interressantes à propos de New York : revenus financiers, éducation .... Il y a juste un problème, les données sont rassemblé en "trace de recensement" et nous n'avons pas de données spatiales associées !

Dans cette partie nous allons

 * Charger la table ``nyc_census_sociodata.sql``
 * Créer une table spatiale pour les traces de recensement
 * Joindre les données attributaires à nos données spatiales
 * Réaliser certaines analises sur nos nouvelles données
 
Chargement du fichier nyc_census_sociodata.sql
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 #. Ouvrez la fenêtre de requêtage SQL depuis PgAdmin
 #. Selectionnez **File->Open** depuis le menu et naviguez jusqu'au fichier ``nyc_census_sociodata.sql``
 #. Cliquez sur le bouton "Run Query"
 #. Si vous cliquez sur le bouton "Refresh" depuis PgAdmin, la liste des table devrait contenir votre nouvelle table ``nyc_census_sociodata``
 
Création de la table traces de recensement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
Comme nous l'avons dans la partie précédente, nous pouvons construire des géométries de niveau suppérieur en utilisant nos blocks de base en utilisant une partie de la clef ``blkid``. Afin de calculer les traces de recensement, nous avons besoin de regrouper les blocks en uitlisant les 11 premiers caractères de la colonne ``blkid``. 
 
  ::

    360610001009000 = 36 061 00100 9000

    36     = State of New York 
    061    = New York County (Manhattan)
    000100 = Census Tract
    9      = Census Block Group
    000    = Census Block

Création de la nouvelle table en utilisant la fonction d'agrégation :command:`ST_Union` :
 
.. code-block:: sql
   
   -- Création de la table
   CREATE TABLE nyc_census_tract_geoms AS
   SELECT 
     ST_Union(the_geom) AS the_geom, 
     SubStr(blkid,1,11) AS tractid
   FROM nyc_census_blocks
   GROUP BY tractid;
     
   -- Indexation du champ tractid
   CREATE INDEX nyc_census_tract_geoms_tractid_idx ON nyc_census_tract_geoms (tractid);
     
   -- Mise à jour de la table geometry_columns
   SELECT Populate_Geometry_Columns();

Regrouper les données attributaires et spatiales
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

L'objectif est ici de regrouper les données spatiales que nous avons créé avec les donées attributaires que nous avions chargé initialement.
  
.. code-block:: sql
  
  -- Création de la table
  CREATE TABLE nyc_census_tracts AS
  SELECT 
    g.the_geom,
    a.*
  FROM nyc_census_tract_geoms g
  JOIN nyc_census_sociodata a
  ON g.tractid = a.tractid;
    
  -- Indexation des géométries
  CREATE INDEX nyc_census_tract_gidx ON nyc_census_tracts USING GIST (the_geom);
    
  -- Mise à jour de la table geometry_columns
  SELECT Populate_Geometry_Columns();

.. _interestingquestion:

Répondre à une question interressante
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     
Répondre à une question interressante ! "Lister les 10 meilleurs quartiers ordonnées par la proportion de personne ayant acquis un diplome". 
  
.. code-block:: sql
  
  SELECT 
    Round(100.0 * Sum(t.edu_graduate_dipl) / Sum(t.edu_total), 1) AS graduate_pct, 
    n.name, n.boroname 
  FROM nyc_neighborhoods n 
  JOIN nyc_census_tracts t 
  ON ST_Intersects(n.the_geom, t.the_geom) 
  WHERE t.edu_total > 0
  GROUP BY n.name, n.boroname
  ORDER BY graduate_pct DESC
  LIMIT 10;

Nous sommons les statistiques qui nous interressent, nous les divisons ensuite à la fin. Afin d'aviter l'erreur de non-division par zero, nous ne prennons pas en compte les quartiers qui n'ont aucune personne ayant obtenu un diplome.

::
  
   graduate_pct |       name        | boroname  
  --------------+-------------------+-----------
           40.4 | Carnegie Hill     | Manhattan
           40.2 | Flatbush          | Brooklyn
           34.8 | Battery Park      | Manhattan
           33.9 | North Sutton Area | Manhattan
           33.4 | Upper West Side   | Manhattan
           33.3 | Upper East Side   | Manhattan
           32.0 | Tribeca           | Manhattan
           31.8 | Greenwich Village | Manhattan
           29.8 | West Village      | Manhattan
           29.7 | Central Park      | Manhattan
    
  
.. _polypolyjoins:

Polygones/Jointures de polygones
---------------------------------

Dans notre requête interressante (dans :ref:`interestingquestion`) nous avons utilisé la fonction :command:`ST_Intersects(geometry_a, geometry_b)` pour déterminer quelle entité polygonale à inclure dans chaque groupe de quartier. Ce qui nous conduit à la question : que ce passe-t-il si une entité tombe ntre deux quartier ? Il intersectera chacun d'entre eux et ainsi sera inclu dans **chacun** des résultats. 

.. image:: ./screenshots/centroid_neighborhood.png

Pour éviter ce cas de double comptage il existe trois méthodes :

 * La méthode simple consiste a s'assurer que chaque entité ne se retrouve que dans **un** seul groupe géograhique (en utilisant :command:`ST_Centroid(geometry)`)
 * La méthode complexe consiste à disviser les parties qui se croisent en utilisant les bordures (en utilisant :command:`ST_Intersection(geometry,geometry)`)
 
Voici un exemple d'utilisation de la méthode simple pour éviter le double comptage dans notre requête précédente :

.. code-block:: sql

  SELECT 
    Round(100.0 * Sum(t.edu_graduate_dipl) / Sum(t.edu_total), 1) AS graduate_pct, 
    n.name, n.boroname 
  FROM nyc_neighborhoods n 
  JOIN nyc_census_tracts t 
  ON ST_Contains(n.the_geom, ST_Centroid(t.the_geom)) 
  WHERE t.edu_total > 0
  GROUP BY n.name, n.boroname
  ORDER BY graduate_pct DESC
  LIMIT 10;
  
Remarquez que la requête prend plus de temps à s'exécuter, puisque la fonction :command:`ST_Centroid` doit être effectuée pour chaque entité.

::

   graduate_pct |       name        | boroname  
  --------------+-------------------+-----------
           49.2 | Carnegie Hill     | Manhattan
           39.5 | Battery Park      | Manhattan
           34.3 | Upper East Side   | Manhattan
           33.6 | Upper West Side   | Manhattan
           32.5 | Greenwich Village | Manhattan
           32.2 | Tribeca           | Manhattan
           31.3 | North Sutton Area | Manhattan
           30.8 | West Village      | Manhattan
           30.1 | Downtown          | Brooklyn
           28.4 | Cobble Hill       | Brooklyn
  
Éviter le double comptage change le résultat !


.. _largeradiusjoins:

Jointures utilisant un large rayon de distance
----------------------------------------------

Une requête qu'il est sympat de demander est : "Comment les temps de permutation des gens proches (dans un rayon de 500 metres ) des stations de métros diffèrent de ceuxqui en vive loin ? "

Néanmoins, la question rencontre les même problème de double comptage : plusieurs personnes seront dans un rayon de 500 metres de plusieurs stations de métros différentes. Coparons la population de New York :

.. code-block:: sql

  SELECT Sum(popn_total)
  FROM nyc_census_blocks;
  
::

  8008278
  
Avec la population des gens de New York dans un rayon de 500 metres d'une station de métros :

.. code-block:: sql

  SELECT Sum(popn_total)
  FROM nyc_census_blocks census
  JOIN nyc_subway_stations subway
  ON ST_DWithin(census.the_geom, subway.the_geom, 500);
  
::

  10556898

Il y a plus de personnes proches du métro qu'il y a de peronnes ! Clairement, notre requête SQL simple rencontre un gros problème de double comptage. Vous pouvez voir le problème en regardant l'image des zones tampons créées pour les stations.

.. image:: ./screenshots/subways_buffered.png

La solution est de s'assurer que nous avons seulement des blocks distincts avant de les les regrouper. Nou spouvons réaliser cela en cassant notre requête en sous-requêtes qui récupère les blocks distincts, regroupé ensuite pour retrouner notre réponse :

.. code-block:: sql

  SELECT Sum(popn_total)
  FROM (
    SELECT DISTINCT ON (blkid) popn_total
    FROM nyc_census_blocks census
    JOIN nyc_subway_stations subway
    ON ST_DWithin(census.the_geom, subway.the_geom, 500)
  ) AS distinct_blocks;
  
::

  4953599

C'est mieux ! Donc un peu plus de 50 % de la population de New York vit à proximité (50m environ 5 à 7 minutes de marche) du métro.



