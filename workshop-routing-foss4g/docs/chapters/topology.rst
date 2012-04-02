==============================================================================================================
Création de la topologie du réseau
==============================================================================================================

:doc:`osm2pgrouting <osm2pgrouting>` est un outil pratique, mais c'est aussi une *boîte noire*. Il y a de nombreux cas où :doc:`osm2pgrouting <osm2pgrouting>` ne peut pas être utilisé. Certaines données de réseau sont fournies avec la topologie du réseau qui peut être utilisé par pgRouting tel-quel. Certaines données de réseau sont stockées au format Shape file (``.shp``) et nous pouvons les charger dans une base de données PostgreSQL à l'aide de l'outil de conversion de PostGIS' ``shape2postgresql`. But what to do then?

.. image:: images/network.png
	:width: 250pt
	:align: center

Dans ce chapitre vous allez apprendre comment créer une topologie de réseau en partant de rien. Pour ce faire, nous allons commencer par les données qui contiennent les attributs minimum requis pour le routage et comment constituer étape par étape des données pour pgRouting.

-------------------------------------------------------------------------------------------------------------
Charger les données de réseau
-------------------------------------------------------------------------------------------------------------

Au début nous allors charger une sauvegarde de base de données à partir du répertoire ``data``des travxu pratiques. Ce répertoire contient un fichier compressé incluant une sauvegarde de base de données ainsi qu'un plus petit ensemble de données de réseau du centre ville de Denver. Si vous n'avez pas encore décompressé, faite le en utilisant la comande :

.. code-block:: bash

	cd ~/Desktop/pgrouting-workshop/
	tar -xvzf data.tar.gz

La commande suivante permet d'importer la sauvegarde de la base de données. Elle ajoutera les fonctions PostGIS et pgRouting à la base, de la même manière que ce nous avons décrit dans le chapitre précédent. Cela chargera aussi le petit échantillon de données de Denver avec un nombre minimum d'attribut, que vous trouverez habituellement dans l'ensemble des données de réseau :

.. code-block:: bash

	# Optionel: supprimer la base de données
	dropdb -U postgres pgrouting-workshop

	# Chargement du fichier de sauvegarde
	psql -U postgres -f ~/Desktop/pgrouting-workshop/data/sampledata_notopo.sql

Regardons quelles tables ont été créées :

.. rubric:: Lancer : ``psql -U postgres -d pgrouting-workshop -c "\d"``
	
.. code-block:: sql

		          List of relations
	 Schema |       Name        | Type  |  Owner   
	--------+-------------------+-------+----------
	 public | classes           | table | postgres
	 public | geography_columns | view  | postgres
	 public | geometry_columns  | table | postgres
	 public | spatial_ref_sys   | table | postgres
	 public | types             | table | postgres
	 public | ways              | table | postgres
	(6 rows)

	
La table contenant les données du réseau routier onle nom ``ways``. Elle possède les attributs suivants :
	
.. rubric:: Lancer : ``psql -U postgres -d pgrouting-workshop -c "\d ways"``
	
.. code-block:: sql

		       Table "public.ways"
	  Column  |       Type       | Modifiers 
	----------+------------------+-----------
	 gid      | integer          | not null
	 class_id | integer          | 
	 length   | double precision | 
	 name     | character(200)   | 
	 the_geom | geometry         | 
	Indexes:
	    "ways_pkey" PRIMARY KEY, btree (gid)
	    "geom_idx" gist (the_geom)
	Check constraints:
	    "enforce_dims_the_geom" CHECK (ndims(the_geom) = 2)
	    "enforce_geotype_the_geom" CHECK (geometrytype(the_geom) = 
	              'MULTILINESTRING'::text OR the_geom IS NULL)
	    "enforce_srid_the_geom" CHECK (srid(the_geom) = 4326)

Il est habituel dans des données de réseau routier de retrouver au moins les informations suivantes :

* Identifiant de tronçon routier (gid)
* Classe de tronçon (class_id)
* Longuer du tronçon routier (length)
* Nom du tronçon (name)
* La géométrie du tronçon (the_geom)

Cela permet d'afficher le réseau routier comme une couche PostGIS depuis un logiciel SIG, par exemple dans QGIS. Notez ue les informations ne suffisent pas au calcul de routes étant donné qu'il ne contient aucune information relative à la topolgie du réseau.

La prochaine étape consiste à démarrer l'outil en ligne de commande PostgreSQL

.. code-block:: bash

	psql -U postgres pgrouting-workshop
	
... ou d'utiliser PgAdmin III.


--------------------------------------------------------------------------------------------------------------
Calcul de la topologie
--------------------------------------------------------------------------------------------------------------

Pour avoir vos données importé dans une base de données PostgreSQL requière généralement des étapes supplémentaires pour pgRouting. Vous devez vous assurer que vos données fournissent une topologie correcte du réseau, ce qui correspond aux informations par rapport au début et à la fin d'un tronçon.

Si les données de votre réseau ont une déjà telle information vous devez exécuter la fonctions ``assign_vertex_id``. Cette fonction permet l'assignation des valeurs pour les colonnes ``source`` et ``target`` pour chaque tronçon et il peut prendre en compte le fait qu'un sommet puisse être éloigné d'un autre suivant une certaine tolérance.

.. code-block:: sql

	assign_vertex_id('<table>', float tolerance, '<geometry column', '<gid>')
	
Premièrement nous devons ajouter les colonnes source et target, pour ensuite utiliser la fonction assign_vertex_id ... et attendre :

.. code-block:: sql

	-- Ajouter les colonnes "source" et "target"
	ALTER TABLE ways ADD COLUMN "source" integer;
	ALTER TABLE ways ADD COLUMN "target" integer;
	
	-- Utiliser la fonction de contruction de topologie
	SELECT assign_vertex_id('ways', 0.00001, 'the_geom', 'gid');

.. note::

	Exécuter ``psql -U postgres -d pgrouting-workshop`` depuis votre terminal afin de vous connecter ààl a base de données et lancer des commandes PostgreSQL en ligne. Quiter la session avec la commande ``\q`` .   

.. warning::

	La dimension du paramètre tolérance dépends du système de projection de vos données. Habituellement c'est soit "degrés" soit "mètres".


-------------------------------------------------------------------------------------------------------------
Ajouter des indexes
-------------------------------------------------------------------------------------------------------------

Heureusement nous n'avons pas à attendre longtemps étant donné que notre jeu de données est très petit. Mais la quantité de données d'un réseau pourrait être beaucoup plus importante, donc il vaut mieux ajouter des indexes pour les colonnes  ``source`` et ``target``.

.. code-block:: sql

	CREATE INDEX source_idx ON ways("source");
	CREATE INDEX target_idx ON ways("target");

Suite à ces étapes, notre base de données routing ressemble à ceci :

.. rubric:: Lancer : ``\d``
	
.. code-block:: sql

		             List of relations
	 Schema |        Name         |   Type   |  Owner   
	--------+---------------------+----------+----------
	 public | geography_columns   | view     | postgres
	 public | geometry_columns    | table    | postgres
	 public | spatial_ref_sys     | table    | postgres
	 public | vertices_tmp        | table    | postgres
	 public | vertices_tmp_id_seq | sequence | postgres
	 public | ways                | table    | postgres
	(6 rows)

.. rubric:: Lancer : ``\d ways``
	
.. code-block:: sql
	
		       Table "public.ways"
	  Column  |       Type       | Modifiers 
	----------+------------------+-----------
	 gid      | integer          | not null
	 class_id | integer          | 
	 length   | double precision | 
	 name     | character(200)   | 
	 the_geom | geometry         | 
	 source   | integer          | 
	 target   | integer          | 
	Indexes:
	    "ways_pkey" PRIMARY KEY, btree (gid)
	    "geom_idx" gist (the_geom)
	    "source_idx" btree (source)
	    "target_idx" btree (target)
	Check constraints:
	    "enforce_dims_the_geom" CHECK (ndims(the_geom) = 2)
	    "enforce_geotype_the_geom" CHECK (geometrytype(the_geom) = 
	                'MULTILINESTRING'::text OR the_geom IS NULL)
	    "enforce_srid_the_geom" CHECK (srid(the_geom) = 4326)
		
Nous sommes fin prêts pour notre première requête de routage avec `l'algorithme de Dijkstra <shortest_path>` !
