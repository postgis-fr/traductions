.. _simple_sql:

Partie 6 : Requêtes SQL simples
===============================

:term:`SQL`, pour "Structured Query Language", définit la manière d'importer et d'interroger des données dans une base. Vous avez déjà rédigé du SQL lorsque nous avons créer notre première base de données.  Rappel:

.. code-block:: sql

   SELECT postgis_full_version();

Maintenant que nous avons charger des données dans notre base, essayons d'utiliser SQL pour les interroger. Par exemple,

  "Quel sont les noms des quartiers de la ville de New York ?"
  
Ouvrez une fenêtre SQL depuis pgAdmin en cliquant sur le bouton SQL

.. image:: ./screenshots/pgadmin_05.png

Puis saisissez la requête suivante dans la fenêtre

.. code-block:: sql

  SELECT name FROM nyc_neighborhoods;
  
et cliquez sur le bouton **Execute Query** (le triangle vert).
  
.. image:: ./screenshots/pgadmin_08.png  

La requête s'exécutera pendant quelques (mili)secondes et retournera 129 résultats.

.. image:: ./screenshots/pgadmin_09.png  

Mais que c'est-il exactement passé ici ? Pour le comprendre, commençons par présenter les quatre types de requêtes du SQL :

 * ``SELECT``, retourne des lignes en réponse à une requête
 * ``INSERT``, ajoute des lignes dans une table
 * ``UPDATE``, modifie des lignes existantes d'une table
 * ``DELETE``, supprime des lignes d'une table
 
Nous travaillerons principalement avec des requêtes de type ``SELECT``afin d'interroger les tables en utilisant des fonctions spatiales.

Requête de type SELECT
----------------------

Une requête de type Select est généralement de la forme :

  SELECT colonnes FROM données WHERE conditions;
  
.. note::

    Pour une description exhaustive des paramètres possible d'une requête ``SELECT``, consultez la `documentaton de PostgresSQL  <http://www.postgresql.org/docs/8.1/interactive/sql-select.html>`_.
    

Les ``colonnes`` sont soit des noms de colonnes, soit des fonctions utilisant les valeurs des colonnes. Les ``données`` sont soit une table seule, soit plusieures tables reliées ensemble en réalisant une jointure sur une clef ou une autre condition. Les ``conditions`` représentent le filtre qui restreint le nombre de lignes à retourner.

  "Quel sont les noms des quartiers de Brooklyn ?"

Nous retournons à notre table ``nyc_neighborhoods`` avec le filtre en main. La table contient tout les quartiers de New York et nous voulons uniquement ceux de Brooklyn.

.. code-block:: sql

  SELECT name 
    FROM nyc_neighborhoods 
    WHERE boroname = 'Brooklyn';

La requête prendra à nouveau quelque (mili)secondes et retournera les 23 éléments résultants.

Parfois nous aurons besoin d'appliquer des fonctions sur le résultats d'une de nos requêtes. Par exemple,

  "Quel est le nombre de lettres dans les noms des quartiers de Brooklyn ?"
  
Heureusement PostgreSQL fournit une fonction calculant la longueur d'une chaîne de caractères : :command:`char_length(string)`.

.. code-block:: sql

  SELECT char_length(name) 
    FROM nyc_neighborhoods 
    WHERE boroname = 'Brooklyn';

Bien souvent nous sommes moins interressés par une ligne particulière mais plus par un calcul statistique sur l'ensemble résultant. Donc, connaitre la longueur des noms de quartiers est moins interressant que de calculer la moyenne des ces longueurs. Les fonctions qui renvoit un résultat unique en utilisant un ensemble de valeurs sont appelée des "fonctions d'aggrégations".

PostgreSQL fournit un ensemble de fonctions d'aggrégations, parmis lesquelles :command:`avg()` pour calculer la moyenne, and :command:`stddev()` pour l'écart type.

  "Quel est le nombre moyen et l'écart type du nombre de lettre dans le noms des quartier de Brooklyn ?"
  
.. code-block:: sql

  SELECT avg(char_length(name)), stddev(char_length(name)) 
    FROM nyc_neighborhoods 
    WHERE boroname = 'Brooklyn';
  
::

           avg         |       stddev       
  ---------------------+--------------------
   11.7391304347826087 | 3.9105613559407395

Les fonctions d'agrégation dans notre dernier exemple sont appliquées à chaque ligne de l'ensemble des résultats. Comment faire si nous voulons rassembler des données ? Pour cela nous utilisons la clause ``GROUP BY``. Les fonctions d'agrégation ont souvent besoin d'une clause ``GROUP BY`` pour regrouper les éléments en utilisant une ou plusieures colonnes.

  "Quel est la moyenne des les noms de quartier de New York, renvoyer par quartiers ?"

.. code-block:: sql

  SELECT boroname, avg(char_length(name)), stddev(char_length(name)) 
    FROM nyc_neighborhoods 
    GROUP BY boroname;
 

Nous ajoutons la colonne ``boroname`` dans le résultat afin de pouvoir déterminer quelle valeur statistique s'applique à quel quartier. Dans une requête agrégée, vous pouvez seulement retourner les colonnes qui sont (a) membre de la clause de regroupement ou (b) des fonctions d'agrégation.
  
::

     boroname    |         avg         |       stddev       
  ---------------+---------------------+--------------------
   Brooklyn      | 11.7391304347826087 | 3.9105613559407395
   Manhattan     | 11.8214285714285714 | 4.3123729948325257
   The Bronx     | 12.0416666666666667 | 3.6651017740975152
   Queens        | 11.6666666666666667 | 5.0057438272815975
   Staten Island | 12.2916666666666667 | 5.2043390480959474
  
Liste de fonctions
------------------

`avg(expression) <http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: fonction d'agrégation de PostgreSQL  qui retourne la valeur moyenne d'une colonne.

`char_length(string) <http://www.postgresql.org/docs/current/static/functions-string.html>`_: fonction s'applicant aux chaînes de caractère de PostgreSQL qui retourne le nombre de lettres dans une chaîne.

`stddev(expression) <http://www.postgresql.org/docs/current/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-STATISTICS-TABLE>`_: fonction d'aggrégation de PostgreSQL qui retourne l'écart type d'un ensemble de valeurs.
  
  
