.. _projection_exercises:

Partie 16 : Exercices sur les projections
=========================================

Voici un rappel de certaines fonctions que nous avons vu. Elles seront utiles pour les exercices !
     
* :command:`sum(expression)` agrégation qui retourne la somme d'un ensemble de valeurs
* :command:`ST_Length(linestring)` retourne la longueur d'une ligne
* :command:`ST_SRID(geometry, srid)` retourne le SRID d'une géométrie
* :command:`ST_Transform(geometry, srid)` reprojette des géométries dans un autre système de références spatiales
* :command:`ST_GeomFromText(text)` retourne un objet ``geometry``
* :command:`ST_AsText(geometry)` retourne le WKT (``text``)
* :command:`ST_AsGML(geometry)` retourne le GML (``text``)

Rappelez-vous les resssources en ligne :

* http://spatialreference.org
* http://prj2epsg.org

Et les tables disponibles :

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

 * **"Quelle est la longueur des rue de New York, mesurée en UTM 18 ?"**
 
   .. code-block:: sql

     SELECT Sum(ST_Length(the_geom))
       FROM nyc_streets;

   :: 
   
     10418904.7172
      
 * **"Quelle est la définition du SRID 2831 ?"**   
    
   .. code-block:: sql

     SELECT srtext FROM spatial_ref_sys
     WHERE SRID = 2831;

Ou, via `prj2epsg <http://prj2epsg.org/epsg/2831>`_

 ::

  PROJCS["NAD83(HARN) / New York Long Island", 
  GEOGCS["NAD83(HARN)", 
    DATUM["NAD83 (High Accuracy Regional Network)", 
      SPHEROID["GRS 1980", 6378137.0, 298.257222101, AUTHORITY["EPSG","7019"]], 
      TOWGS84[-0.991, 1.9072, 0.5129, 0.0257899075194932, -0.009650098960270402, -0.011659943232342112, 0.0], 
      AUTHORITY["EPSG","6152"]], 
    PRIMEM["Greenwich", 0.0, AUTHORITY["EPSG","8901"]], 
    UNIT["degree", 0.017453292519943295], 
    AXIS["Geodetic longitude", EAST], 
    AXIS["Geodetic latitude", NORTH], 
    AUTHORITY["EPSG","4152"]], 
  PROJECTION["Lambert Conic Conformal (2SP)", AUTHORITY["EPSG","9802"]], 
  PARAMETER["central_meridian", -74.0], 
  PARAMETER["latitude_of_origin", 40.166666666666664], 
  PARAMETER["standard_parallel_1", 41.03333333333333], 
  PARAMETER["false_easting", 300000.0], 
  PARAMETER["false_northing", 0.0], 
  PARAMETER["scale_factor", 1.0], 
  PARAMETER["standard_parallel_2", 40.666666666666664], 
  UNIT["m", 1.0], 
  AXIS["Easting", EAST], 
  AXIS["Northing", NORTH], 
  AUTHORITY["EPSG","2831"]]
  

 * **"Quelle est la longueur des rue de New York, mesuré en utilisant le SRID 2831 ?"**
 
   .. code-block:: sql

     SELECT Sum(ST_Length(ST_Transform(the_geom,2831)))
       FROM nyc_streets;

   :: 
   
     10421993.706374
     
   .. note::
   
     La différence entre les mesure en UTM 18 et en Stateplane Long Island est de (10421993 - 10418904)/10418904, soit 0.02%. Calculé sur la sphéroïde en utilissant en :ref:`geography`, le total des longueurs des route est 10421999, ce qui est proche de la valeur dans l'autre système de projection (Stateplane Long Island). Ce dernier est précisément calibré pour une petite zone géographique (la ville de New York) alors que le système UTM 18 doit fournir un résultat raisonable pour une zone régionale beaucoup plus large.
     
 * **"Quelle est la représentation KML du point de la station de métris 'Broad St' ?"**
 
   .. code-block:: sql
   
     SELECT ST_AsKML(the_geom) 
     FROM nyc_subway_stations
     WHERE name = 'Broad St';
     
   :: 
   
     <Point><coordinates>-74.010671468873468,40.707104815584088</coordinates></Point>
     
Hé ! les coordonnées sont géographiques bien que nous n'ayons pas fait appel à la fonction  :command:`ST_Transform`, mais pourquoi ? Parce que le standard KML spécifie que toutes les coordonnées *doivent* être géographiques (en fait, dans le système EPSG:4326), donc la fonction :command:`ST_AsKML` réalise la transformation automatiquement.
