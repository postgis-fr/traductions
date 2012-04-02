.. _postgis-functions:

Annexes A : Fonctions PostGIS
=============================

Constructeurs
------------

:command:`ST_MakePoint(Longitude, Latitude)` 
  Retourne un nouveau point. Note : ordre des coordonées (longitude puis latitude).

:command:`ST_GeomFromText(WellKnownText, srid)`
  Retourne une nouvelle géométrie à partir d'un représentation au format WKT et un SRID.

:command:`ST_SetSRID(geometry, srid)`
  Met à jour le SRID d'une géométrie. Retourne la même géométrie. Cela ne modifie pas les coordonnées de la géométrie, cela met simplement à jour le SRID. Cette fonction est utile pour reconditionner les géométries sans SRID.

:command:`ST_Expand(geometry, Radius)`
  Retourne une nouvelle géométrie qui est une extension de l'étendue de la géométrie passé en argument. Cette fonction est utile pour créer des envelopes pour des recherches utilisants les indexations.

Srotie
-------

:command:`ST_AsText(geometry)`
  Retourne une géométrie au format WKT.

:command:`ST_AsGML(geometry)`
  Retourne la géométrie au format standard OGC :term:`GML`.

:command:`ST_AsGeoJSON(geometry)`
  Retourne une géométrie au format "standard" `GeoJSON <http://geojson.org>`_.

Measures
------------

:command:`ST_Area(geometry)`
  Retourne l'aire d'une géométrie dans l'unité du système de références spatiales.

:command:`ST_Length(geometry)`
  Retourne la longueur de la géométrie dans l'unité du système de références spatiales.

:command:`ST_Perimeter(geometry)`
  Retourne le périmétre de la géométrie dans l'unité du système de références spatiales.

:command:`ST_NumPoints(linestring)`
  Retourne le nombre de sommets dans une ligne.

:command:`ST_NumRings(polygon)`
  Retourne le nombre de contours dans un polygone.

:command:`ST_NumGeometries(geometry)` 
  Retourne le nombre de géométries dans une collections de géométries.

Relations
-------------

:command:`ST_Distance(geometry, geometry)`
  Retourne la distance entre deux géométries dans l'unité du  système de références spatiales.

:command:`ST_DWithin(geometry, geometry, radius)` 
  Retourne vrai si les géométries sont distant d'un rayon de l'autre, sinon faux.

:command:`ST_Intersects(geometry, geometry)`
  Retourne vrai si les géométries sont disjointes, sinon faux.

:command:`ST_Contains(geometry, geometry)`
  Retourne vrai si la première géométrie est totalement contenu dans la seconde, sinon faux.

:command:`ST_Crosses(geometry, geometry)`
  Retourne vrai si une ligne ou les contours d'un polygone croisent une ligne ou un contour de polygone, sinon faux.
