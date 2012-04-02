.. _simple_sql_exercises:

Partie 7 : Exercices simples de SQL
===================================

En utilisant la table ``nyc_census_blocks``, répondez au questions suivantes (et n'allez pas directement aux réponses ! ). 

Vous trouverez ci-dessous des informations utiles pour commencer. Référez-vous à la partie :ref:`À propos des nos données` pour la définition de notre table ``nyc_census_blocks``.

.. list-table::
   :widths: 20 80

   * - **blkid**
     - Un code à 15 chiffres qui définit de manière unique chaque **bloc** ressencé . Ex: 360050001009000
   * - **popn_total**
     - Nombre total de personnes dans un bloc ressensé
   * - **popn_white**
     - Nombre de personnes se déclarant "blancs"
   * - **popn_black**
     - Nombre de personnes se déclarant "noirs"
   * - **popn_nativ**
     - Nombre de personnes se déclarant comme "nés aux états-unis"
   * - **popn_asian**
     - Nombre de personne se déclarant comme "asiatiques"
   * - **popn_other**
     - Nombre de personne se déclarant d'une autre catégorie
   * - **hous_total**
     - Nombre de pièces appartements
   * - **hous_own**
     - Nombre de propriétaires occupant les appartements
   * - **hous_rent**
     - Nombre de locations disponibles
   * - **boroname**
     - Nom du quartier de New York. Manhattan, The Bronx, Brooklyn, Staten Island, Queens
   * - **the_geom**
     - Polygone délimitant le bloc

Ici se trouvent certaines des fonctions d'aggrégation qui vous seront utiles pour répondre aux questions :

 * avg() - la moyenne des vlauers dans un ensemble d'enregistrements
 * sum() - la somme des valeurs d'un ensembe d'enregistrements
 * count() - le nombre d'élément contenu dans un ensembe d'enregistrements.

Maintenant les questions :

 * **"Quelle est la population de la ville de New York ?"**
 
   .. code-block:: sql
   
     SELECT Sum(popn_total) AS population
       FROM nyc_census_blocks;
     
   :: 
   
     8008278 
   
   .. note:: 
   
       Qu'est-ce que ce ``AS`` dans la requête ? vous pouvez donner un nom à une table ou a des colonnes en utilisant un alias. Les alias permettent de rendre les requêtes plus simple à écrire et à lire. Donc au lieu que notre colonne résultat soit nommée ``sum`` nous utilisons le  **AS** pour la renommer en ``population``. 
       
 * **"Quelle est la population du Bronx ?"**

   .. code-block:: sql
 
     SELECT Sum(popn_total) AS population
       FROM nyc_census_blocks
       WHERE boroname = 'The Bronx';
     
   :: 
   
     1332650 
   
 * **"Quelle est en moyenne le nombre de personne vivant dans chaque appartement de la ville de New York ?"**
 
   .. code-block:: sql

     SELECT Sum(popn_total)/Sum(hous_total) AS popn_per_house
       FROM nyc_census_blocks;

   :: 
   
     2.6503540522400804 
   
 * **"Pour chaque quartier, quel est le pourcentage de population blanche ?"**

   .. code-block:: sql

     SELECT 
         boroname, 
         100 * Sum(popn_white)/Sum(popn_total) AS white_pct
       FROM nyc_census_blocks
       GROUP BY boroname;

   :: 
   
        boroname    |      white_pct      
     ---------------+---------------------
      Brooklyn      | 41.2005552206888663
      The Bronx     | 29.8655310846808990
      Manhattan     | 54.3594013771837665
      Queens        | 44.0806610271290794
      Staten Island | 77.5968611401579346
 
Liste des fonctions
-------------------

`avg(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: fonction d'aggrégation de PostgreSQL qui renvoit la moyenne d'un esemble de nombres.

`count(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: une fonction d'aggrégation de PostgreSQL qui retourne le nombre d'éléments dans un esemble.

`sum(expression) <http://www.postgresql.org/docs/8.2/static/functions-aggregate.html#FUNCTIONS-AGGREGATE-TABLE>`_: une fonction d'aggrégation de PostgreSQL qui retourne la somme des valeurs numériques d'un ensemble.
