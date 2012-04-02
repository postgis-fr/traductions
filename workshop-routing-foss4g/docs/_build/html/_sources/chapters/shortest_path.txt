==============================================================================================================
Plus courts chemins
==============================================================================================================

pgRouting été initialement appelé *pgDijkstra*, puisque il implémentait seulement la recherche de plus court chemin à l'aide de l'agorithme de *Dijkstra*. Plus tard, d'uatres fonctions se sont ajoutées et la bibliotèque fut renommée.

.. image:: images/route.png
	:width: 250pt
	:align: center
	
Ce chapitre explique les trois différents algorithmes et les attributs nécessaires.


.. note::

	Si vous lancez l'outils :doc:`osm2pgrouting <osm2pgrouting>` pour importer des données *OpenStreetMap*, la table des chemins (``ways``) contient déjà tout les attributs nécessaires pour utiliser les fonctions de recherche de plus court chemins. Au contraire, la table ``ways`` de la base de données ``pgrouting-workshop`` du :doc:`chapitre précédent <topology>` manque d'un certain nombre d'attributs, qui sont présentés dans ce chapitre dans les **Prérequis**.


-------------------------------------------------------------------------------------------------------------
Dijkstra
-------------------------------------------------------------------------------------------------------------

L'algorithme de Dijkstraa été la première implémentation disponible dans pgRouting. Il ne nécessite pas d'autre attributs que les champs ``source`` et ``target``, les attributs ``id`` et ``cost``. Il peut être utilisé sur des graphes orientés ou non. Vous pouvez spécifier que votre réseau à un coût de ``parcours inverse`` (``reverse cost``) ou non.

.. rubric:: Prérequis

Pour être en mesure d'utiliser un coût de parcuors invers, vous devez ajouter une colonne de coût. Nous pouvons affecter la longuer au coût de parcours inverse.

.. code-block:: sql

	ALTER TABLE ways ADD COLUMN reverse_cost double precision;
	UPDATE ways SET reverse_cost = length;

.. rubric:: Fonction avec paramètres

.. code-block:: sql

	shortest_path( sql text, 
			   source_id integer, 
			   target_id integer, 
			   directed boolean, 
			   has_reverse_cost boolean ) 

.. note::

	* Les identifiant pour source et target sont les identifiant des noeuds.
	* Graphes non-orientés ("directed=false") implique que le paramètre "has_reverse_cost" est ignoré


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Bases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Chaque algorithme a une fonction de *base*.

.. code-block:: sql

	SELECT * FROM shortest_path('
			SELECT gid as id, 
				 source::integer, 
				 target::integer, 
				 length::double precision as cost 
				FROM ways', 
			5700, 6733, false, false); 

.. code-block:: sql

	 vertex_id | edge_id |        cost         
	-----------+---------+---------------------
	      5700 |    6585 |   0.175725539559539
	      5701 |   18947 |   0.178145491343884
	      2633 |   18948 |   0.177501253416424
	       ... |     ... |                 ...
	      6733 |      -1 |                   0
	 (38 rows)


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Enveloppe
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. rubric:: Enveloppe SANS limite d'étendue

les fonctions enveloppes sont des fonctions qui étendent les fonctions de bases en y ajoutant des transformations, ajoutant des limites de l'étendue géograhpique de la recherhe, etc.. Les enveloppes peuvent changer le format et l'ordre des résultats. Il affectent aussi automatiquement certains paramètre et rend l'utilisation de pgRouting encore plsu simple.

.. code-block:: sql

	SELECT gid, AsText(the_geom) AS the_geom 
		FROM dijkstra_sp('ways', 5700, 6733);
		
.. code-block:: sql
		
	  gid   |                              the_geom      
	--------+---------------------------------------------------------------
	   5534 | MULTILINESTRING((-104.9993415 39.7423284, ... ,-104.9999815 39.7444843))
	   5535 | MULTILINESTRING((-104.9999815 39.7444843, ... ,-105.0001355 39.7457581))
	   5536 | MULTILINESTRING((-105.0001355 39.7457581,-105.0002133 39.7459024))
	    ... | ...
	  19914 | MULTILINESTRING((-104.9981408 39.7320938,-104.9981194 39.7305074))
	(37 rows)


.. note::

	Il est possible de visualiser le chemin dans QGIS. Cela fonctionne pour la requête de recherche du plus court chemin qui retourne une colonne géométrique.

	* Créer la connexion à la base de données et ajoutez la table route comme couche de fond.
	* Ajoutez une autre couche de la table "ways" mais selectionnez l'option ``Build query`` avant de l'ajouter.
	* Saissez ``"gid"  IN ( SELECT gid FROM dijkstra_sp('ways',5700,6733))`` dans le champ **SQL where clause**.
	
	``SQL query`` peut aussi être sélectionné depuis le menu contextuel de la couche. 

	
.. rubric:: Enveloppe AVEC une étendue limite

Vous pouvez limiter votre recherche à une zone précise en ajoutant un cadre limite. Cela améliorera les performances tout spécialement pour les réseaux volumineux.

.. code-block:: sql

	SELECT gid, AsText(the_geom) AS the_geom 
		FROM dijkstra_sp_delta('ways', 5700, 6733, 0.1);
		
.. code-block:: sql

	  gid   |                              the_geom      
	--------+---------------------------------------------------------------
	   5534 | MULTILINESTRING((-104.9993415 39.7423284, ... ,-104.9999815 39.7444843))
	   5535 | MULTILINESTRING((-104.9999815 39.7444843, ... ,-105.0001355 39.7457581))
	   5536 | MULTILINESTRING((-105.0001355 39.7457581,-105.0002133 39.7459024))
	    ... | ...
	  19914 | MULTILINESTRING((-104.9981408 39.7320938,-104.9981194 39.7305074))
	(37 rows)

.. note:: 

	La projection des données OSM est en "degrés", donc nous définirons un cadre limite contenant le point de départ et celui d'arrivée plus une zone tampon de ``0.1`` degrés par exemple.


-------------------------------------------------------------------------------------------------------------
A-Étoile
-------------------------------------------------------------------------------------------------------------

L'algortihme A-Étoile est un autre algrithme bien connu. Il ajoute l'information de la position géographique du début et la fin de chaque tronçon. Cela permet une recherche privilégiant les tronçons proches du point d'arrivée de la recherche.

.. rubric:: Prérequis

Pour A-* vous avez besoin de préparer votre table de réseau et d'y ajouter les colonnes latitute/longitude (``x1``, ``y1`` et ``x2``, ``y2``) et de calculer leur valeurs.

.. code-block:: sql

	ALTER TABLE ways ADD COLUMN x1 double precision;
	ALTER TABLE ways ADD COLUMN y1 double precision;
	ALTER TABLE ways ADD COLUMN x2 double precision;
	ALTER TABLE ways ADD COLUMN y2 double precision;
	
	UPDATE ways SET x1 = x(ST_startpoint(the_geom));
	UPDATE ways SET y1 = y(ST_startpoint(the_geom));
	
	UPDATE ways SET x2 = x(ST_endpoint(the_geom));
	UPDATE ways SET y2 = y(ST_endpoint(the_geom));
	
	UPDATE ways SET x1 = x(ST_PointN(the_geom, 1));
	UPDATE ways SET y1 = y(ST_PointN(the_geom, 1));
	
	UPDATE ways SET x2 = x(ST_PointN(the_geom, ST_NumPoints(the_geom)));
	UPDATE ways SET y2 = y(ST_PointN(the_geom, ST_NumPoints(the_geom)));

.. Note:: 

	La fonction ``endpoint()`` ne fonctionne pas avec certaines versions de PostgresQL (par exemple les version 8.2.5 et 8.1.9). Un moyen de résoudre ce problème consiste à utiliser la fonction ``PointN()`` à la place:


.. rubric:: Fonction avec paramètres

La fonction de recherche de plus court chemin A-* est très semblable à la fonction Dijkstra, bien qu'elle préfère les tronçons qui sont plus proche du point d'arrivée de la recherche. Les heuristiques de cette recherche sont prédéfinis, donc vous aurez besoin de recompiler pgRouting si vous souhaitez modifier ces heuristiques.

.. code-block:: sql

	shortest_path_astar( sql text, 
			   source_id integer, 
			   target_id integer, 
			   directed boolean, 
			   has_reverse_cost boolean ) 

.. note::
	* Les identifiants source et target sont les identifiant des sommets IDs.
	* Graphes non-orienté ("directed=false") ne prennent pas en compte le paramètre "has_reverse_cost"

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Bases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sql

	SELECT * FROM shortest_path_astar('
			SELECT gid as id, 
				 source::integer, 
				 target::integer, 
				 length::double precision as cost, 
				 x1, y1, x2, y2
				FROM ways', 
			5700, 6733, false, false); 
		
.. code-block:: sql
		
	 vertex_id | edge_id |        cost         
	-----------+---------+---------------------
	      5700 |    6585 |   0.175725539559539
	      5701 |   18947 |   0.178145491343884
	      2633 |   18948 |   0.177501253416424
	       ... |     ... |                 ...
	      6733 |      -1 |                   0
	 (38 rows)


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Enveloppe
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. rubric:: Fonction envelopper AVEC cadre limite

.. code-block:: sql

	SELECT gid, AsText(the_geom) AS the_geom 
		FROM astar_sp_delta('ways', 5700, 6733, 0.1);

.. code-block:: sql

	  gid   |                              the_geom      
	--------+---------------------------------------------------------------
	   5534 | MULTILINESTRING((-104.9993415 39.7423284, ... ,-104.9999815 39.7444843))
	   5535 | MULTILINESTRING((-104.9999815 39.7444843, ... ,-105.0001355 39.7457581))
	   5536 | MULTILINESTRING((-105.0001355 39.7457581,-105.0002133 39.7459024))
	    ... | ...
	  19914 | MULTILINESTRING((-104.9981408 39.7320938,-104.9981194 39.7305074))
	(37 rows)

	
.. note::

	* Il n'y a pas actuellement de fonction pour a-* sans cadre limite, étant donnée que le cadre limite permet d'améliorer grandement les performances. Si vous n'avez pas besoin de cadre limite, Dijkstra sera suffisant.


-------------------------------------------------------------------------------------------------------------
Shooting-Star
-------------------------------------------------------------------------------------------------------------

L'algorithme Shooting-Star est le dernier algorithme de recherche de plus court chemin. Sa spécialité c'est qu'il recherche un parcours d'un tronçon à un autre, pas d'un sommet à un sommet comme les agorithmes de Dijkstra et A-Star le font. Cela rend possible la définition de relations entre les tronçons par exemple, et cela résoud certains problèmes liés aux recherches d'un sommets à un autre comme les "tronçons parallèles", qui ont les même sommet de début et de fin mais des coût différents.

.. rubric:: Prérequis

Pour Shooting-Star vous avez besoin de préparer votre table de réseau et d'ajouter les colonnes ``rule`` et ``to_cost``. Come l'algorithme A-* il a aussi un fonction heuristique, qui favorise les tronçons plus proche du point d'arrivée.which prefers links closer to the target of the search.

.. code-block:: sql

	-- Ajouter les colonnes rule et to_cost
	ALTER TABLE ways ADD COLUMN to_cost double precision;	
	ALTER TABLE ways ADD COLUMN rule text;

.. rubric:: L'algorithme Shooting-Star introduit deux nouveaux attributs

.. list-table::
   :widths: 10 90

   * - **Attribut**
     - **Déscription**
   * - rule
     - une chaine de caractères contenant une liste d'identifiants de tronçno séparés par une virgule, qui décrivent le sens giratoir (si vous venez de cet tronçon, vous pouvez rejoindre le suivant en ajoutant un coût défini dans la colonne to_cost)
   * - to_cost
     - un coût pour passer d'un tronçon à un autre (peut être très élevé s'il est interdit de trouner vers ce tronçon ce qui est comparable au coût de parcours d'un tronçon en cas de feu de signalisation)

.. rubric:: Fonction avec paramètres

.. code-block:: sql

	shortest_path_shooting_star( sql text, 
			   source_id integer, 
			   target_id integer, 
			   directed boolean, 
			   has_reverse_cost boolean ) 

.. note::

	* Identifiant du départ et de l'arrivée sont des identifiants de tronçons.
	* Graphes non-orientés ("directed=false") ne prennent pas en compte le paramètre "has_reverse_cost" 

Pour décrire une interdiction de tourner :

.. code-block:: sql

	 gid | source | target | cost | x1 | y1 | x2 | y2 | to_cost | rule
	-----+--------+--------+------+----+----+----+----+---------+------
	  12 |      3 |     10 |    2 |  4 |  3 |  4 |  5 |    1000 | 14
  
... signifie que le coût pour aller du tronçon 14 au tronçon 12 est de 1000, et

.. code-block:: sql

	 gid | source | target | cost | x1 | y1 | x2 | y2 | to_cost | rule
	-----+--------+--------+------+----+----+----+----+---------+------
	  12 |      3 |     10 |    2 |  4 |  3 |  4 |  5 |    1000 | 14, 4

... signifie que le coût pour aller du tronçon 14 au tronçon 12 via le tronçon 4 est de 1000.

Si vous avez besoin de plusieurs restrictions pour une arrête donnée you devez ajouter plusieurs enregistrements pour ce tronçon avec un restriction particulière.

.. code-block:: sql

	 gid | source | target | cost | x1 | y1 | x2 | y2 | to_cost | rule
	-----+--------+--------+------+----+----+----+----+---------+------
	  11 |      3 |     10 |    2 |  4 |  3 |  4 |  5 |    1000 | 4
	  11 |      3 |     10 |    2 |  4 |  3 |  4 |  5 |    1000 | 12

... signifie que le coût pour aller soit du tronçon  4 soit du 12 au 11 est de 1000. Et donc vous devez ordoner vos données par gid lorsque vous chargez vos données dans la fonction de recherche de plus court chemin...


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Bases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Un exemple d'utilisation de l'algorithme Shooting Star : 

.. code-block:: sql

	SELECT * FROM shortest_path_shooting_star('
			SELECT gid as id, 
				 source::integer,
				 target::integer, 
				 length::double precision as cost, 
				 x1, y1, x2, y2,
				 rule, to_cost 
				FROM ways', 
			6585, 8247, false, false); 

.. code-block:: sql

	 vertex_id | edge_id |        cost
	-----------+---------+---------------------
	     15007 |    6585 |   0.175725539559539
	     15009 |   18947 |   0.178145491343884
	      9254 |   18948 |   0.177501253416424
	       ... |     ... |   ...
	      1161 |    8247 |   0.051155648874288
	 (37 rows)

.. warning::

	L'algorithme Shooting Star calcul un chemin d'un tronçon à un autre (pas d'un sommet à un autre). La colonnes vertex_id contienr le point de départ du tronçon de la colonne edge_id.


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Envelopper
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: sql

	SELECT gid, AsText(the_geom) AS the_geom
		FROM shootingstar_sp('ways', 6585, 8247, 0.1, 'length', true, true);

.. code-block:: sql

	  gid   |                              the_geom      
	--------+---------------------------------------------------------------
	   6585 | MULTILINESTRING((-104.9975345 39.7193508,-104.9975487 39.7209311))
	  18947 | MULTILINESTRING((-104.9975487 39.7209311,-104.9975509 39.7225332))
	  18948 | MULTILINESTRING((-104.9975509 39.7225332,-104.9975447 39.7241295))
	    ... | ...
	   8247 | MULTILINESTRING((-104.9978555 39.7495627,-104.9982781 39.7498884))
	(37 rows)

.. note::

	Il n'y a actuellement pas de fonction enveloppe pour Shooting-Star sans cadre limite, puisque le cadre limite améliore grandement les performances. 
