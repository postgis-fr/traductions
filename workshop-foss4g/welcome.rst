.. _welcome:

Bienvenue
*********

Conventions d'écriture
======================

Cette section présente les différentes conventions d'écriture qui seront utilisées dans ce
document afin d'en faciliter la lecture. 

Indications
-----------

Les indications pour vous, lecteurs de ce document, seront noté en **gras**.

Par exemple:

  Cliquez sur **Suivant** pour continuer.

Code
----

Les exemples de requêtes SQL seront affichées de la manière suivante :

.. code-block:: sql

   SELECT postgis_full_version();

Cet exemple peut être saisi dans la fenêtre de requêtage ou depuis l'interface en ligne de commande.

Notes
-----

Les notes sont utilisées pour fournir une information utile mais non critique pour la 
compréhension globale du sujet traité.

.. note:: Si vous n'avez pas manger une pomme aujourd'hui, le docteur devrait se 
    mettre en route.

Fonctions
---------

Lorsque les noms de fonctions seront contenu dans une phrase, ils seront affiché en  :command:`gras`.

Par exemple:

   :command:`ST_Touches(geometry A, geometry B)` retourne vrai si un des contours des géométries s'intersectent

Fichiers, Tables et nom de colonne
----------------------------------

Les nom de fichier, les chemins, le noms de tables et les noms de colones seront affiché comme suit

   Select the ``name`` column in the ``nyc_streets`` table.

Menus et formulaires
-----------------------

Les menus et les éléments de formulaire comme les champs ou les boîtes à cocher ainsi 
que les autre objets sont affichés en *italique*.

Par exemple:

  Cliquez sur *Fichier > Nouveau*.  Cochez la case qui contient *Confirmer*.

Organisation
------------

Les différentes sections de ce document permettent d'évoluer progressivement. Chaque 
section suppose que vous avez terminé et compris les sections précédentes

Certaines sections fournissent des exemples fonctionnels aisni que des exercices. Dans certains cas, il y'a aussi des sections "Les choses à essayer" pour les curieux. Ces tâches contiennent des problèmes plus complexes que dans les exercices.