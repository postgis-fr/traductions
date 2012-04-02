.. _joins:

Partie 12 : Les jointures spatiales
===================================

Les jointures spatiales sont la cerise sur le gâteau des base de données spatiales. Elles vous pemettent de combiner les informations de plusieures tables en utilisant une relation spatiale comme clause de jointure. La plupart des "analyses SIG standards" peuvent être exprimées à l'aide de jointure spatiales.

Dans la partie précédente, nous avons utilisé les relations spatiales en utilisant deux étapes dans nos requêtes : nous avons dans un premier temps extrait la station de métro "Broad St" puis nous avons utilisé ce résultat dans nos autres requêtes pour répondre aux questions comme "dans quel quartier se situe la station 'Broad St' ?"

En utilisant les jointures spatiales, nous pouvons répondre aux questions en une seule étape, récupérant les informations relatives à la station de métro et le quartier la contenant : 

.. code-block:: sql

  SELECT 
    subways.name AS subway_name, 
    neighborhoods.name AS neighborhood_name, 
    neighborhoods.boroname AS borough
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_subway_stations AS subways
  ON ST_Contains(neighborhoods.the_geom, subways.the_geom)
  WHERE subways.name = 'Broad St';

:: 

   subway_name | neighborhood_name  |  borough  
  -------------+--------------------+-----------
   Broad St    | Financial District | Manhattan

Nous avons pu regrouper chaque station de métro avec le quartier auquel elle appartient, mais dans ce cas nous n'en voulions qu'une. Chaque fonction qui envoit un résultat du type vrai/faux peut être utilisée pour joindre spatialement deux tables, mais la plupart du temps on utilise : :command:`ST_Intersects`, :command:`ST_Contains`, et :command:`ST_DWithin`.

Jointure et regroupement
------------------------

La combinaison de ``JOIN`` avec ``GROUP BY`` fournit le type d'analyse qui est couramment utilisé dans les systèmes SIG.

Par exemple : **Quelle est la population et la répartition raciale du quartier de Manhattan ?** Ici nous avons une question qui combine les informations relatives à la population recensée et les contours des quartiers, or nous ne voulons qu'un seul quartier, celui de Manhattan.

.. code-block:: sql

  SELECT 
    neighborhoods.name AS neighborhood_name, 
    Sum(census.popn_total) AS population,
    Round(100.0 * Sum(census.popn_white) / Sum(census.popn_total),1) AS white_pct,
    Round(100.0 * Sum(census.popn_black) / Sum(census.popn_total),1) AS black_pct
  FROM nyc_neighborhoods AS neighborhoods
  JOIN nyc_census_blocks AS census
  ON ST_Intersects(neighborhoods.the_geom, census.the_geom)
  WHERE neighborhoods.boroname = 'Manhattan'
  GROUP BY neighborhoods.name
  ORDER BY white_pct DESC;

::

   neighborhood_name  | population | white_pct | black_pct 
 ---------------------+------------+-----------+-----------
  Carnegie Hill       |      19909 |      91.6 |       1.5
  North Sutton Area   |      21413 |      90.3 |       1.2
  West Village        |      27141 |      88.1 |       2.7
  Upper East Side     |     201301 |      87.8 |       2.5
  Greenwich Village   |      57047 |      84.1 |       3.3
  Soho                |      15371 |      84.1 |       3.3
  Murray Hill         |      27669 |      79.2 |       2.3
  Gramercy            |      97264 |      77.8 |       5.6
  Central Park        |      49284 |      77.8 |      10.4
  Tribeca             |      13601 |      77.2 |       5.5
  Midtown             |      70412 |      75.9 |       5.1
  Chelsea             |      51773 |      74.7 |       7.4
  Battery Park        |       9928 |      74.1 |       4.9
  Upper West Side     |     212499 |      73.3 |      10.4
  Financial District  |      17279 |      71.3 |       5.3
  Clinton             |      26347 |      64.6 |      10.3
  East Village        |      77448 |      61.4 |       9.7
  Garment District    |       6900 |      51.1 |       8.6
  Morningside Heights |      41499 |      50.2 |      24.8
  Little Italy        |      14178 |      39.4 |       1.2
  Yorkville           |      57800 |      31.2 |      33.3
  Inwood              |      50922 |      29.3 |      14.9
  Lower East Side     |     104690 |      28.3 |       9.0
  Washington Heights  |     187198 |      26.9 |      16.3
  East Harlem         |      62279 |      20.2 |      46.2
  Hamilton Heights    |      71133 |      14.6 |      41.1
  Chinatown           |      18195 |      10.3 |       4.2
  Harlem              |     125501 |       5.7 |      80.5


Que ce passe-t-il ici ?  Voici ce qui se passe (l'ordre d'évaluation est optimisé par la base de données) :

#. La clause ``JOIN`` crée une table virtuelle qui contient les colonnes à la fois des quartiers et des recensements (tables neighborhoods et census).
#. La clause ``WHERE`` filtre la table virtuelle pour ne conserver que la ligne correspondant à Manhattan. 
#. Les lignes restantes sont regroupées par le nom du quartier et sont utilisées par la fonction d'agrégation : :command:`Sum()` pour réaliser la somme des valeurs de la populations.
#. Après un peu d'arythmétique et de formatage (ex: ``GROUP BY``, ``ORDER BY``)) sur le nombres finaux, notre requête calcul les pourcentages.

.. note:: 

   La clause ``JOIN`` combine deux parties ``FROM``.  Par défaut, nous utilisons un jointure du type :``INNER JOIN``, mais il existe quatres autres types de jointures. Pour de plus amples informations à ce sujet, consultez la partie `type_jointure <http://docs.postgresql.fr/9.1/sql-select.html>`_ de la page de la documentation officielle de PostgreSQL.

Nous pouvons aussi utiliser le test de la distance dans notre clef de jointure, pour créer une regroupement de "tout les éléments dans un certain rayon". Essayons d'analyser la géographie raciale de New York en utilisant les requêtes de distance.

Premièrement, essayons d'obtenir la répartition raciale de la ville.

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks;

:: 

        white_pct      |      black_pct      | popn_total 
  ---------------------+---------------------+------------
   44.6586020115685295 | 26.5945063345703034 |    8008278


Donc, 8M de personnes dans New York, environ 44% sont "blancs" et 26% sont "noirs".

Duke Ellington chantait que "You / must take the A-train / To / go to Sugar Hill way up in Harlem." Comme nous l'avons vu précédemment, Harlem est de très loin le quartier ou se trouve la plus grande concentration d'africains-américains de Manhattan (80.5%). Est-il toujours vrai qu'il faut prendre le train A dont Duke parlait dans sa chanson ?

Premièrement, le contenu du champ ``routes`` de la table ``nyc_subway_stations`` va nous servir à récupérer le train A. Les valeurs de ce champs sont un peu complexes.

.. code-block:: sql

  SELECT DISTINCT routes FROM nyc_subway_stations;
  
:: 

 A,C,G
 4,5
 D,F,N,Q
 5
 E,F
 E,J,Z
 R,W

.. note::

   Le mot clef ``DISTINCT`` permet d'éliminer les répétitions de lignes de notre résultat. Dans ce mot clef, notre requête renverrait 491 résultats au lieu de 73.
   
Donc pour trouver le train A, nous allons demander toutes les lignes ayant pour ``routes`` la valeur 'A'. Nous pouvons faire cela de différentes manières, mais nous utiliserons aujourd'hui le fait que la fonction :command:`strpos(routes,'A')` retourne un entier différent de 0 si la lettre 'A' se trouve dans la valeur du champs route.

.. code-block:: sql

   SELECT DISTINCT routes 
   FROM nyc_subway_stations AS subways 
   WHERE strpos(subways.routes,'A') > 0;
   
::

  A,B,C
  A,C
  A
  A,C,G
  A,C,E,L
  A,S
  A,C,F
  A,B,C,D
  A,C,E
  
Essayons de regrouper la répartition raciale dans un rayon de 200 mètres de la ligne du train A.

.. code-block:: sql

  SELECT 
    100.0 * Sum(popn_white) / Sum(popn_total) AS white_pct, 
    100.0 * Sum(popn_black) / Sum(popn_total) AS black_pct, 
    Sum(popn_total) AS popn_total
  FROM nyc_census_blocks AS census
  JOIN nyc_subway_stations AS subways
  ON ST_DWithin(census.the_geom, subways.the_geom, 200)
  WHERE strpos(subways.routes,'A') > 0;

::

        white_pct      |      black_pct      | popn_total 
  ---------------------+---------------------+------------
   42.0805466940877366 | 23.0936148851067964 |     185259

La répartition raciale le long de la ligne du train A n'est pas radicallement différente de la répartition générale de la ville de New York.

Jointures avancées
------------------

Dans la dernière partie nous avons vu que le train A n'est pas utilisé par des populations si éloignées de la répartition totale du reste de la ville. Y-a-t-il des train qui passent par des parties de la ville qui ne sont pas dans la moyenne de la répartition raciale ?

Pour répondre à cette question, nous ajouterons une nouvelle jointure à notre requête, de telle manière que nous puissions calculer simultanément la répartition raciale de plusieures lignes de métro à la fois. Pour faire ceci, nous créerons une table qui permettra d'énumérer toutes les lignes que nous voulons regrouper.

.. code-block:: sql

    CREATE TABLE subway_lines ( route char(1) );
    INSERT INTO subway_lines (route) VALUES 
      ('A'),('B'),('C'),('D'),('E'),('F'),('G'),
      ('J'),('L'),('M'),('N'),('Q'),('R'),('S'),
      ('Z'),('1'),('2'),('3'),('4'),('5'),('6'),
      ('7');

Maintenant nous pouvons joindre les tables des lignes de métros à notre requête précédente.

.. code-block:: sql

    SELECT 
      lines.route,
      Round(100.0 * Sum(popn_white) / Sum(popn_total), 1) AS white_pct, 
      Round(100.0 * Sum(popn_black) / Sum(popn_total), 1) AS black_pct, 
      Sum(popn_total) AS popn_total
    FROM nyc_census_blocks AS census
    JOIN nyc_subway_stations AS subways
    ON ST_DWithin(census.the_geom, subways.the_geom, 200)
    JOIN subway_lines AS lines
    ON strpos(subways.routes, lines.route) > 0
    GROUP BY lines.route
    ORDER BY black_pct DESC;

::

     route | white_pct | black_pct | popn_total 
    -------+-----------+-----------+------------
     S     |      30.1 |      59.5 |      32730
     3     |      34.3 |      51.8 |     201888
     2     |      33.6 |      45.5 |     535414
     5     |      32.1 |      45.1 |     407324
     C     |      41.3 |      35.9 |     430194
     4     |      34.7 |      30.9 |     328292
     B     |      36.1 |      30.6 |     261186
     Q     |      52.9 |      26.3 |     259820
     J     |      29.5 |      23.6 |     126764
     A     |      42.1 |      23.1 |     370518
     Z     |      29.5 |      21.5 |      81493
     D     |      39.8 |      20.9 |     233855
     G     |      44.8 |      20.0 |     138602
     L     |      53.9 |      17.1 |     104140
     6     |      52.7 |      16.3 |     257769
     1     |      54.8 |      12.6 |     659028
     F     |      60.0 |       8.6 |     438212
     M     |      50.0 |       7.8 |     166721
     E     |      69.4 |       5.3 |      86118
     R     |      57.7 |       4.8 |     389124
     7     |      42.4 |       3.8 |     107543


Comme précédemment, les jointures créent une table virtuelle de toutes les combinaisons possibles et disponibles à l'aide des contraintes de type ``JOIN ON`. Ces lignes sont ensuite utilisées dans le regroupement ``GROUP``. La magie spatiale tiend dans l'utilisation de la fonction ``ST_DWithin`` qui s'assure que les blocs sont suffisamment proches des lignes de métros inclues dans le calcul.

Liste de fonctions
------------------

`ST_Contains(geometry A, geometry B) <http://postgis.org/docs/ST_Contains.html>`_: retourne TRUE si et seulement si aucun point de B est à l'extérieur de A, et si au moins un point à l'intérieur de B  est à l'intérieur de A.

`ST_DWithin(geometry A, geometry B, radius) <http://postgis.org/docs/ST_DWithin.html>`_: retourne TRUE si les géométries sont distantes du rayon donné. 

`ST_Intersects(geometry A, geometry B) <http://postgis.org/docs/ST_Intersects.html>`_: retourne TRUE si les géométries/géographies "s'intersectent spatialement" (partage une portiond de l'espace) et FALSE sinon (elles sont dijointes). 

`round(v numeric, s integer) <http://www.postgresql.org/docs/7.4/interactive/functions-math.html>`_: fonction de PostgreSQL qui arrondit à s décimales.

`strpos(chaîne, sous-chaîne) <http://www.postgresql.org/docs/current/static/functions-string.html>`_: fonction de chaîne de caractères de PostgreSQL qui retourne la position de la sous-chaine.

`sum(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: fonction d'agrégation de PostgreSQL qui retourne la somme d'un ensemble de valeurs.

.. rubric:: Footnotes

.. [#PostGIS_Doco] http://postgis.org/documentation/manual-1.5/

