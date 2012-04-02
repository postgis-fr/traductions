==============================================================================================================
Requêtes de routage avancées
==============================================================================================================

Comme expliqué dans le chapitre précédent une requête de recherche de plus court chemine ressemble généralement à ce qui suit :

.. code-block:: sql

	SELECT * FROM shortest_path_shooting_star(
		'SELECT gid as id, source, target, length as cost, x1, y1, x2, y2, rule, 
		to_cost, reverse_cost FROM ways', 6585, 8247, true, true);
	
On parle généralement de **plus court** chemin, ce qui signifie que la longueur d'un arc est son coût. Mais le coût n'est pas nécessairement une longueur, il peut représenter n'importe quoi, par exemple le temps, la surface, le type de routes, etc ... Ou il peut être la combinaison de plusieurs paramètres ("coûts pondérés").

.. note::

	Si vous souhaitez continuer avec une base de données contenant les fonctions pgRouting, les données exemples ainsi que les attributs nécessaires, vous pouvez charger le fichier de sauvegarded la manière suivante.

.. code-block:: bash

	# Optionel: supprimer la base de données
	dropdb -U postgres pgrouting-workshop

	# Charger le fichier de sauvegarde
	psql -U postgres -f ~/Desktop/pgrouting-workshop/data/sampledata_routing.sql

-------------------------------------------------------------------------------------------------------------
Coûts pondérés
-------------------------------------------------------------------------------------------------------------

Dans un vrai réseau il y a différents types de limitations ou de préférences suivant les types de routes par exemple. En d'autre termes, nous ne voulons pas calculer *le plus court* chemin mais le chemin *le moins cher* - un chemin avec un coût minimum. Il n'y aucune limitation dans ce qui peut êtreutilsié pour définir le coût.

Lorsque nous avons convertis les données au format OSM en utilisant l'outil osm2pgrouting, nous avons deux autres tables permettant de déifinir les ``types`` de routes et les ``classes``.

.. note::

	Nous passons maintenant à la base de données que nuos avons générée avec osm2pgrouting. Depuis l'invite de commandes de PostgreSQL ceci est possible avec la commande ``\c routing``.

.. rubric:: Lancer : ``SELECT * FROM types;``

.. code-block:: sql

	  id |   name    
	-----+------------
	   2 | cycleway
	   1 | highway
	   4 | junction
	   3 | tracktype
   
.. rubric:: Lancer : ``SELECT * FROM classes;``

.. code-block:: sql

	 id  | type_id |        name        |  cost 
	-----+---------+--------------------+--------
	 201 |       2 | lane               |     
	 204 |       2 | opposite           |     
	 203 |       2 | opposite_lane      |    
	 202 |       2 | track              |     
	 117 |       1 | bridleway          |     
	 113 |       1 | bus_guideway       |     
	 118 |       1 | byway              |     
	 115 |       1 | cicleway           |     
	 116 |       1 | footway            |     
	 108 |       1 | living_street      |     
	 101 |       1 | motorway           |    
	 103 |       1 | motorway_junction  |     
	 102 |       1 | motorway_link      |     
	 114 |       1 | path               |     
	 111 |       1 | pedestrian         |     
	 106 |       1 | primary            |     
	 107 |       1 | primary_link       |     
	 107 |       1 | residential        |     
	 100 |       1 | road               |     
	 100 |       1 | unclassified       |     
	 106 |       1 | secondary          |    
	 109 |       1 | service            |     
	 112 |       1 | services           |     
	 119 |       1 | steps              |     
	 107 |       1 | tertiary           |     
	 110 |       1 | track              |     
	 104 |       1 | trunk              |     
	 105 |       1 | trunk_link         |     
	 401 |       4 | roundabout         |     
	 301 |       3 | grade1             |     
	 302 |       3 | grade2             |     
	 303 |       3 | grade3             |     
	 304 |       3 | grade4             |     
	 305 |       3 | grade5             |     

La classe de route est liée avec la tables des cheminspar le champ ``class_id``. Suite à l'importation des données la valeur de la colonne ``cost`` n'est pas encore attribuée. Sa valeur peut être modifiée à l'aide d'une requête ``UPDATE``. Dans cet exemple les valeurs de coût pour la table des classe sont assigné de façon arbitraire, donc nous exécutons :

.. code-block:: sql

	UPDATE classes SET cost=1 ;
	UPDATE classes SET cost=2.0 WHERE name IN ('pedestrian','steps','footway');
	UPDATE classes SET cost=1.5 WHERE name IN ('cicleway','living_street','path');
	UPDATE classes SET cost=0.8 WHERE name IN ('secondary','tertiary');
	UPDATE classes SET cost=0.6 WHERE name IN ('primary','primary_link');
	UPDATE classes SET cost=0.4 WHERE name IN ('trunk','trunk_link');
	UPDATE classes SET cost=0.3 WHERE name IN ('motorway','motorway_junction','motorway_link');

Pour de meilleures performances, tout spécialement si le réseau est important, il est préférable de créer un index sur la colonnes ``class_id`` de la table des chemins et eventuellement le champ ``id`` de la table ``types``.

.. code-block:: sql

	CREATE INDEX ways_class_idx ON ways (class_id);
	CREATE INDEX classes_idx ON classes (id);

L'idée de ces deux tables est de les utiliser afin de spécifier un facteur qui sera multiplié par le coût de parcour d'un tronçon (habituellement la longueur) :

.. code-block:: sql

	SELECT * FROM shortest_path_shooting_star(
		'SELECT gid as id, class_id, source, target, length*c.cost as cost, 
			x1, y1, x2, y2, rule, to_cost, reverse_cost*c.cost as reverse_cost 
		FROM ways w, classes c 
		WHERE class_id=c.id', 6585, 8247, true, true);

-------------------------------------------------------------------------------------------------------------
Restriction d'accès
-------------------------------------------------------------------------------------------------------------

Une autre possibilité est de restreindre l'accès à des routes d'un certains types soit en affectant un coût très élevé à un tronçon ayant un certain attribut soit en s'assurant de ne sélectionner aucun de ces tronçons :

.. code-block:: sql

	UPDATE classes SET cost=100000 WHERE name LIKE 'motorway%';

En utilisant des sous-requêtes vous pouvez "mixer" vos coût comme bon vous semble et cela modifiera le résultat obtenu imédiatement. Les changements de coûts affecteront la prochaine recherche de plus courts chemins, sans avoir à reconstruire le votre réseau.

Bien entendu, certaines classes de tronçon peuvent aussi être exclues à l'aide d'une clause ``WHERE`` dans la requête, par exemple pour exclure la classe "living_street" :

.. code-block:: sql

	SELECT * FROM shortest_path_shooting_star(
		'SELECT gid as id, class_id, source, target, length*c.cost as cost, 
			x1, y1, x2, y2, rule, to_cost, reverse_cost*c.cost as reverse_cost 
		FROM ways w, classes c 
		WHERE class_id=c.id AND class_id != 111', 6585, 8247, true, true);

Bien entendu, pgRouting vus permet tout types de requêtes SQL supportées par PostgreSQL/PostGIS.
 
