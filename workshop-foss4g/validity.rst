.. _validity:

Partie 20 : Validité
====================

Dans 90% des cas la réponse à la question "pourquoi mes requêtes me renvoit un message d'erreur du type 'TopologyException' error"" est : "un ou plusieurs des arguments passés sont invalides". Ce qui nous conduit à nous demander : que signifie invalide et pourquoi est-ce important ?

Qu'est-ce que la validité ?
---------------------------

La validité est surtout importante pour les polygones, qui définissent des aires et requièrent une bonne structuration. Les lignes sont vraiment simples et ne peuvent pas être invalides ainsi que les points.

Certaines des règles de validation des polygones semble évidentes, et d'autre semblent arbitraires (et le sont vraiment).

 * Les contours des Polygon doivent être fermés.
 * Les contours qui définissent des trous doivent être inclus dans la zone définit par le coutour extérieur.
 * Les contours ne doivent pas s'intersecter (ils ne doivent ni se croiser ni se toucher).
 * Les contours ne doivent pas toucher les autres contours, sauf en un point unique.

Les deux dernières règles font partie de la catégorie arbitraires. Il y a d'autre moyen de définir des poygones qui sont consistent mais les règles ci-dessus sont celles utilisées dans le standard :term:`OGC` :term:`SFSQL` que respecte PostGIS.

La raison pour laquelle ces règles sont importants est que les algorythmes de calcul dépendent de cette structuration consistante des arguments. Il est possible de construire des algorythmes qui n'utilise pas cette structuration, mais ces fonctions tendents à être très lentes, étant donné que la première étape consistera à "analyser et construire  des strcuture à l'intérieur des données".

Voici un exemple de pourquoi cette structuration est importante. Ce polygone n'est pas valide :

::

  POLYGON((0 0, 0 1, 2 1, 2 2, 1 2, 1 0, 0 0));
  
Vous pouvez comprendre ce qui n'est pas valide en regardant cette figure :

.. image:: ./validity/figure_eight.png

Le contour externe est exactement en forme en 8 avec une intersection au milieux. Notez que la fonction de rendu graphique est tout de même capable d'en afficher l'intérieur, don visuellement cela ressemble bien à une "aire" : deux unités quarré, donc une aire couplant ces deux unités.

Essayons maintenant de voir ce que pense la base de données de notre polygone :

.. code-block:: sql

  SELECT ST_Area(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));
  
::

    st_area 
   ---------
          0

Que ce passe-t-il ici ? L'algorythme qui calcule l'aire suppose que les contours ne s'intersectent pas. Un contours normal devra toujours avoir une aire qui est bornée (l'intérieur) dans un sens de la ligne du contour (peu importe quelle sens, juste *un* sens). Néanmoins, dans notre figure en 8, le contour externe est à droite de la ligne pour un lobe et à gauche pour l'autre. Cela entraine que les aire qui sont calculées pour chaque lobe annulent la précédente (l'une vaut 1 et l'autre -1) donc le résultat est une "aire de zéro".


Détecter la validité
--------------------

Dans l'exemple précédent nous avions un polygone que nous **savions** non-valide. Comment déterminer les géométries non valides dans une tables d'un million d'enregistrements ? Avec la fonction :command:`ST_IsValid(geometry)`. Utilisé avec notre polygone précédent, nous abtenons rapidement la réponse :

.. code-block:: sql

  SELECT ST_IsValid(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));

:: 

  f

Maintenant nous savons que l'entité est non-valide mais nous ne savons pas pourquoi. Nous pouvons utilier la fonction :command:`ST_IsValidReason(geometry)` pour trtouver la cause de non validité :

.. code-block:: sql

  SELECT ST_IsValidReason(ST_GeometryFromText('POLYGON((0 0, 0 1, 1 1, 2 1, 2 2, 1 2, 1 1, 1 0, 0 0))'));

::

  Self-intersection[1 1]

Vous remarquerez qu'en plus de la raison (intersection) la localisation de la non validité (coordonnée (1 1)) est aussi renvoyée.

Nous pouvons aussi utiiliser la fonction :command:`ST_IsValid(geometry)` pour tester nos tables : 

.. code-block:: sql

  -- Trouver tout les polygones non valides et leur problème 
  SELECT name, boroname, ST_IsValidReason(the_geom)
  FROM nyc_neighborhoods
  WHERE NOT ST_IsValid(the_geom);

::

           name           |   boroname    |                     st_isvalidreason                      
 -------------------------+---------------+-----------------------------------------------------------
  Howard Beach            | Queens        | Self-intersection[597264.083368305 4499924.54228856]
  Corona                  | Queens        | Self-intersection[595483.058764138 4513817.95350787]
  Steinway                | Queens        | Self-intersection[593545.572199759 4514735.20870587]
  Red Hook                | Brooklyn      | Self-intersection[584306.820375986 4502360.51774956]



Réparer les invalides
---------------------

Commençons par la mauvaise nouvelle : il n'y a aucune garantie de pouvoir corriger une géométrie non valide. Dans le pire des scénarios, vous pouvez utiliser la fonction  :command:`ST_IsValid(geometry)` pour identifier les entités non valides, les déplacer dans une autre table, exporter cette table et les réparer à l'aide d'un outils extérieur.

Voici un exemple de requête SQL qui déplacent les géométries non valides hors de la table principale dans une table à part pour les exporter vers un programme de réparation.

.. code-block:: sql

  -- Table à part des géométries non-valide
  CREATE TABLE nyc_neighborhoods_invalid AS
  SELECT * FROM nyc_neighborhoods
  WHERE NOT ST_IsValid(the_geom);
  
  -- Suppression de la table principale
  DELETE FROM nyc_neighborhoods
  WHERE NOT ST_IsValid(the_geom);
  
Un bon outils pour réparer visuellemen des géométries non valide est OpenJump (http://openjump.org) qui contient un outils de validation depuis le menu **Tools->QA->Validate Selected Layers**.

Maintenant, la bonne nouvelle : un grand nombre de non-validités **peuvent être résolues dans la base de données** en utilisant la fonction : :command:`ST_Buffer`.

Le coup du Buffer tire avantafe de la manière dont les buffers sont construit : une géométrie bufferisée est une nouvelle géométrie, construite en déplaçant les lignes de la géométrie d'origine. Si vous déplacez les lignes originales par *rien* (zero) alors la nouvelle géométrie aura une structure identique à l'originale, mais puisqu'elle utilise les règles topologiques de l':term:`OGC, elle sera valide.

Par exemple, voici un cas classique de non-validité - le "polygone de la banane" - un seul contour que crée une zone mais se touche, laissant un "trou" qui n'en est pas un.

:: 

  POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))
  
.. image:: ./validity/banana.png

En créant un buffer de zero sur le polygone retourne un polygone :term:`OGC` valide, le contour externe et un contour interne qui touche l'autre en un seul point.

.. code-block:: sql

  SELECT ST_AsText(
           ST_Buffer(
             ST_GeometryFromText('POLYGON((0 0, 2 0, 1 1, 2 2, 3 1, 2 0, 4 0, 4 4, 0 4, 0 0))'),
             0.0
           )
         );

::

  POLYGON((0 0,0 4,4 4,4 0,2 0,0 0),(2 0,3 1,2 2,1 1,2 0))

.. note::

  Le "polygone banane" (ou "coquillage inversé") est un cas où le modèle topologique de l':term:`OGC` et de ESRI diffèrent. Le model ESRI considère que les contours que se touchent sont non valides et préfère la forme de banane pour ce cas de figure. Le modèle de l'OGC est l'inverse.
  
