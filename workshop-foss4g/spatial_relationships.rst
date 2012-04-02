.. _spatial_relationships:

Partie 10 : Les relations spatiales
=================================

Jusqu'à présent, nous avons utilisé uniquement des fonctions qui permettent de mesurer (:command:`ST_Area`, :command:`ST_Length`), de sérialiser (:command:`ST_GeomFromText`) ou désérialiser (:command:`ST_AsGML`) des géométries. Ces fonctions sont toutes utilisées sur une géométrie à la fois.

Les base de données spatiales sont puissantes car elle ne se contentent pas de stocker les géométries, elle peuvent aussi vérifier les *relations entre les géométries*.

Pour les questions comme "Quel est le plus proche garage à vélo prêt du parc ?" ou "Ou est l'intersection du métro avec telle rue ?", nous devrons comparer les géométries représentant les garage à vélo, les rues et les lignes de métro.

Le standard de l'OGC définit l'ensemble de fonctions suivant pour comparer les géométries.

ST_Equals
---------
 
:command:`ST_Equals(geometry A, geometry B)` teste l'égalité spatiale de deux géométries. 

.. figure:: ./spatial_relationships/st_equals.png
   :align: center

ST_Equals retourne TRUE si les deux géométries sont du même type et ont des coordonnées x.y identiques.

Premièrement, essayons de récupérer la représentation d'un point de notre table ``nyc_subway_stations``. Nous ne prendrons que l'entrée : 'Broad St'.

.. code-block:: sql

  SELECT name, the_geom, ST_AsText(the_geom)
  FROM nyc_subway_stations 
  WHERE name = 'Broad St';             

::

     name   |                      the_geom                      |      st_astext
  ----------+----------------------------------------------------+-----------------------
   Broad St | 0101000020266900000EEBD4CF27CF2141BC17D69516315141 | POINT(583571 4506714)
 
Maintenant, copiez / collez la valeur affichée pour tester la fonction :command:`ST_Equals`:

.. code-block:: sql

  SELECT name 
  FROM nyc_subway_stations 
  WHERE ST_Equals(the_geom, '0101000020266900000EEBD4CF27CF2141BC17D69516315141');

::

   Broad St

.. note::

  La représentation du point n'est pas vraiment compréhensible (``0101000020266900000EEBD4CF27CF2141BC17D69516315141``) mais c'est exactement la représentation des coordonnées. Pour tester l'égalité, l'utilisation de ce format est nécessaire.


ST_Intersects, ST_Disjoint, ST_Crosses et ST_Overlaps
------------------------------------------------------

:command:`ST_Intersects`, :command:`ST_Crosses`, et :command:`ST_Overlaps` teste si l'intérieur des géométries s'intersecte, se croise ou se chevauche.

.. figure:: ./spatial_relationships/st_intersects.png
   :align: center

:command:`ST_Intersects(geometry A, geometry B)` retourne t (TRUE) si l'intersection ne renvoit pas un ensemble vide de résultats. Intersects retourne le résultat exactement inverse de la fonction disjoint.

.. figure:: ./spatial_relationships/st_disjoint.png
   :align: center

L'opposé de ST_Intersects est :command:`ST_Disjoint(geometry A , geometry B)`. Si deux géométries sont disjointes, elle ne s'intersectent pas et vice-versa. En fait, il est souvent plus éfficace de tester si deux géométries ne s'intersectent pas que de tester si elles sont dijointes du fait que le test d'intersection peut être spatialement indexé alors que le test disjoint ne le peut pas.

.. figure:: ./spatial_relationships/st_crosses.png  
   :align: center

Pour les comparaisons de couples de types multipoint/polygon, multipoint/linestring, linestring/linestring, linestring/polygon, et linestring/multipolygon, :command:`ST_Crosses(geometry A, geometry B)` retourne t (TRUE) si les résultats de l'intersection sont à l'intérieur des deux géométries.

.. figure:: ./spatial_relationships/st_overlaps.png
   :align: center

:command:`ST_Overlaps(geometry A, geometry B)` compare deux géométries de même dimension et retourne TRUE si leur intersection est une géométrie différente des deux fournies mais de même dimension.

Essayons de prendre la station de métro de Broad Street et de déterminer sont voisinage en utilisant la fonction :command:`ST_Intersects` :

.. code-block:: sql

  SELECT name, boroname 
  FROM nyc_neighborhoods
  WHERE ST_Intersects(the_geom, '0101000020266900000EEBD4CF27CF2141BC17D69516315141');

::

          name        | boroname  
  --------------------+-----------
   Financial District | Manhattan



ST_Touches
----------

:command:`ST_Touches` teste si deux géométries se touchent en leur contours extérieurs, mais leur contours intérieurs ne s'intersectent pas

.. figure:: ./spatial_relationships/st_touches.png
   :align: center

:command:`ST_Touches(geometry A, geometry B)` retourn TRUE soit si les contours des géométries s'intersectent ou si l'un des contours intérieurs de l'une intersecte le contour extérieur de l'autre.

ST_Within et ST_Contains
-------------------------

:command:`ST_Within` et :command:`ST_Contains` test si une géométrie est totalement incluse dans l'autre. 

.. figure:: ./spatial_relationships/st_within.png
   :align: center
    
:command:`ST_Within(geometry A , geometry B)` retourne TRUE si la première géométrie est complètement contenue dans l'autre. ST_Within test l'exact opposé au résultat de ST_Contains.  

:command:`ST_Contains(geometry A, geometry B)` retourne TRUE si la seconde géométrie est complètement contenue dans la première géométrie.


ST_Distance et ST_DWithin
--------------------------

Une question fréquente dans le domaine du SIG est "trouver tout les éléments qui se trouvent à une distance X de cet autre élément".

La fonction :command:`ST_Distance(geometry A, geometry B)` calcule la *plus courte* distance entre deux géométries. Cela est pratique pour récupérer la distance entre les objets.

.. code-block:: sql

  SELECT ST_Distance(
    ST_GeometryFromText('POINT(0 5)'),
    ST_GeometryFromText('LINESTRING(-2 2, 2 2)'));

::

  3

Pour tester si deux objets sont à la même distance d'un autre, la fonction :command:`ST_DWithin` fournit un test tirant profit des indexes. Cela est très utile pour répondre a une question telle que: "Combien d'arbre se situent dans un buffer de 500 mètres autour de cette route ?". Vous n'avez pas à calculer le buffer, vous avez simplement besoin de tester la distance entre les géométries.

  .. figure:: ./spatial_relationships/st_dwithin.png
     :align: center
    
En utilisant de nouveau notre station de métro Broad Street, nous pouvons trouver les rues voisines (à 10 mètres de) de la station :

.. code-block:: sql

  SELECT name 
  FROM nyc_streets 
  WHERE ST_DWithin(
          the_geom, 
          '0101000020266900000EEBD4CF27CF2141BC17D69516315141', 
          10
        );

:: 

       name     
  --------------
     Wall St
     Broad St
     Nassau St

Nous pouvons vérifier la réponse sur une carte. La station Broad St est actuellement à l'intersection des rues Wall, Broad et Nassau. 

.. image:: ./spatial_relationships/broad_st.jpg

Liste des fonctions
-------------------

`ST_Contains(geometry A, geometry B) <http://postgis.org/docs/ST_Contains.html>`_ : retourne TRUE si aucun des points de B n'est à l'extérieur de A, et au moins un point de l'intérieur de B est à l'intérieur de A.

`ST_Crosses(geometry A, geometry B)  <http://postgis.org/docs/ST_Crosses.html>`_ : retourne TRUE si la géométrie A a certains, mais pas la totalité, de ses points à l'intérieur de B.

`ST_Disjoint(geometry A , geometry B) <http://postgis.org/docs/ST_Disjoint.html>`_ : retourne TRUE si les gémétries nes s'intersectent pas - elles n'ont aucun point en commun.

`ST_Distance(geometry A, geometry B)  <http://postgis.org/docs/ST_Distance.html>`_ : retourne la distance cartésienne en 2 dimensions minimum entre deux géométries dans l'unité de la projection. 

`ST_DWithin(geometry A, geometry B, radius) <http://postgis.org/docs/ST_DWithin.html>`_ : retourne TRUE si les géométries sont distante (radius) l'une de l'autre. 

`ST_Equals(geometry A, geometry B) <http://postgis.org/docs/ST_Equals.html>`_ : retourn TRUE si les géométries fournis représentent la même géométrie. L'ordre des entités n'est pas prit en compte.

`ST_Intersects(geometry A, geometry B) <http://postgis.org/docs/ST_Intersects.html>`_ : retourne TRUE si les géométries s'intersectent - (ont un espace en commun) et FALSE si elles n'en ont pas (elles sont disjointes). 

`ST_Overlaps(geometry A, geometry B) <http://postgis.org/docs/ST_Overlaps.html>`_ : retourne TRUE si les géométries ont un espace en commun, sont de la même dimension, mais ne sont pas complètement contenues l'une dans l'autre.

`ST_Touches(geometry A, geometry B)  <http://postgis.org/docs/ST_Touches.html>`_ : retourne TRUE si les géométries ont au moins un point en commun, mais leur intérieurs ne s'intersectent pas.

`ST_Within(geometry A , geometry B) <http://postgis.org/docs/ST_Within.html>`_ : retourne TRUE si la géométrie A est complètement à l'intérieur de B



