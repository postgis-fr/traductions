==============================================================================================================
Outils d'import osm2pgrouting
==============================================================================================================

**osm2pgrouting** est un outils en ligne de commande qui rend simple l'importation de données OpenStreetMap dans une base de données pgRouting. Il contruit le réseau routier automatiquement et crée les tables pour les types de données et les classes de routes. osm2pgrouting a été écrit initialement par Daniel Wendt et est maintenant disponible sur le site du projet pgRouting : http://www.pgrouting.org/docs/tools/osm2pgrouting.html

.. note::

	Il y a quelques limitations, particulièrement par rapport à la taille du réseay. La version actuelle nécessite le chargement en mémoire de l'ensemble des données, ce qui le rend rapide mais consome aussi beaucoup de mémoire pour les gros enesemble d'objets. Un outils alternatif n'ayant pas de limitation sur la taille du réseauest **osm2po** (http://osm2po.de). Il est disponible sous licence "Freeware License".
	

Les données brutres d'OpenStreetMap contiennent bien plus d'éléments et d'informations qu'il est nécessaire pour du routage. Ainsi le format n'est pas utilisable tel-quel avec pgRouting. Un fichier XML ``.osm`` contient trois types de données majeurs :

* noeuds
* chemins
* relations

Les données de sampledata.osm par exemple ressemble à ce qui suit :

.. literalinclude:: code/osm_sample.osm
	:language: xml

Une description détaillée de tout les types et classes possibles d'OpenStreetMap peuvent-être trouvé ici : http://wiki.openstreetmap.org/index.php/Map_features.

Lorsuqe vous utilisez osm2pgrouting, nous devons conserver uniquement les noeuds et les chemin ayant pour types et classes celles stipulée dans le fichier ``mapconfig.xml`` qui seront improtés dans notre base de données routing :

.. literalinclude:: code/mapconfig_sample.xml
	:language: xml

Le fichier ``mapconfig.xml`` par défaut est installé dans le répertoire : ``/usr/share/osm2pgrouting/``.


-------------------------------------------------------------------------------------------------------------
Création de la base de données routing
-------------------------------------------------------------------------------------------------------------

Avant de lancer osm2pgrouting nous devons créer la base de données et y charger les fonctionalités de PostGIS et pgRouting .
Si vous avez installé le modèle de base de données comme décrit dans le chapitre précédent, créer une base de données prête à utiliser pgRouting est fait par une simple commande. Ouvrez une fenêtre de terminal et utiliser la commande suivante : 

.. code-block:: bash

	createdb -U postgres -T template_routing routing
	
... vous avez terminé.

Une alternative consiste à  utiliser l'outil **PgAdmin III** et des requêtes SQL. Démarrez  PgAdmin III (disponible sur le LiveDVD), connectez vous à une base de données eyt ouvrez l'éditeur de requêtes afin d'y saisir les requêtes SQL suivantes :

.. code-block:: sql

	-- Créationde la base routing
	CREATE DATABASE "routing" TEMPLATE "template_routing";


Sinon, vous devez manuellement charger les différents fichier dans la base de données. Voir : :ref:`le chapitre précédent <installation_load_functions>`.

	
-------------------------------------------------------------------------------------------------------------
Utiliser osm2pgrouting
-------------------------------------------------------------------------------------------------------------

La prochaine étape c'est de lancer l'outil ``osm2pgrouting``, qui est un outil en ligne de commande, donc vous devrez l'utiliser depuis une fenêtre de terminal.

Nous prendrons par défaut le fichier de configuration ``mapconfig.xml`` et la base de données ``routing`` que nous avons créer précédemment. De plus nous prendrons le fichier ``~/Desktop/pgrouting-workshop/data/sampledata.osm`` comme données brutes. Ce fichier contient seulement les données OSM du centre ville de Denver afin d'accélérer le processus de chargement des données.

Les données sont disponibles au format compressé, qui doit donc être décompressé soit en utlisant un navigateur de fichiers soit en utilisant la commande suivante :

.. code-block:: bash

	cd ~/Desktop/pgrouting-workshop/
	tar -xvzf data.tar.gz

Lancer ensuite l'outil :
	
.. code-block:: bash

	osm2pgrouting -file "data/sampledata.osm" \
				  -conf "/usr/share/osm2pgrouting/mapconfig.xml" \
				  -dbname routing \
				  -user postgres \
				  -clean
					
Liste des paramètres possible :

.. list-table::
   :widths: 15 15 60 10

   * - **Paramètre**
     - **Valeur**
     - **Déscription**
     - **Requis**
   * - -file
     - <fichier>
     - le nom du fichier XML .osm
     - yes
   * - -dbname
     - <nom_de_base>
     - le nom de votre base de données
     - yes
   * - -user
     - <utilisateur>
     - le nom de l'utilisateur, qui a le droit d'accès en écriture sur la base de données
     - yes
   * - -conf
     - <fichier>
     - le nom du fichier XML de configuration
     - yes
   * - -host
     - <hôte>
     - l'hôte de votre base de données postgresql (par défaut : 127.0.0.1)
     - no
   * - -port
     - <port>
     - le numéro de port de votre serveur de base de données(par défaut: 5432)
     - no
   * - -passwd
     - <mot_de_passe>
     - le mot de passe pour se connecter à la base de données
     - no
   * - -clean
     - 
     - Suprrimer les tables précédemment créées
     - no

.. note::

	Si vous obtenez un message d'erreur relatif aux droits de l'utilisateur postgres, vous pouvez définir la méthode comme ``trust`` dans le fichier  ``/etc/postgresql/8.4/main/pg_hba.conf`` et redémarrer le serveur PostgreSQL avec la commande ``sudo service postgresql-8.4 restart``.


Suivant la taille de votre réseau le temps de cacul et d'importation de données peut être long. Une fois terminé, connectez à votre base de données et vérifiez les tables qui aurait du être créées :

.. rubric:: Lancer: ``psql -U postgres -d routing -c "\d"``	

Si tout se passe bien, le résultat devrait ressembler à ceci :
	
.. code-block:: sql

		             List of relations
	 Schema |        Name         |   Type   |  Owner   
	--------+---------------------+----------+----------
	 public | classes             | table    | postgres
	 public | geometry_columns    | table    | postgres
	 public | nodes               | table    | postgres
	 public | spatial_ref_sys     | table    | postgres
	 public | types               | table    | postgres
	 public | vertices_tmp        | table    | postgres
	 public | vertices_tmp_id_seq | sequence | postgres
	 public | ways                | table    | postgres
	(8 rows)

	

