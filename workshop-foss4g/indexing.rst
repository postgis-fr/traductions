.. _indexing:

Partie 14 : L'indexation spatiale
=================================

Rapellez-vous que l'indexation spatiale est l'une des trois fonctionnalités clés d'une base de données spatiales. Les indexes permettent l'utilisation de grandes quantités de données dans une base. Sans l'indexation, chaque recherche d'entité nécessitera d'accéder séquentiellement à tout les enregistrements de la base de données. L'indexation accélères les recherche en organisant les données dans des arbres de recherche qui peuvent être parcouru efficacement pour retrouver une entité particulière.

L'indexation spatiale l'un des plus grands atouts de PostGIS. Dans les exemples précédents, nous avons construit nos jointures spatiales en comparant la totalité des tables. Ceci peut parfois s'averrer très coûteux : Réaliser la jointure de deux tables de 10000 enregistrements sans indexation nécessitera de comparer 100000000 valeurs, les comparaisons requises ne seront plus que 20000 avec l'indexation.

Lorsque nous avons chargé la table  ``nyc_census_blocks``, l'outils pgShapeLoader crée automatiquement un indexe spatial appelé ``nyc_census_blocks_the_geom_gist``.

Pour démontrer combien il est important d'indexer ses données pour la performance des requêtes, essayons de requêter notre table ``nyc_census_blocks`` **sans** utiliser notre indexe.

La première étape consiste à supprimer l'index.

.. code-block:: sql

  DROP INDEX nyc_census_blocks_the_geom_gist;
  
.. note::

   La commande ``DROP INDEX`` supprime un index existant de la base de données. Pour de plus amples informations à ce sujet, consultez la `documentation officielle de PostgreSQL <http://docs.postgresql.fr/9.1/sql-dropindex.html>`_.
   
Maintenant, regardons le temps d'exécution dans le coin en bas à droite de l'interface de requêtage de pgAdmin, puis lançons la commande suivante. Notre requête recherche les blocs de la rue Broad.

.. code-block:: sql

  SELECT blocks.blkid
  FROM nyc_census_blocks blocks
  JOIN nyc_subway_stations subways
  ON ST_Contains(blocks.the_geom, subways.the_geom)
  WHERE subways.name = 'Broad St';
  
::

       blkid      
 -----------------
  360610007003006
  
La table ``nyc_census_blocks`` est très petite (seulement quelque millier d'enregistrements) donc même sans l'index, la requête prends **55 ms** sur l'ordinateur de test.

Maintenant remettons en place l'index et lançons de nouveau la requête.

.. code-block:: sql

  CREATE INDEX nyc_census_blocks_the_geom_gist ON nyc_census_blocks USING GIST (the_geom);

.. note:: l'utilisation de la clause ``USING GIST`` spécifie à PostgreSQL de créer une structure (GIST) pour cet index. Si vous recevez un message d'erreur ressemblant à ``ERROR: index row requires 11340 bytes, maximum size is 8191`` lors de la création, cela signifie sans doute que vous avez omis la clause ``USING GIST``.

Sur l'rdinateur de test le temps d'exécution se réduit à **9 ms**. Plus votre table est grande, plus la différence de temps d'exécution pour une requête utilisant les indexes augmentera.

Comment les indexes spatiaux fonctionnent
-----------------------------------------

Les indexes des base de données standards créent des arbres hierarchiques basés sur les valeurs des colonnes à indexer. Les indexes spatiaux sont un peu différents - ils ne sont pas capables d'indexer des entités géométriques elles-même mais indexe leur étendues.

.. image:: ./indexing/bbox.png

Dans la figure ci-dessus, le nombre de lignes qui intersectent l'étoile jaune est *unique*, la ligne rouge. Mais l'étendue des entités qui intersectent la boîte jaune sont *deux*, la boîte rouge et la boîte bleue.

La manière dont les bases de données répondent de manière efficace à la question "Quelles lignes intersectent l'étoile jaune ?" correspond premièrement à répondre à la question "Quelle étendue intersecte l'étendue jaune" en utilisant les indexes (ce qui est très rapide) puis à calculer le résultat exact de la question "Quelles lignes intersectent l'étoile jaune ?" **seulement en utilisant les entités retourné par le premier test**.

Pour de grandes tables, il y a un système en "deux étapes" d'évaluation en utilisant dans un premier temps l'approximation à l'aide d'indexes, puis en réalisant le test exact sur une quantité bien moins importante de données ce qui réduit drastiquement le temps de calcul nécessaire à cette deuxième étape.

PotGIS et Oracle Spatial partage la même notion d'index structuré sous la forme "d'arbres R" [#RTree]_. Les arbres R classent les données sous forme de rectangles, de sous-rectangles etc. Cette structure d'index gère automatiquement la densité et la taille des objets.

.. image:: ./indexing/index-01.png

Requête avec seulement des indexes
----------------------------------

La plupart des fonctions utilisées par PostGIS (:command:`ST_Contains`, :command:`ST_Intersects`, :command:`ST_DWithin`, etc) prennent en compte les indexes automatiquement. Mais certaines fonctions (comme par exemple : :command:`ST_Relate`) ne les utilisent pas.

Pour utiliser une recherche par étendue utilisant les indexes (et pas de filtres), vous pouvez utiliser l'opérateur :command:`&&`. Pour les géométries, l'opérateur :command:`&&` signifie "l'étendue recouvre ou touche" de la même manière que l'opérateur :command:`=` sur des entiers signifie que les valeurs sont égales.

Essayons de comparer une requête avec seulement un indexe pour la population du quartier 'West Village'. En utilisant la commande :command:`&&` notre requête ressemble à cela :

.. code-block:: sql

  SELECT Sum(popn_total) 
  FROM nyc_neighborhoods neighborhoods
  JOIN nyc_census_blocks blocks
  ON neighborhoods.the_geom && blocks.the_geom
  WHERE neighborhoods.name = 'West Village';
  
::

  50325
  
Maintenant essayons la même requête en utilisant la fonction plus précise :command:`ST_Intersects`.

.. code-block:: sql

  SELECT Sum(popn_total) 
  FROM nyc_neighborhoods neighborhoods
  JOIN nyc_census_blocks blocks
  ON ST_Intersects(neighborhoods.the_geom, blocks.the_geom)
  WHERE neighborhoods.name = 'West Village';
  
::

  27141

Un plus faible nombre de résultats ! La première requête nous renvoit tout les blocs qui intersectent l'étendue du quartier, la seconde nous renvoit seulement les blocs qui intersectent le quartier lui-même.

Analyse
---------

Le plannificateur de requête de PostgreSQL choisit intelligemment d'utiliser ou non les indexes pour réaliser une requête. Il n'est pas toujours plus rapide d'utiliser un index pour réaliser une recherche : si la recherche doit renvoyer l'ensemble des enregistrements d'une table, parcourir l'index pour récupérer chaque valeur sera plus lent que de parcourir linéairement l'ensemble de la table.

Afin de savoir dans quelle situation il est nécessaire d'utiliser les idexes (lire une petite partie de la table plutôt qu'une grande partie), PostgreSQL conserve des statistiques relatives à la distribution des données dans chaque colonne indexée. Par défaut, PostgreSQL rassemble les statistiques sur une base régulière. Nénamoins, si vous changez dramatiquement le contenu de vos tables dans une période courte, les statisuqes ne seront alors plus à jour.

Pour vous assurez que les statistiques correspondent bien au contenu de la table actuelle, il est courrant d'utiliser la commande ``ANALYZE`` après un grand nombre de modifications ou de suppression de vos données. Cela force le système de gestion des statistiques à récupérer l'ensemble des données des colonnes indexées.

La commande ``ANALYZE`` demande à PostgreSQL de parcourir la table et de mettre à jour les statistiques utilisées par le plannificateur de requêtes (la plannification des requêtes sera traité utiltérieurement).

.. code-block:: sql

   ANALYZE nyc_census_blocks;
   
Néttoyage
---------

Il est souvent stressant de constater que la simple création d'un indexe n'est pas suffisant pour que PostgreSQL l'utilise efficacement. Le nettoyage doit être réalisé après qu'un indexe soit créé ou après un grand nombre de requêtes UDATE, INSERT ou DELETE est été réalisé sur une table. La commande ``VACUUM`` demande à PostgreSQL de récupérer chaque espace non utilisé dans les pages de la table qui sont laissé en l'état lors des requêtes UPDATE ou DELETE à cause du modèle d'estapillage multi-versions.

Le nettoyage des données est tellement important pour une utilisation efficace du serveur de base de données PostgreSQL qu'il existe maintenant une option "autovacuum".

Activée par défaut, le processus autovacuum nettoie (récupère l'espace libre) et analyse (met à jour les statistiques) vos tables suivant un interval donné déterminé par l'activité des bases de données. Bien que cela fonctionne avec les bases de données hautement transactionnelles, il n'est pas supportable de devoir attendre que le processus autovacuum se lance lors de la mise à jour ou la suppression massive de données. Dans ce cas, il faut lancer la commande ``VACUUM`` manuellement.

Le nettoyage et l'analyse de la base de données peut être réalisé séparément si nécessaire. Utiliser la commande ``VACUUM`` ne mettra pas à jour les statistiques alors que lancer la commande ``ANALYZE`` ne récupèrera pas l'espace libre des lignes d'une table. Chacune de ces commandes peut être lancée sur l'intégralité de la base de données, sur une table ou sur une seule colonne.

.. code-block:: sql

   VACUUM ANALYZE nyc_census_blocks;

Liste des fonctions
-------------------

`geometry_a && geometry_b <http://postgis.org/docs/ST_Geometry_Overlap.html>`_: retourne TRUE si l'étendue de A cheuvauche celle de B.

`geometry_a = geometry_b <http://postgis.org/docs/ST_Geometry_EQ.html>`_: retourne TRUE si l'étendue de A est la même que celle de B.

`ST_Intersects(geometry_a, geometry_b) <http://postgis.org/docs/ST_Intersects.html>`_: retourne TRUE si l'objet Geometrie/Geography "intersecte spatiallement" - (ont une partie en commun) et FALSE sinon (elles sont dijointes). 

.. rubric:: Footnotes

.. [#RTree] http://postgis.org/support/rtree.pdf

