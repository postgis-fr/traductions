==============================================================================================================
Installation et prérequis
==============================================================================================================

Pour ces travaux pratiques, vous aurez besoin de :

* un serveur web comme Apache avec le support PHP d'activé (et le module PostgreSQL de PHP)
* De préférence un système d'exploitation de type Linux comme Ubuntu
* Un éditeur de texte comme Gedit
* Une connexion internet

Tout les outils requis sont disponibles sur l'OSGeo LiveDVD, donc les références qui suivent représentent un rapide résumé de comment l'installer sur votre propre machine tournant sur Ubuntu 10.04 ou suppérieur.

--------------------------------------------------------------------------------------------------------------
pgRouting
--------------------------------------------------------------------------------------------------------------

L'installation de pgRouting sur Ubuntu est devenue extrêmement simple du fait de la disponibilité d'un paquet binaire dans le `dépot Launchpad <https://launchpad.net/~georepublic/+archive/pgrouting>`_: . 

Tout ce que vous avez à faire maintenant c'est d'ouvrir une fenêtre de terminal et de lancer :

.. code-block:: bash
	
	# Ajouter le dépôt launchpad
	sudo add-apt-repository ppa:georepublic/pgrouting
	sudo apt-get update

	# Installer les paquets pgRouting
	sudo apt-get install gaul-devel \
		postgresql-8.4-pgrouting \
		postgresql-8.4-pgrouting-dd \
		postgresql-8.4-pgrouting-tsp

	# Installer le paquet osm2pgrouting
	sudo apt-get install osm2pgrouting

	# Installé le contenu des travaux pratiques (optionel)
	sudo apt-get install pgrouting-workshop

Cela installera aussi tout les paquets dépendants comme PostgreSQL et PostGIS s'ils ne sont pas déjà installés.

.. note::

	* Les paquets "multiverse" doivent être disponibles comme des sources de logiciels. Actuellement les paquets pour Ubuntu 10.04 à 11.04 sont disponibles.
	* Pour prendre en compte les nouveaux dépôts et avoir une liste des tout deniers paquets à jour, vous devez lancer ``sudo apt-get update && sudo apt-get upgrade`` de temps en temps, tout spécialement si vous utilisez une ancienne version du LiveDVD.
	* Afin d'éviter les problèmes de permissions, vous pouvez utiliser la méthode de connexion ``trust`` dans ``/etc/postgresql/8.4/main/pg_hba.conf`` et redémarrer le serveur PostgreSQL avec ``sudo service postgresql-8.4 restart``.
	
--------------------------------------------------------------------------------------------------------------
Travaux pratiques
--------------------------------------------------------------------------------------------------------------

Suite à l'installation du paquet workshop, vous touverez tout les documents dans ``/usr/share/pgrouting/workshop/``.

Nous recommendons de copier l'ensemble de ces fichiers dans le répertoire de votre utilisateur et de créer un lient symbolique vers votre serveur web :

.. code-block:: bash
	
	cp -R /usr/share/pgrouting/workshop ~/Desktop/pgrouting-workshop
	sudo ln -s ~/Desktop/pgrouting-workshop /var/www/pgrouting-workshop

Vous pouvez ensuite trouver les fichiers des travaux pratiques dans le répertoire ``pgrouting-workshop`` et y accéder :

* Répertoire Web : http://localhost/pgrouting-workshop/web/
* Manuel en ligne : http://localhost/pgrouting-workshop/docs/html/

.. note::

	Des exemples de données additionelles sont disponibles dans le répertoire ``data`` des travaux pratique. Ils contiennent un fichier compressé contenant les sauvegardes de base de données ainsi qu'un plus petit ensemble de données du réseau routier du centre ville de Denver. Pour décompresser ce fichier, exécuter la commande ``tar -xzf ~/Desktop/pgrouting-workshop/data.tar.gz``.


--------------------------------------------------------------------------------------------------------------
Base de données à partir de modèle
--------------------------------------------------------------------------------------------------------------

C'est une bonne idée de créer un modèle de bases de données PostGIS et pgRouting. Cela rendra plus facile la création de nouvelles bases de données incluant déjà les fonctionnalités requises, sans avoir à charger les fichiers SQL pour chaque nouvelle base.

Un script est disponible dans le répertoire ``bin`` des travaux pratiques pour ajouter des modèles de bases de données incluant les fonctionnalités de PostGIS et pgRouting. Pour créer une base de données modèles, exécutez les commandes suivantes depuis une fenêtre de terminal :

.. code-block:: bash
	
	cd ~/Desktop/pgrouting-workshop
	bash bin/create_templates.sh

Maintenant vous pouvez créer une nouvelle base incluant les fonctionnalités pgRouting en utilsant ``template_routing`` comme modèle. Lancez la commande suivante dans une fenêtre de terminal :

.. code-block:: bash
	
	# Création de la base de données "routing"
	createdb -U postgres -T template_routing routing

Vous povez aussi utiliser **PgAdmin III** et des commandes SQL. Démarrez PgAdmin III (disponible sur le LiveDVD), connectez-vous à n'importe quelle base de données et ouvrez l'éditeur SQL afin de lancer les commandes SQL suivantes :

.. code-block:: sql

	-- Création de la base routing
	CREATE DATABASE "routing" TEMPLATE "template_routing";


.. _installation_load_functions:

--------------------------------------------------------------------------------------------------------------
Chargement des functions
--------------------------------------------------------------------------------------------------------------

Sans une base de données modèle, de nombreux fichiers contenant les fonctions de pgRouting doivent être chargés dans la base. Pour procéder de la sorte, utilsez les commandes suivantes depuis une fenêtre de terminal :

.. code-block:: bash

	# Passer en utilisateur "postgres" (ou lancez, en tant qu'utilisateur "postgres")
	sudo su postgres

	# Création d'un base routing
	createdb routing
	createlang plpgsql routing

	# Ajouter les fonctions PostGIS
	psql -d routing -f /usr/share/postgresql/8.4/contrib/postgis-1.5/postgis.sql
	psql -d routing -f /usr/share/postgresql/8.4/contrib/postgis-1.5/spatial_ref_sys.sql

	# Ajouter les fonctions de base de pgRouting
	psql -d routing -f /usr/share/postlbs/routing_core.sql
	psql -d routing -f /usr/share/postlbs/routing_core_wrappers.sql
	psql -d routing -f /usr/share/postlbs/routing_topology.sql
	
Encore un fois, vous pouvez utiliser **PgAdmin III** et des commandes SQL. Démarrez PgAdmin III, connextez-vous à n'importe quelle base de données, ouvrez l'éditeur de commande SQL et saisissez les commandes suivantes :

.. code-block:: sql

	-- Création de la base routing
	CREATE DATABASE "routing";
	
Connectez-vous ensuite à la base ``routing`` et ouvrez une nouvelle fenêtre d'éditeur SQL :
	
.. code-block:: sql

	-- Ajouter le support plpgsql et les fonctions PostGIS/pgRouting
	CREATE PROCEDURAL LANGUAGE plpgsql;

Maintenant, ouvrez les fichiers ``.sql`` contenant les fonctions PostGIS/pgRouting listée précédemment et chargez les dans la base de données ``routing``.
	
.. note::

	PostGIS ``.sql`` files can be stored in different directories. The exact location depends on your version of PostGIS and PostgreSQL. The example above is valid for PostgeSQL/PostGIS version 1.5 installed on OSGeo LiveDVD.
	

--------------------------------------------------------------------------------------------------------------
Données
--------------------------------------------------------------------------------------------------------------

Les travaux pratiques pgRouting utiliseront les données de Denver d'OpenStreetMap, quisont déjà disponibles sur le LiveDVD. Si vous n'utilisez pas le LiveDV ou si vous voulez télécharger les dernières données ou des données de votre choix, vous pouvez utiliser l'API OpenStreetMap depuis votre fenêtre de terminal :

.. code-block:: bash
	
	# Télécharger le fichier sampledata.osm
	wget --progress=dot:mega -O "sampledata.osm"  
		"http://jxapi.openstreetmap.org/xapi/api/0.6/*
						[bbox=-105.2147,39.5506,-104.594,39.9139]"

L'API a une limite de taille de téléchargment, ce qui peut être problématique pour télécharger une grande étendu géographique avec de nombreux éléments. Une alternative est d'utiliser  `l'éditeur JOSM <http://josm.openstreetmap.de>`_, qui utilisera aussi des appels à l'API pour télécharger les données, mais il fournit un interface facile d'utilisation pour les utilisateurs. Vous pouvez sauvegarder les données come un fichier ``.osm`` pour l'utiliser avec ces travaux pratiques. JSOM est aussi disponible sur le LiveDVD.

.. note::

	* OpenStreetMap API v0.6, voir pour plus d'informations http://wiki.openstreetmap.org/index.php/OSM_Protocol_Version_0.6
	* Les données de Denver sont disponibles sur le LiveDVD dans le répertoire ``/usr/local/share/osm/``

Une alternative, pour de très grandes étendues est d'utiliser le service de téléchargement de `CloudMade <http://www.cloudemade.com>`_. Cette entreprise offre des extractions de cartes pour tous les pays du monde. Pour les données du Colorado par exemple, allez sur le page http://download.cloudmade.com/americas/northern_america/united_states/colorado et téléchargez le fichier compressé ``.osm.bz2`` :

.. code-block:: bash

	wget --progress=dot:mega http://download.cloudmade.com/americas/northern_america/united_states/colorado/colorado.osm.bz2
	
.. warning::

	Les données d'un pays complet peuvent être trop grande par rapport à l'espace disponible sur le LiveDVD et nécessité des temps de calculs extrêmement long.  
	






