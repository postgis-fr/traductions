.. _geography:

Partie 17 : Coordonnées géographiques
=====================================

Il est très fréquent de manipuler des données à coordonnées "géographiques" ou de "longitude/latitude". 

Au contraire des coordonnées de type Mercator, UTM ou Stateplane, les coordonnées géographiques ne représentent pas une distance linéaire depuis une origine, tel que dans un plan. Elles décrivent la distance angulaire entre l'équateur et les pôles. Dans les sytèmes de coordonnées sphériques, un point est spécifié par son rayon (distance à l'origine), son angle de rotation par rapport au méridien plan, et son angle par rapport à l'axe pôlaire. 

.. image:: ./geography/cartesian_spherical.jpg


Vous pouvez continuer à utiliser des coordonnées géographiques comme des coordonnées cartésiennes approximatives pour vos analyses spatiales. Par contre les mesures de distances, d'aires et de longueur seront éronées. Etant donné que les coordonnées spériques mesurent des angles, l'unité est le dégré. Par exemple, les résultats cartésien approximatifs de tests tels que 'intersects' et 'contains' peuvent s'avérer terriblement faux. Par ailleurs, plus une zone est située près du pôle ou de la ligne de date internationale, plus la distance entre les points est agrandie.  
 

Voici par exemple les coordonnées des villes de Los Angeles et Paris.

 * Los Angeles: ``POINT(-118.4079 33.9434)``
 * Paris: ``POINT(2.3490 48.8533)``
 
La requête suivante calcule la distance entre Los Angeles et Paris en utilisant le système cartésien standard de PostGIS :command:`ST_Distance(geometry, geometry)`.  Notez que le SRID 4326 déclare un système de références spatiales géographiques.

.. code-block:: sql

  SELECT ST_Distance(
    ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
    ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
    );

::

  121.898285970107
  
Aha! 121! Mais, que veut dire cela ? 

L'unité pour SRID 4326 est le degré. Donc la réponse signifie 121 degrés. Sur une sphère, la taille d'un degré "au carré" est assez variable. Elle devient plsu petite au fur et à mesure que l'on s'éloigne de l'équateur. Pensez par exemple aux méridiens sur le globe qui se ressèrent entre eux au niveau des pôles. Donc une distance de 121 degrés ne veut rien dire !

Pour calculer une distance ayant du sens, nous devons traiter les coordonnées géographiques non pas come des coordonnées cartésiennes approximatives, mais plutôt comme de réelles coordonnées sphériques. Nous devons mesurer les distances entre les points comme de vrais chemins par dessus uen sphère, comme une portion d'un grand cercle.

Depuis sa version 1.5, PostGIS fournit cette fonctionnalité avec le type ``geography``.

.. note::

  Différentes bases de données spatiales développent différentes approches pour manipuler les coordonnées géographiques.
  
  * Oracle essaye de mettre à jour la différence de manière transparente en lanacant des calculs lorsuqe le SRID est géographique.
  * SQL Server utilise deux types spatiaux, "STGeometry" pour les coordonnées cartésiens et STGeography" pour les coordonnées géographqiues. 
  * Informix Spatial est une pure extension cartésienne d'Informix, alors qu'Informix Geodetic est une pure extension géographique. 
  * Comme SQL Server, PostGIS utilise deux types: "geometry" et "geography".
  
En utilisant le type ``geography`` plutot que ``geometry``, essayon sà nouveau de mesurer la distance entre Los Angeles et Paris. Au lieu de la commande :command:`ST_GeometryFromText(text)`, nous utiliserons cette fois :command:`ST_GeographyFromText(text)`.

.. code-block:: sql

  SELECT ST_Distance(
    ST_GeographyFromText('POINT(-118.4079 33.9434)'), -- Los Angeles (LAX)
    ST_GeographyFromText('POINT(2.5559 49.0083)')     -- Paris (CDG)
    );

::

  9124665.26917268

Toutes les valeurs retournées étant en mètres, notre réponse est donc 9124 kilomètres.

Les versions plus anciennes de PostGIS supportaient uniquement des calculs sur sphère très basiques comme la fonction :command:`ST_Distance_Spheroid(point, point, measurement)`. Celle-ci est très limitée et ne fonctionne uniquement sur des  points. Elle ne supporte pas non plus l'indexation au niveau des pôles ou de la ligne de date internationale.

Le besoin du support des autres types de géométries se fit ressentir lorsqu'il s'agissait de répondre à des questions du type  "A quelle distance la ligne de vol d'un avion Los Angeles/Paris passe-t-elle de l'Islande?" 

.. image:: ./geography/lax_cdg.jpg

Répondre à cette question en travaillant avec un plan cartésien fournit une très mauvaise réponse en effet ! En utilisant la ligne rouge, nou sobtenon sune bien meilleure réponse. Si nous convertissons notre vol LAX-CDG en une ligne et que nous calculons la distance à un point en Islande, nous obtiendrons la réponse exacte, en mètres. 

.. code-block:: sql

  SELECT ST_Distance(
    ST_GeographyFromText('LINESTRING(-118.4079 33.9434, 2.5559 49.0083)'), -- LAX-CDG
    ST_GeographyFromText('POINT(-21.8628 64.1286)')                        -- Iceland  
  );

::

  531773.757079116
  
Donc le point le plu sproche de l'Islande pendant le vol LAX-CDG est de 532 kilomètres.S

L'approche cartésienne pour manipuler les coordonnées géographiques pert tout son sens pour les objets situées au dessus de la ligne de date internationale. La route "sphérique" la plus courte entre Los-Angeles et Tokyo traverse l'océan Pacifique. La route "cartésienne" la plus courte traverse quant à elle les océans Atlantique et Indien.  

.. image:: ./geography/lax_nrt.png

.. code-block:: sql

   SELECT ST_Distance(
     ST_GeometryFromText('Point(-118.4079 33.9434)'),  -- LAX
     ST_GeometryFromText('Point(139.733 35.567)'))     -- NRT (Tokyo/Narita)
       AS geometry_distance, 
   ST_Distance(
     ST_GeographyFromText('Point(-118.4079 33.9434)'), -- LAX
     ST_GeographyFromText('Point(139.733 35.567)'))    -- NRT (Tokyo/Narita) 
       AS geography_distance; 
    
::

   geometry_distance | geography_distance 
  -------------------+--------------------
    258.146005837336 |   8833954.76996256


Utiliser le type 'Geography'
----------------------------

Afin d'importer des données dans une table de type geography, les objets géographiques doivent d'avord être projetées dans le système EPSG:4326 (longitude/latitude), ensuite elles doivent être converties en objets de type géographies. La fonction :command:`ST_Transform(geometry,srid)` convertie les coordonnées en géographies et la fonction :command:`Geography(geometry)` change le type ("cast") de géométrie à géographie.

.. code-block:: sql

  CREATE TABLE nyc_subway_stations_geog AS
  SELECT 
    Geography(ST_Transform(the_geom,4326)) AS geog, 
    name, 
    routes
  FROM nyc_subway_stations;
   
La construction d'une indexation spatiale sur une table stockant des objets de type géographie est exactement identique à la méthode employée pour les géométries :   

.. code-block:: sql

  CREATE INDEX nyc_subway_stations_geog_gix 
  ON nyc_subway_stations_geog USING GIST (geog);

La différence est camouflé : l'indexation des objets de type géographie gére correctement les requêtes qui recouvrent les pôles ou traverses les fuseaux horraires, alors que les géométries ne le supporteront pas.

Il n'y a qu'un petit nombre de fonctions disponibles pour le type géographie :  
 
 * :command:`ST_AsText(geography)` retourne la représentation ``textuelle``
 * :command:`ST_GeographyFromText(text)` retourne un objet de type ``geography``
 * :command:`ST_AsBinary(geography)` retourne la représentation binaire ``bytea``
 * :command:`ST_GeogFromWKB(bytea)` retourne un objet de type ``geography``
 * :command:`ST_AsSVG(geography)` retourne ``text``
 * :command:`ST_AsGML(geography)` retourne ``text``
 * :command:`ST_AsKML(geography)` retourne ``text``
 * :command:`ST_AsGeoJson(geography)` retourne ``text``
 * :command:`ST_Distance(geography, geography)` retourne ``double``
 * :command:`ST_DWithin(geography, geography, float8)` retourne ``boolean``
 * :command:`ST_Area(geography)` retourne ``double``
 * :command:`ST_Length(geography)` retourne ``double``
 * :command:`ST_Covers(geography, geography)` retourne ``boolean``
 * :command:`ST_CoveredBy(geography, geography)` retourne ``boolean``
 * :command:`ST_Intersects(geography, geography)` retourne ``boolean``
 * :command:`ST_Buffer(geography, float8)` retourne ``geography`` [#Casting_note]_
 * :command:`ST_Intersection(geography, geography)` retourne ``geography`` [#Casting_note]_
 
Création d'une table stockant des géograhpies
---------------------------------------------
 
Le code SQL permettant la création d'une nouvelle table avec une colonne de type géographie ressemble à la création d'une table stockant des géométries. Cependant, les objets de type géographie permettent de spécifier directement le type d'objet géographique à la création de la table. Par exemple :

.. code-block:: sql

  CREATE TABLE airports (
    code VARCHAR(3),
    geog GEOGRAPHY(Point)
  );
  
  INSERT INTO airports VALUES ('LAX', 'POINT(-118.4079 33.9434)');
  INSERT INTO airports VALUES ('CDG', 'POINT(2.5559 49.0083)');
  INSERT INTO airports VALUES ('REK', 'POINT(-21.8628 64.1286)');
  
Lors de la définitionn le type ``GEOGRAPHY(Point)`` spécifie que nos airoports sont des points. Les nouveau champs géographie n'est pas référencé dans la table ``geometry_columns``. Le stockage des métadonnées relatives aux données de type géograhpie sont stockées dans une vue appellée ``geography_columns`` qui est maintenue à jour automatiquement sans avoir besoin d'utiliser des fonctions comme ``geography_columns``.

.. code-block:: sql

  SELECT * FROM geography_columns;
  
::

           f_table_name         | f_geography_column | srid |   type   
 -------------------------------+--------------------+------+----------
  nyc_subway_stations_geography | geog               |    0 | Geometry
  airports                      | geog               | 4326 | Point
  
.. note::

  La possibilité de définir les types et le SRID lors de la création de la table (requête ``CREATE``), et la mise à jour automatique des métadonnées ``geometry_columns`` sont des fonctionalités qui seront adaptées pour le type géométrie pour la version 2.0 de PostGIS.  

Conversion de type
-------------------

Bien que les fonctions de bases qui s'appliquent au type géographie peuvent être utilisées dans un grand nombre de cas d'utilisation, il est parfois nécessaire d'accéder aux autres fonctions qui ne supportent que le type géométrie. Heureusement, il est possible de convertir des objets de type géométries en des objets de types géographies et inversement.

La syntaxe habituelle de PostgreSQL pour les conversion de type  consiste à ajouter à la valeur la chaîne suivante ``::typename``. Donc, ``2::text`` convertie la valeur numérique deux en une chaîne de caractères '2'. La commande : ``'POINT(0 0)'::geometry`` convertira la représentation textuelle d'un point en une point géométrique.

La fonction :command:`ST_X(point)` supporte seulement le type géométrique. Comment lire la coordonée X d'une de nos géographie ?

.. code-block:: sql

  SELECT code, ST_X(geog::geometry) AS longitude FROM airports;

::

  code | longitude 
 ------+-----------
  LAX  | -118.4079 
  CDG  |    2.5559
  REK  |  -21.8628

En ajoutant la chaîne ``::geometry`` à notre valeur géographique, nous la convertissons en une géographie ayant le SRID : 4326. À partir de maintenant, nous pouvons utiliser autemps de fonctions s'appliquant au géométries que nous le souhaitons. Mais, souvenez-vous - maintenant que nos objets sont des géométries, leur coordonnées seront interprétées comme des coordonnées cartésiennes, non pas sphériques.
 
 
Pourquoi (ne pas) utiliser les géographies
------------------------------------------

Les géographies ont des coordonnées universellement acceptées - chacun peut comprendre que représente la latitue et la longitude, mais peut de personne comprennent ce que les coordonnées UTM signifient. Pourquoi ne pas tout le temps utiliser  des géographies ?

 * Premièrement, comme indiqué précédemment, il n'y a que quelques fonctions qui supportent ce type de données. Vous risquer de perdre beaucoup de temps à contourner les problèmes liés à la non-disponibilité de certaines fonctions.
 * Deuxièmement, les calculs sur une sphère sont plus consomateurs en ressource que les mêmes calculs dans un système cartésien. Par exemple, la formule de calcul de distance (Pythagore) entraine un seul appèle à la fonction racine carré (sqrt()). La formule de calcul de distance sphérique (Haversine) utilise deux appèle à la fonction racine carré, et un appèle à arctan(), quatre appèle à sin() et deux à cos(). Les fonctions trigonométriques sont très couteuses, et les calculs sphériques les utilisent massivement.
 
Quel conclusion en tirer ? 

Si vos données sont géograhpiquement compact (contenu à l'intérieur d'un état, d'un pays ou d'une ville), utilisez le type ``geometry`` avec une projection cartésienne qui est pertinent pour votre localisation. Consultez le site http://spatialreference.org et tapez le nom de votre région pour visualiser la liste des système de projection applicables dans votre cas.

Si, d'un autre coté, vous avez besoin de calculer des distances qui est géographiquement éparse (recouvrant la plupart du monde), utiliser le type ``geography``. La compléxité de l'application que vous éviterait en travaillant avec des objets de type ``geography`` dépassera les problèmes de performances. La conversion de type  en géométrie permettra de dépasser les limites des fonctionnalités proposé pour ce type.

Liste des fonctions
-------------------

`ST_Distance(geometry, geometry) <http://postgis.org/docs/ST_Distance.html>`_: Pour le type géométrie, renvoit la distance cartésienne, pour les géographies la distance sphérique en métres.

`ST_GeographyFromText(text) <http://postgis.org/docs/ST_GeographyFromText.html>`_: Retourne la valeur géographique à partir d'une représentation en WKT ou EWKT.

`ST_Transform(geometry, srid) <http://postgis.org/docs/ST_Transform.html>`_: Retourne une nouvelle géométrie avec ses coordonnées reprojetées dans le système de référence spatial référencé par le SRID fournit.

`ST_X(point) <http://postgis.org/docs/ST_X.html>`_: Retourne la coordonnée X d'un point, ou NULL si non disponible. La valeur passée doit être un point.


.. rubric:: Footnotes

.. [#Casting_note] Les fonctions buffer et intersection sont actuellement construite sur le principe de conversion de type en géométries, et ne sont pas actuellement capable de gérer des coordonnées sphariques. Il en résulte qu'elles peuvent ne pas parvenir à retourner un résultat correcte pour des objets ayant une grande étendue qui ne peut être représenté correctement avec une représentation planaire.
 
   Par exemple, la fonction :command:`ST_Buffer(geography,distance)`  transforme les objets géographiques dans la "meilleure" projection, crée la zone tampon, puis les transforme à nouveau en des géographies. S'il n'y a pas de "meilleure" projection (l'objet est trop vaste), l'opération peut ne pas réussir à retourner une valeur correct ou retourner une one tampon mal formée.

