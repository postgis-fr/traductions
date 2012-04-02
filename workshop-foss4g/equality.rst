.. _equality:

Partie 22 : Égalité
=================================

Égalité
--------

Être en mesure de déterminer si deux geométries sont égales peut être compliqué. PostGIS met à votre disposition différentes fonctions permettant de juger de l'égalité à différents niveaux, bien que pour des raison de simplicité nous nuos contenterons ici de la définition fournie plus bas. Pour illustrer ces fonctions, nous utiliseront les polygones suivants.

.. image:: ./equality/polygon-table.png

Ces polygones sont charger à l'aide des commandes suivantes.

.. code-block:: sql

  CREATE TABLE polygons (name varchar, poly geometry);
  
  INSERT INTO polygons VALUES 
    ('Polygon 1', 'POLYGON((-1 1.732,1 1.732,2 0,1 -1.732,
        -1 -1.732,-2 0,-1 1.732))'),
    ('Polygon 2', 'POLYGON((-1 1.732,-2 0,-1 -1.732,1 -1.732,
        2 0,1 1.732,-1 1.732))'),
    ('Polygon 3', 'POLYGON((1 -1.732,2 0,1 1.732,-1 1.732,
        -2 0,-1 -1.732,1 -1.732))'),
    ('Polygon 4', 'POLYGON((-1 1.732,0 1.732, 1 1.732,1.5 0.866,
        2 0,1.5 -0.866,1 -1.732,0 -1.732,-1 -1.732,-1.5 -0.866,
        -2 0,-1.5 0.866,-1 1.732))'),
    ('Polygon 5', 'POLYGON((-2 -1.732,2 -1.732,2 1.732, 
        -2 1.732,-2 -1.732))');
        
   SELECT Populate_Geometry_Columns();

.. image:: ./equality/start13.png

Exactement égaux
^^^^^^^^^^^^^^^^^^

L'égalité exacte est déterminée en comparant deux géométries, sommets par sommets, dans l'ordre, pour s'assurer que chacun est à une position identique. Les exemples suivant montrent comment cette méthode peut être limitée dans son éfficacité.

.. code-block:: sql

  SELECT a.name, b.name, CASE WHEN ST_OrderingEquals(a.poly, b.poly)
      THEN 'Exactly Equal' ELSE 'Not Exactly Equal' end
    FROM polygons as a, polygons as b;

.. image:: ./equality/start14.png

Dans cette exemple, les polygones sont seulement égaux à eux-même, mais jamais avec un des autres polygones (dans notre exemple les polygones de 1 à 3). Dans le cas des polygones 1, 2 et 3, les sommets sont à des position identiques mais sont définies dans un ordre différent. Le polygone 4 a des sommets en double causant la non-égalité avec le polygone 1.

Spatiallement égaux
^^^^^^^^^^^^^^^

Comme nous l'avons précédemment, l'égalité exacte ne prend pas en compte la nature spatiale des géométries. Il y a une fonction, nommée :command:`ST_Equals`, permettant de tester l'égalité spatiale ou l'équivalent des géométries.

.. code-block:: sql

  SELECT a.name, b.name, CASE WHEN ST_Equals(a.poly, b.poly) 
      THEN 'Spatially Equal' ELSE 'Not Equal' end
    FROM polygons as a, polygons as b;

.. image:: ./equality/start15.png

Ces résultats sont plus proches de notre compréhension intuitive de l'égalité. Les polygones de 1 à 4 sont cosidérés comme égaux, puisque qu'elles recouvrent la même zone. Notez que ni la direction despolygones n'est considérée, le point de départ pour la définition du polygone, ni le nombre de points. Ce qui importe c'est que la zone géographique représentée est la même.

Égalité des étendues
^^^^^^^^^^^^^^^^^^^^^

L'égalité exacte nécessite, dans le pire des cas, de comparer chaqu'un  des sommets d'une géométrie pour déterminé l'égalité. Ceci peut être très lent, et s'avérer innaproprié pour comparer un grand nombre de géométries. Pour permettre de rendre plus rapide ces comparaison, l'opération d'égalité des étendue est fournit :  :command:`=`. Cet opérateur utilise uniquement les étendues (cadre limite rectangulaire), assurant que les géométries occupent le même espace dans un repère cartésien en deux dimensions, mais ne représente pas nécessairement le même espace.

.. code-block:: sql

  SELECT a.name, b.name, CASE WHEN a.poly = b.poly 
      THEN 'Equal Bounds' ELSE 'Non-equal Bounds' end
    FROM polygons as a, polygons as b;

.. image:: ./equality/start17.png

Comme vous pouvez le constater, toutes les géométries égales ont aussi une étendue égales. Malheureusement, le polygone 5 est aussi retourné comme étant égal avec ce test, puisqu'il partage la même étendue que les autres géométries. Mais alors, pourquoi est-ce utile ? Bien que cela soit traité en détail plus tard, la réponse courte est que cela permet l'utilisation d'indexation spatiales qui peuvent réduire drastiquement les ensembles de géométries à comparrer en utilisant des filtres utilisant cette égalité d'étendues.

