.. _geometries_exercises:

Partie 9 : Exercices sur les géométries
======================================

Voici un petit rappel de toutes les fonction que nous avons abordé jusqu'à présent. Elles devraient être utiles pour les exercices !

 * :command:`sum(expression)` agrégation retournant la somme d'un ensemble
 * :command:`count(expression)` agrégation retournant le nombre d'éléments d'un ensemble
 * :command:`ST_GeometryType(geometry)` retourne le type de la géométrie
 * :command:`ST_NDims(geometry)` retourne le  nombre de dimensions
 * :command:`ST_SRID(geometry)` retourne l'identifiant du système de références spatiales
 * :command:`ST_X(point)` retourne la coordonnée X
 * :command:`ST_Y(point)` retourne la coordonnée Y
 * :command:`ST_Length(linestring)` retourne la longueur d'une ligne
 * :command:`ST_StartPoint(geometry)` retourne le premier point d'une ligne
 * :command:`ST_EndPoint(geometry)` retourne le dernier point d'une ligne
 * :command:`ST_NPoints(geometry)` retourne le nombre de points d'une ligne
 * :command:`ST_Area(geometry)` retourne l'aire d'un polygone
 * :command:`ST_NRings(geometry)` retourne le nombre de contours (1 ou plus si il y a des trous)
 * :command:`ST_ExteriorRing(polygon)` retourne le contour exterieur (ligne) d'un polygon
 * :command:`ST_InteriorRingN(polygon, integer)` retourne le contour intérieur (ligne) d'un polygone
 * :command:`ST_Perimeter(geometry)` retourne la longueur de tout les contours
 * :command:`ST_NumGeometries(multi/geomcollection)` retourne le nombre de composantes dans une collection
 * :command:`ST_GeometryN(geometry, integer)` retourne la nième entité de la collection
 * :command:`ST_GeomFromText(text)` retourne ``geometry``
 * :command:`ST_AsText(geometry)` retourne WKT ``text``
 * :command:`ST_AsEWKT(geometry)` retourne EWKT ``text``
 * :command:`ST_GeomFromWKB(bytea)` retourne ``geometry``
 * :command:`ST_AsBinary(geometry)` retourne WKB ``bytea``
 * :command:`ST_AsEWKB(geometry)` retourne EWKB ``bytea``
 * :command:`ST_GeomFromGML(text)` retourne ``geometry``
 * :command:`ST_AsGML(geometry)` retourne GML ``text``
 * :command:`ST_GeomFromKML(text)` retourne ``geometry``
 * :command:`ST_AsKML(geometry)` retourne KML ``text``
 * :command:`ST_AsGeoJSON(geometry)` retourne JSON ``text``
 * :command:`ST_AsSVG(geometry)` retourne SVG ``text``

Souvenez-vous aussi des tables disponibles:

 * ``nyc_census_blocks`` 
 
   * name, popn_total, boroname, the_geom
 
 * ``nyc_streets``
 
   * name, type, the_geom
   
 * ``nyc_subway_stations``
 
   * name, the_geom
 
 * ``nyc_neighborhoods``
 
   * name, boroname, the_geom

Exercices
---------

 * **"Quelle est l'aire du quartier 'West Village'?"**
 
   .. code-block:: sql

     SELECT ST_Area(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';
       
   :: 

     1044614.53027344

   .. note::

      L'aire est donnée en metres carrés. Pour obtenir l'aire en hectare, divisez par 10000. Pour obrenir l'aire en acres, divisez par 4047.

 * **"Quelle est l'aire de Manhattan en acres ?"** (Astuce: ``nyc_census_blocks`` et ``nyc_neighborhoods`` on toutes deux le champ ``boroname``.)
 
   .. code-block:: sql

     SELECT Sum(ST_Area(the_geom)) / 4047
       FROM nyc_neighborhoods
       WHERE boroname = 'Manhattan';

   :: 
   
     13965.3201224118

   or...

   .. code-block:: sql

     SELECT Sum(ST_Area(the_geom)) / 4047
       FROM nyc_census_blocks
       WHERE boroname = 'Manhattan';

   :: 
   
     14572.1575543757


 * **"Combien de blocs de la ville de New York ont des trous ?"**
 
   .. code-block:: sql

     SELECT Count(*) 
       FROM nyc_census_blocks
       WHERE ST_NRings(the_geom) > 1;

   :: 
   
     66 
   
 * **"Quel est la longueur totale des rues (en kilomètres) dans la ville de New York ?"** (Astuce: l'unité de mesure des données spatiales est le mètre, il y a 1000 mètres dans un kilomètre.)
  
    .. code-block:: sql

     SELECT Sum(ST_Length(the_geom)) / 1000
       FROM nyc_streets;

   :: 
   
     10418.9047172

 * **"Quelle est la longueur de 'Columbus Cir' (Columbus Circle) ?"**
 
     .. code-block:: sql
 
      SELECT ST_Length(the_geom)
        FROM nyc_streets
        WHERE name = 'Columbus Cir';

     :: 
   
       308.34199

 * **"Quelle est le contour de 'West Village' au format JSON ?"**
 
   .. code-block:: sql

     SELECT ST_AsGeoJSON(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';

   ::
     
      {"type":"MultiPolygon","coordinates":
       [[[[583263.2776595836,4509242.6260239873],
          [583276.81990686338,4509378.825446927], ...
          [583263.2776595836,4509242.6260239873]]]]}

La géométrie de type "MultiPolygon", interressant !
   
      
 * **"Combien de polygones sont dans le multi-polygone 'West Village' ?"**
 
   .. code-block:: sql

     SELECT ST_NumGeometries(the_geom)
       FROM nyc_neighborhoods
       WHERE name = 'West Village';

   ::

      1
       
   .. note::
   
      Il n'est pas rare de trouver des éléments de type multi-polygone ne contenant qu'un seul polygone dans des tables. L'utilisation du type multi-polygone permet d'utiliser une seule table pour y stocker des géométries simples et multiples sans mélanger les types.
       
       
 * **"Quel est la longueur des rues de la ville de New York, suivant leur type ?"**
 
   .. code-block:: sql

      SELECT type, Sum(ST_Length(the_geom)) AS length
       FROM nyc_streets
       GROUP BY type
       ORDER BY length DESC;

   ::
   
                            type                       |      length      
     --------------------------------------------------+------------------
      residential                                      | 8629870.33786606
      motorway                                         | 403622.478126363
      tertiary                                         | 360394.879051303
      motorway_link                                    | 294261.419479668
      secondary                                        | 276264.303897926
      unclassified                                     | 166936.371604458
      primary                                          | 135034.233017947
      footway                                          | 71798.4878378096
      service                                          |  28337.635038596
      trunk                                            | 20353.5819826076
      cycleway                                         | 8863.75144825929
      pedestrian                                       | 4867.05032825026
      construction                                     | 4803.08162103562
      residential; motorway_link                       | 3661.57506293745
      trunk_link                                       | 3202.18981240201
      primary_link                                     | 2492.57457083536
      living_street                                    | 1894.63905457332
      primary; residential; motorway_link; residential | 1367.76576941335
      undefined                                        |  380.53861910346
      steps                                            | 282.745221342127
      motorway_link; residential                       |  215.07778911517

    
   .. note::

      La clause ``ORDER BY length DESC`` ordonne le résultat par la valeur des longueurs dans l'ordre décroissant. Le résultat avec la plus grande valeur se retrouve au début la liste de résultats.

 
 
 
        
