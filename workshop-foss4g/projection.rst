.. _projection:

Partie 15 : Projections des données
===================================

La Terre n'est pas plate et il n'y a pas de moyen simple de la poser à plat sur une carte en papier (ou l'écran d'un ordinateur). Certaines projections préservent les aires, donc tout les objets ont des tailles relatives aux autres, d'autre projections conservent les angles (conformes) comme la projection Mercator. Certaines projections tentent de minimiser la distorsion des différents paramètres. Le point commun entre toutes les projections est qu'elles transforment le monde (sphérique) en un système plat de coordonnées cartésiennes, et le choix de la projection dépend de ce que vous souhaitez faire avec vos données.

Nous avons déjà recontrer des projections, lorsque nous avons charger les données de la ville de Ney York .Rappelez-vous qu'elles utilisaient le SRID 26918. Parfois, vous aurez malgré tout besoin de transformer et de reprojeter vos données d'un système de projection à un autre, en utilisant la fonction  :command:`ST_Transform(geometry, srid)`. Pour manipuler les identifiant de systèmes de références spatiales à partir d'une géométrie, PostGIS fournit les fonctions :command:`ST_SRID(geometry)` et :command:`ST_SetSRID(geometry, srid)`.

Nous pouvons vérifier le SRID de nos données avec la commande :command:`ST_SRID` :

.. code-block:: sql

  SELECT ST_SRID(the_geom) FROM nyc_streets LIMIT 1;
  
::

  26918
  
Et quelle est la définition du "26918" ? Comme nous l'avons vu lors de la partie ":ref:`chargement des données`", la définition se trouve dans la table ``spatial_ref_sys``. En fait, **deux** définitions sont présentes. La définition au format :term:`WKT` dans la colonne ``srtext``

.. code-block:: sql

   SELECT * FROM spatial_ref_sys WHERE srid = 26918;
   
En fait, pour les calculs internes de re-projection, c'est le contenu de la colonne ``proj4text`` qui est utilisé. Pour notre projection 26918, voici la définition au format proj.4 :

.. code-block:: sql

  SELECT proj4text FROM spatial_ref_sys WHERE srid = 26918;
  
::

  +proj=utm +zone=18 +ellps=GRS80 +datum=NAD83 +units=m +no_defs 
  
En pratique, les deux colonnes ``srtext`` et ``proj4text`` sont importantes : la colonne ``srtext`` est utilisée par les applications externes comme `GeoServer <http://geoserver.org>`_, uDig <udig.refractions.net>`_, `FME <http://www.safe.com/>`_  et autres, alors que la colonne ``proj4text`` est principalement utilisée par PostGIS en interne.

Comparaison de données
----------------------

Combinés, une coordonnée et un SRID définissent une position sur le globe. Sans le SRID, une coordonnée est juste une notion abstraite. Un système de coordonnées "cartésiennes" est définit comme un système de coordonnées "plat" sur la surface de la Terre. Puisque les fonctions de PostGIS utilisent cette surface plane, les opérations de comparaison nécessitent que l'ensemble des objets géométriques soient représentés dans le même système, ayant le même SRID.

Si vous utilisé des géométries avec différents SRID vous obtiendrez une erreur comme celle-ci :

.. code-block:: sql

  SELECT ST_Equals(
           ST_GeomFromText('POINT(0 0)', 4326),
           ST_GeomFromText('POINT(0 0)', 26918)
           );

::

  ERROR:  Operation on two geometries with different SRIDs
  CONTEXT:  SQL function "st_equals" statement 1
  

.. note::

   Faites attention de pas utiliser la transformation à la volée à l'aide de :command:`ST_Transform` trop souvent. Les indexes spatiaux sont construits en utilisant le SRID inclu dans les géométries. Si la comparaison est faite avec un SRID différent, les indexes spatiaux ne seront pas (la plupart du temps) utilisés. Il est reconnu qu'il vaut mieux choisir **un SRID** pour toutes les tables de votre base de données. N'utilisez la fonction de tranformation que lorsque vous lisez ou écrivez les données depuis une application externe.


Transformer les données
-----------------------

Si vous retournez à la définition au format proj4 du SRID 26918, vous pouvez voir que notre projectioin actuelle est de type UTM zone 18 (Universal Transvers Mercator), avec le mètre comme unité de mesure.

::

   +proj=utm +zone=18 +ellps=GRS80 +datum=NAD83 +units=m +no_defs 

Essayons de convertir certaines données de notre système de projection dans un système de coordonnées géographiques connu comme "longitude/latitude".

Pour convertir les données d'un SRID à l'autre, nous devons dans un premier temps vérifier que nos géométries ont un SRID valide. une fois que nous avons vérifié cela, nous devons ensuite trouver le SRID dans le lequel nous souhaitons re-projeter. En d'autre terme, quel est le SRID des coordonnées géographiques ?

Le SRID le plus connu pour les coordonnées géographiques est le 4326, qui correspond au couple "longitude/latitude sur la sphéroïde WGS84". Vous pouvez voir sa définition sur le site spatialreference.org.

  http://spatialreference.org/ref/epsg/4326/
  
Vous pouvez aussi récupérer les définitions dans la table  ``spatial_ref_sys`` :

.. code-block:: sql

  SELECT srtext FROM spatial_ref_sys WHERE srid = 4326;
  
::

  GEOGCS["WGS 84",
    DATUM["WGS_1984",
      SPHEROID["WGS 84",6378137,298.257223563,AUTHORITY["EPSG","7030"]],
      AUTHORITY["EPSG","6326"]],
    PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],
    UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],
    AUTHORITY["EPSG","4326"]]

Essayons de convertir les cordonnées de la station 'Broad St' :

.. code-block:: sql

  SELECT ST_AsText(ST_Transform(the_geom,4326)) 
  FROM nyc_subway_stations 
  WHERE name = 'Broad St';
  
::

  POINT(-74.0106714688735 40.7071048155841)

Si vous chargez les données ou créez une nouvelle géométrie sans spécifier de SRID, la valeur du SRID prendra alors la valeur -1. Rapellez-vous que dans les :ref:`geometries`, lorsque nous avons créé nos tables géométriques nous n'avions pas spécifié un SRID. Si nous interrogeons la base, nous devons nous attendre à ce que toutes les tables préfixées par ``nyc_`` aient le SRID 26918, alors que la table ``geometries`` aura la valeur -1 par défaut.

Pour visualiser la table d'assignation des SRID, interroger la table ``geometry_columns`` de la base de données.

.. code-block:: sql

  SELECT f_table_name AS name, srid 
  FROM geometry_columns;
  
::

          name         | srid  
  ---------------------+-------
   nyc_census_blocks   | 26918
   nyc_neighborhoods   | 26918
   nyc_streets         | 26918
   nyc_subway_stations | 26918
   geometries          |    -1

  
Néanmoins, si vous connaissez le SRID de vos données, vous pouvez l'affecter par la suite en utilisant la fonction :command:`ST_SetSRID` sur les géométries. Ensuite vous pourrez les tranformer dans d'autres systèmes de projections.

.. code-block:: sql

   SELECT ST_AsText(
    ST_Transform(
      ST_SetSRID(geom,26918),
    4326)
   )
   FROM geometries;

Liste des fonctions
-------------------

`ST_AsText <http://postgis.org/docs/ST_AsText.html>`_: retourne la représentation au format Well-Known Text (WKT) sans la métadonnée SRID.

`ST_SetSRID(geometry, srid) <http://postgis.org/docs/ST_SetSRID.html>`_: affecte une valeur au SRID d'une géométrie.

`ST_SRID(geometry) <http://postgis.org/docs/ST_SRID.html>`_: retourne l'indentifiant du système de références spatialesd'un objet ST_Geometry comme définit dans la table spatial_ref_sys.

`ST_Transform(geometry, srid) <http://postgis.org/docs/ST_Transform.html>`_: retourne une nouvelle géométrie après avoi re-projeté  les données dans le système correspondant au SRID passé en paramètre.
