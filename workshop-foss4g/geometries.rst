.. _geometries:

Partie 8 : Les géometries
=====================

Introduction
------------

Dans :ref:`une partie précédente<loading_data>` nous avons charger différentes données. Avant de commencer à jouer avec, commençons par regarder quelques exemples simples. Depuis pgAdmin, choisissez de nouveau la base de donnée **nyc** et ouvrez l'outil de requêtage SQL. Copiez cette exemple de code SQL (après avoir supprimer le contenu présent par défaut si nécessaire) puis exécutez-le.

.. code-block:: sql

  CREATE TABLE geometries (name varchar, geom geometry);
  
  INSERT INTO geometries VALUES 
    ('Point', 'POINT(0 0)'),
    ('Linestring', 'LINESTRING(0 0, 1 1, 2 1, 2 2)'),
    ('Polygon', 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ('PolygonWithHole', 'POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))'),
    ('Collection', 'GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))');
    
  SELECT Populate_Geometry_Columns();
  
  SELECT name, ST_AsText(geom) FROM geometries;

.. image:: ./geometries/start01.png

L'exemple ci-dessus créé une table (**geometries**) puis y insert cinq géométries : un point, une ligne, un polygone, un polygone avec un trou, et une collection. Les lignes insérées sont sélectionnées et affichées dans le tableau de sortie.

Les tables de métadonnées
-------------------------

Dans le respect de la spécification Simple Features for SQL (:term:`SFSQL`), PostGIS fournit deux tables pour récupérer et s'informer sur les types de géométries disponibles dans une base de données spécifique.

* La première table, ``spatial_ref_sys``, définit tout les systèmes de projection connus de la base de données et sera décrite plus en détails plus tard.  
* La seconde table, ``geometry_columns``, fournit une liste de toutes les "entités" (définit comme un objet avec un attribut géométrique) et les détails de base relatives à ces entités. 

.. image:: ./geometries/table01.png

Dans l'exemple founit en introduction, la fonction :command:`Populate_Geometry_Columns()` détecte toute les colonnes de la base de données qui contiennent des géométries et met à jour la table ``geometry_columns`` pour y inclure leurs références.

Regardons maintenant la table ``geometry_columns`` de notre base de données. Copiez cette commande dans la fenêtre de requêtage :


.. code-block:: sql

  SELECT * FROM geometry_columns;

.. image:: ./geometries/start08.png

* ``f_table_catalog``, ``f_table_schema``, et ``f_table_name`` fournissent le nom complet de la table contenant une géométrie donnée. Étant donné que PostgreSQL n'utilise pas de catalogues, ``f_table_catalog`` est toujouts vide.  
* ``f_geometry_column`` est le nom de la colonne qui contient la géométrie -- pour les tables ayant plusieurs colonnes géométriques, il y a un enregistrement dans cette table pour chacune.
* ``coord_dimension`` et ``srid`` définissent respectivement la dimension de la géométrie (en 2-, 3- or 4-dimensions) et le système de références spatiales qui fait référence à la table ``spatial_ref_sys``.  
* La colonne ``type`` définit le type de géométrie comme décrit plus tôt, nous avons déjà vu les points et les lignes.  

En interrogeant cette table, les clients SIG et les libraires peuvent déterminer quoi attendre lors de la récupération des données et peuvent réaliser les opération de reprojection, transformation ou rendu sans avoir à inspecter chaque géométrie.

Réprésenter des objets du monde réel
-----------------------------------

La spécification Simple Features for SQL (:term:`SFSQL`), le standard ayant guidé le développement de PostGIS, définit comment les objets du monde réel doivent être représentés. En considérant une forme continue à une seule résolution fixe, nous obtenons une piètre représentation des objets. SFSQL considère uniquement les représentations en 2 dimensions. PostGIS a étendu cela en ajoutant les représentation en 3 et 4 dimensions. Plus récemment, la spécification SQL-Multimedia Part 3 (:term:`SQL/MM`) a officiellement définit sa propre représentation.

Notre table exemple contient différents types de géométries. Nous pouvons récupérer les informations de chaque objet en utilisant les fonctions qui lisent les métadonnées de la géométrie.

 * :command:`ST_GeometryType(geometry)` retourne le type de la géométrie
 * :command:`ST_NDims(geometry)` retourne le nombre de dimensions d'une géométrie
 * :command:`ST_SRID(geometry)` retourne l'identifiant de référence spatiale de la géométrie

.. code-block:: sql

  SELECT name, ST_GeometryType(geom), ST_NDims(geom), ST_SRID(geom)
    FROM geometries;

::

       name       |    st_geometrytype    | st_ndims | st_srid 
 -----------------+-----------------------+----------+---------
  Point           | ST_Point              |        2 |      -1
  Polygon         | ST_Polygon            |        2 |      -1
  PolygonWithHole | ST_Polygon            |        2 |      -1
  Collection      | ST_GeometryCollection |        2 |      -1
  Linestring      | ST_LineString         |        2 |      -1



Les points
~~~~~~~~~~~

.. image:: ./introduction/points.png
   :align: center

Un **point** représente une localisation unique sur la Terre. Ce point est représenté par une seule coordonnée (incluant soit 2, 3 ou 4 dimensions). Les points sont utilisés pour représenter des objets lorsque le détail exact du contour n'est pas important à une échelle donnée. Par exemple, les villes sur une carte du monde peuvent être décrite sous la forme de points alors qu'une carte régionale utiliserait une représentation polygonale des villes.

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Point';

::

  POINT(0 0)

Certaines des fonctions spécifiques pour travailler avec les points sont :

 * :command:`ST_X(geometry)` retourne la composante X
 * :command:`ST_Y(geometry)` retourne la composante Y 

Donc, nous pouvons lire les coordonnées d'un point de la manière suivante :

.. code-block:: sql

  SELECT ST_X(geom), ST_Y(geom)
    FROM geometries
    WHERE name = 'Point';

La table des stations de métro de la ville de New York  (``nyc_subway_stations``) est un ensemble de données représenté sous la forme de points. La requête SQL suivante renverra la géométrie associée à un point (dans la colonne :command:`ST_AsText`).

.. code-block:: sql

  SELECT name, ST_AsText(the_geom)
    FROM nyc_subway_stations
    LIMIT 1;


Les lignes 
~~~~~~~~~~~

.. image:: ./introduction/lines.png
   :align: center

Une **ligne** est un chemin entre plusieurs points. Elle prend la forme d'un tableau ordonné composé de deux (ou plusieurs) points. Les routes et les rivières sont typiquement représentés sous la forme de lignes. Une ligne est dite **fermée** si elle commence et fini en un même point. Elle est dite **simple** si elle ne se coupe pas ou ne se touche pas elle-même (sauf à ses extrémités si elle est fermée). Une ligne peut être à la fois **fermée** et **simple**.

Le réseau des rues de New York (``nyc_streets``) a été chargé auparavant. Cet ensemble de données contient les détails comme le nom et le type des rues. Une rue du monde réel pourrait être constituée de plusieurs lignes, chacune représentant une file avec différents attributs.

La requête SQL suivante retourne la géométrie associée à une ligne (dans la colonne :command:`ST_AsText`) :

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Linestring';
  
::

  LINESTRING(0 0, 1 1, 2 1, 2 2)

Les fonctions spatiales permettant de travailler avec les lignes sont les suivantes :

 * :command:`ST_Length(geometry)` retourne la longueur d'une ligne
 * :command:`ST_StartPoint(geometry)` retourne le premier point d'une ligne
 * :command:`ST_EndPoint(geometry)` retourne le denier point d'une ligne
 * :command:`ST_NPoints(geometry)` retourne le nombre de points dans une ligne

Donc, la longueur de notre ligne est :

.. code-block:: sql

  SELECT ST_Length(geom) 
    FROM geometries
    WHERE name = 'Linestring';

::

  3.41421356237309


Les polygones
~~~~~~~~~~~~~~

.. image:: ./introduction/polygons.png
  :align: center

Un polygone est représenté comme une zone. Le contour externe du polygone est représenté par une ligne simple et fermée. Les trous sont représenté de la même manière.

Les polygones sont utilisés pour représenter les objets dont les tailles et la forme sont importants. Les limites de villes, les parcs, les bâtiments ou les cours d'eau sont habituellement représenté par des polygones lorsque l'échelle est suffisament élevée pour voir distinguer leurs zones. Les routes et les rivières peuvent parfois être représenté comme des polygones.

La requête SQL suivante retournera la géométrie associée à un polygone (dans la colonne :command:`ST_AsText`).

.. code-block:: sql

  SELECT ST_AsText(geom) 
    FROM geometries
    WHERE name LIKE 'Polygon%';

.. note::

  Plutôt que d'utiliser le signe ``=`` dans notre clause ``WHERE``, nous avons utilisé l'opérateur ``LIKE`` pour pouvoir définir notre comparaison. Vous auriez sans doute voulu utiliser le symbole ``*`` pour exprimer "n'importe quelle valeur" mais en SQL vous devez utiliser : ``%`` et l'opérateur ``LIKE`` pour informer le système que cette comparaison doit être possible.

::

 POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))
 POLYGON((0 0, 10 0, 10 10, 0 10, 0 0),(1 1, 1 2, 2 2, 2 1, 1 1))

Le premier polygone a seulement une ligne. Le second a un "trou". La plupart des systèmes de rendu graphique supportent le concept de "polygone", mais les systèmes SIG sont les seuls à accepter que les polygones puissent contenir des trous.

.. image:: ./screenshots/polygons.png

Certaines des fonctions spatiales spécifiques de traitement des polygones sont :

 * :command:`ST_Area(geometry)` retourne l'aire d'un polygone
 * :command:`ST_NRings(geometry)` retourne le nombre de contours (habituellement 1, plus lorsqu'il y a des trous)
 * :command:`ST_ExteriorRing(geometry)` retourne le contour extérieur
 * :command:`ST_InteriorRingN(geometry,n)` retourne le contour intérieur numéro n
 * :command:`ST_Perimeter(geometry)` retourne la longueur de tout les contours

Nous pouvons calculer l'aire de nos polygones en utilisant la fonction area :

.. code-block:: sql

  SELECT name, ST_Area(geom) 
    FROM geometries
    WHERE name LIKE 'Polygon%';

::

  Polygon            1
  PolygonWithHole    99

Remarquez que le polygone contenant un trou a une aire égale à l'aire du contour externe (un carré de 10 sur 10) moins l'aire du trou (un carré de 1 sur 1).

Les collections
~~~~~~~~~~~~~~~~ 

Il y a quatre types de collections, qui regroupe ensemble plusieurs géométries simples.

 * **MultiPoint**, une collection de points
 * **MultiLineString**, une collection de lignes
 * **MultiPolygon**, une collection de polygones
 * **GeometryCollection**, une collection hétérogène de n'importe quelle géométrie (dont d'autre collections)

Les collections sont un concept présents dans les logiciels SIG  plus que dans les applications de rendu graphique génériques. Elles sont utiles pour directement modeler les objets du monde réel comme des objet spatiaux. Par exemple, comment modéliser une parcelle qui a été coupée par un chemin ? Comme un **MultiPolygon**, ayant une partie de chaque coté du chemin.

.. image:: ./screenshots/collection2.png

Notre collection exemple contient un polygone et un point :

.. code-block:: sql

  SELECT name, ST_AsText(geom) 
    FROM geometries
    WHERE name = 'Collection';

::

  GEOMETRYCOLLECTION(POINT(2 0),POLYGON((0 0, 1 0, 1 1, 0 1, 0 0)))

.. image:: ./screenshots/collection.png

Certaines des fonctions spatiales spécifiques à la manipulation des collections sont :

 * :command:`ST_NumGeometries(geometry)` retourne le nombre de composantes d'une collection
 * :command:`ST_GeometryN(geometry,n)` retourne une composante spécifique
 * :command:`ST_Area(geometry)` retourne l'aire totale des composantes de types polygones
 * :command:`ST_Length(geometry)` retourne la longueur totale des composantes de types lignes

Entré / Sortie des géométries
-----------------------------

Dans la base de données, les géométries sont stockées dans un format utilisé uniquement par le programme PostGIS. Afin que des programmes externes puissent insérer et récupérer les données utiles, elles ont besoin d'être converties dans un format compris par l'application. Heureusement, PostGIS supporte un grand nombre de formats en entrée et en sortie :

 * Format texte bien connu (Well-known text :term:`WKT`)
 
   * :command:`ST_GeomFromText(text)` retourne une ``geometry``
   * :command:`ST_AsText(geometry)` retourne le ``texte``
   * :command:`ST_AsEWKT(geometry)` retourne le ``texte``
   
 * Format binaire bien connu (Well-known binary :term:`WKB`)
 
   * :command:`ST_GeomFromWKB(bytea)` retourne ``geometry``
   * :command:`ST_AsBinary(geometry)` retourne ``bytea``
   * :command:`ST_AsEWKB(geometry)` retourne ``bytea``
   
 * Geographic Mark-up Language (:term:`GML`)
 
   * :command:`ST_GeomFromGML(text)` retourne ``geometry``
   * :command:`ST_AsGML(geometry)` retourne ``text``
   
 * Keyhole Mark-up Language (:term:`KML`)
 
   * :command:`ST_GeomFromKML(text)` retourne ``geometry``
   * :command:`ST_AsKML(geometry)` retourne ``text``
   
 * :term:`GeoJSON`
 
   * :command:`ST_AsGeoJSON(geometry)` retourne ``text``
   
 * Scalable Vector Graphics (:term:`SVG`)
 
   * :command:`ST_AsSVG(geometry)` retourne ``text``
 
La requête SQL suivante montre un exemple de représentation en :term:`WKB` (l'appel à :command:`encode()` est requis pour convertir le format binaire en ASCII pour l'afficher) :

.. code-block:: sql

  SELECT encode(
    ST_AsBinary(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)')), 
    'hex');

.. image:: ./geometries/represent-04.png

Dans le reste de ces travaux pratiques, nous utiliserons principalement le format WKT pour que vous puissiez lire et comprendre les géométries que nous voyons. Néanmoins, pour la plupart des traitement actuels, comme la visualisation des données dans une application SIG, le transfert de données à des services web, ou l'exécution distante de traitements, le format WKB est un format de choix.

Puisque le WKT et le WKB sont définit dans la spécification :term:`SFSQL`, elles ne prennent pas en compte les géométries à 3 ou 4 dimensions. C'est pour cette raison que PostGIS définit les formats Extended Well Known Text (EWKT) et Extended Well Known Binary (EWKB). Cela permet de gérer de façon similaire aux formats WKT et WKB les dimensions ajoutées.

Voici un exemple de ligne 3D au format WKT :

.. code-block:: sql

  SELECT ST_AsEWKT(ST_GeometryFromText('LINESTRING(0 0 0,1 0 0,1 1 2)'));

.. image:: ./geometries/represent-05.png

.. code-block:: sql

  SELECT encode(ST_AsEWKB(ST_GeometryFromText(
      'LINESTRING(0 0 0,1 0 0,1 1 2)')), 'hex');

.. image:: ./geometries/represent-06.png

En plus de pouvoir générer les différents formats en sortie (WKT, WKB, GML, KML, JSON, SVG), PostGIS permet aussi de lire 4 de ces formats (WKT, WKB, GML, KML). La plupart des applications utilisent des fonctions créant des géométries à l'aide du format WKT ou WKB, mais les autres marchent aussi. Voici un exemple qui lit du GML et retourne du JSON :

.. code-block:: sql

  SELECT ST_AsGeoJSON(ST_GeomFromGML('<gml:Point><gml:coordinates>1,1</gml:coordinates></gml:Point>'));

.. image:: ./geometries/represent-07.png

Liste des fonctions
-------------------

`Populate_Geometry_Columns <http://postgis.org/docs/Populate_Geometry_Columns.html>`_: s'assure que les colonnes géométriques on les contraintes spatiales appropriées et qu'elles sont présentes dans la table  geometry_columns.

`ST_Area <http://postgis.org/docs/ST_Area.html>`_: retourne l'aire de la surface si c'est un polygon ou un multi-polygone. Pour le type "geometry" l'aire est dans l'unité du SRID. Pour les "geography" l'aire est en mètres carrés.

`ST_AsText <http://postgis.org/docs/ST_AsText.html>`_: retourne la représentation de la geometry/geography au format Well-Known Text (WKT) sans metadonnée correspondant au SRID.

`ST_AsBinary <http://postgis.org/docs/ST_AsBinary.html>`_: retourne la représentation de la geometry/geography au format Well-Known Binary (WKB) sans metadonnée correspondant u SRID. 

`ST_EndPoint <http://postgis.org/docs/ST_EndPoint.html>`_: retourne le dernier point d'une ligne.

`ST_AsEWKB <http://postgis.org/docs/ST_AsEWKB.html>`_: retourne la représentation de la geometrie au format Well-Known Binary (WKB) avec la métadonnée SRID.

`ST_AsEWKT <http://postgis.org/docs/ST_AsEWKT.html>`_: retourne la représentation de la geometrie au format Well-Known Text (WKT) avec la métadonnée SRID.

`ST_AsGeoJSON <http://postgis.org/docs/ST_AsGeoJSON.html>`_: retourne la géométrie au format GeoJSON.

`ST_AsGML <http://postgis.org/docs/ST_AsGML.html>`_: retourne la géométrie au format GML version 2 ou 3.

`ST_AsKML <http://postgis.org/docs/ST_AsKML.html>`_: retourne la géométrie au format KML. Nombreuses variantes. Par défaut : version=2 et precision=15.

`ST_AsSVG <http://postgis.org/docs/ST_AsSVG.html>`_: retourne la géométrie au format SVG.

`ST_ExteriorRing <http://postgis.org/docs/ST_ExteriorRing.html>`_: retourne une ligne représentant le contour extérieur du polygone. Retourne NULL si la géométrie n'est pas un polygone. Ne fonctionne pas avec les multi-polygone.

`ST_GeometryN <http://postgis.org/docs/ST_GeometryN.html>`_: retourne nième composante si la géométrie est du type GEOMETRYCOLLECTION, MULTIPOINT, MULTILINESTRING, MULTICURVE ou MULTIPOLYGON. Sinon, retourne NULL.

`ST_GeomFromGML <http://postgis.org/docs/ST_GeomFromGML.html>`_: prend en entrée une représentation GML de la géométrie et retourne un object PostGIS de type geometry.

`ST_GeomFromKML <http://postgis.org/docs/ST_GeomFromKML.html>`_: prend en entrée une représentation KML de la géométrie et retourne un object PostGIS de type geometry.

`ST_GeomFromText <http://postgis.org/docs/ST_GeomFromText.html>`_: retourne une valeur de type ST_Geometry à partir d'une représentation au format Well-Known Text (WKT).

`ST_GeomFromWKB <http://postgis.org/docs/ST_GeomFromWKB.html>`_: retourne une valeur de type ST_Geometry à partir d'une représenattion au format Well-Known Binary (WKB).

`ST_GeometryType <http://postgis.org/docs/ST_GeometryType.html>`_: retourne le type de géométrie de la valeur de type ST_Geometry.

`ST_InteriorRingN <http://postgis.org/docs/ST_InteriorRingN.html>`_: retourne le nième contour intérieur d'un polygone. Retourne NULL si la géométrie n'est pas un polygone ou si N est hors des limites.

`ST_Length <http://postgis.org/docs/ST_Length.html>`_: retourne la longueur en 2-dimensions si c'est une ligne ou une multi-lignes. Les objets de type geometry sont dans l'unité du système de références spatiales et les objet de type geography sont en metres (sphéroïde par défaut).

`ST_NDims <http://postgis.org/docs/ST_NDims.html>`_: retourne le nombre de dimensions d'une géométrie. Les valeurs possibles sont : 2,3 ou 4.

`ST_NPoints <http://postgis.org/docs/ST_NPoints.html>`_: retourne le nombre de points dans une géométrie.

`ST_NRings <http://postgis.org/docs/ST_NRings.html>`_: si la géométrie est un polygone ou un multi-polygone, retourne le nombre de contours.

`ST_NumGeometries <http://postgis.org/docs/ST_NumGeometries.html>`_: si la géométrie est du type GEOMETRYCOLLECTION (ou MULTI*) retourne le nombre de géométries, sinon retourne NULL.

`ST_Perimeter <http://postgis.org/docs/ST_Perimeter.html>`_: retourne la longueur du contours extérieur d'une valeur de type ST_Surface ou ST_MultiSurface (polygone, multi-polygone).

`ST_SRID <http://postgis.org/docs/ST_SRID.html>`_: retourne l'identifiant du système de référence spatiale définit dans la table spatial_ref_sys d'un objet de type ST_Geometry.

`ST_StartPoint <http://postgis.org/docs/ST_StartPoint.html>`_: retourne le premier point d'une ligne.

`ST_X <http://postgis.org/docs/ST_X.html>`_: retourne la coordonnée X d'un point, ou NULL si non présent. L'argument passé doit être un point.

`ST_Y <http://postgis.org/docs/ST_Y.html>`_: retourne la coordonnée Y d'un point, ou NULL si non présent. L'argument passé doit être un point.

