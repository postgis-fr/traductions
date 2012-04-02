.. _loading_data:

Partie 4 : Charger des données spatiales
=========================================

Supporté par une grande variété de librairies et d'applications, PostGIS fournit de nombreux outils pour charger des données. Cette partie traitera uniquement du chargement basique de données, c'est à dire le chargement de fichiers Shapefile (.shp) en utilisant l'outil dédié de PostGIS.

#. Premièrement, retournez sur le Dashboard et cliquez sur le lien **Import shapefiles** de la section PostGIS. L'interface d'import de données Shapefile pgShapeLoader se lance.

   .. image:: ./screenshots/pgshapeloader_01.png

#. Ensuite, ouvrez le navigateur de fichier *Shape File* puis dans le répertoire file:`\\postgisintro\\data` sélectionnez le fichier :file:`nyc_census_blocks.shp`. 

#. Saisissez les détails de la section *connexion PostGIS* et cliquez sur le bouton **Test Connection...**.

   .. list-table::

      * - **Username**
        - ``postgres``
      * - **Password**
        - ``postgres``
      * - **Server Host**
        - ``localhost`` ``54321``
      * - **Database**
        - ``nyc``

  .. note:: 
  
     Affecter le numéro de port **54321** est très important ! Le serveur PostGIS d'OpenGeo utilise ce port et non le port par défaut (5432).

#. Saisissez les détails de la section *Configuration*.

   .. list-table::

      * - **Destination Schema**
        - ``public``
      * - **SRID**
        - ``26918``
      * - **Destination Table**
        - ``nyc_census_blocks``
      * - **Geometry Column**
        - ``the_geom``

#. Cliquez sur le bouton **Options** et sélectionnez "Load data using COPY rather than INSERT." Ce qui implique que le chargement des données sera plus rapide.

   .. image:: ./screenshots/pgshapeloader_02.png

#. Pour finir, cliquez sur le bouton **Import** et regardez l'importation s'exécuter. Cela peut prendre plusieurs minutes pour charger, mais ce fichier est le plus gros que nous aurons à charger.

#. Repétez la méthode afin d'importer les autres données présentes dans le répertoire data. Ormis le nom du fichier et le nom de la table de sortie, les autres paramètres de pgShapeLoader devrait rester les mêmes :

   * ``nyc_streets.shp``
   * ``nyc_neighborhoods.shp``
   * ``nyc_subway_stations.shp``
 
#. Lorsque tout les fichiers sont chargés, cliquez sur le bouton "Refresh" de pgAdmin pour mettre à jour l'arbre affiché. Vous devriez voir vos quatre nouvellles tables affichées dans la section **Tables** de l'arbre.

   .. image:: ./screenshots/refresh.png
 
 
Shapefile ? Qu'est-ce que c'est ?
-------------------------------

Il est possible que vous vous demandiez "Qu'est-ce que c'est ce shapefile ?" On utilise communément le terme "Shapefile" pour parler d'un ensemble de fichiers d'extension ``.shp``, ``.shx``, ``.dbf``, ou autre ayant un nom commun (ex: nyc_census_blocks). Le fichier Shapefile est en réalité le fichier d'extension ``.shp``, mais ce fichier seul n'est pas complet sans ses fichiers associés.

Fichiers obligatoires :

  * ``.shp`` — les formes; les entités géographiques elle-mêmes
  * ``.shx`` — l'index de formes; un index base sur les positions des entités géographiques
  * ``.dbf`` — les attributs; les données attributaires associées à chaque formes, au format dBase III
    
Les fichiers optionels possibles:

  * ``.prj`` — la projection; le système de coordonnées et l'information de projection, un fichier texte décrivant la projection utilisant le format text bien connu (WKT)

Afin d'utiliser un fichier Shapefile dans PostGIS, vous devez le convertir en une série de requêtes SQL. En utilisant pgShapeLoader, un Shapefile est converti en une table que PostgreSQL peut comprendre.


SRID 26918 ? Qu'est que c'est ?
-------------------------------

La plupart des paramètres de l'importation de données sont explicites mais même les proffessionnels du SIG peuvent rencontrer des difficulté à propos du  **SRID**.

"SRID" signifie "IDentifiant de Référénce Spatiale". Il définit tout les paramètres de nos données, telles les coordonnées géographiques et la projection. Un SRID est pratique car il encapsule sous la forme d'un nombre toutes les informations à propos de la projection de la carte (ce qui peut être très compliqué).

Vou pouvez consulter la définition de la projection de la carte en consultant la base de données en ligne suivante :

  http://spatialreference.org/ref/epsg/26918/

ou directement depuis PostGIS en interrogeant la table ``spatial_ref_sys``.

.. code-block:: sql

  SELECT srtext FROM spatial_ref_sys WHERE srid = 26918;
  
.. note::

   La table ``spatial_ref_sys`` de PostGIS est une table standard OGC qui définit tout les systèmes de références spatiales connus par la base de données. Les données livrées avec PostGIS, contiennent 3000 systèmes de références spatiales et précisent les informations nécessaires à la tranformation ou la reprojection.  
   
Dans les deux cas, vous obtiendrez une représentation du système de références spatiales **26918** (affiché sur plusieurs lignes ici pour plus de clarté).

::

  PROJCS["NAD83 / UTM zone 18N",
    GEOGCS["NAD83",
      DATUM["North_American_Datum_1983",
        SPHEROID["GRS 1980",6378137,298.257222101,AUTHORITY["EPSG","7019"]],
        AUTHORITY["EPSG","6269"]],
      PRIMEM["Greenwich",0,AUTHORITY["EPSG","8901"]],
      UNIT["degree",0.01745329251994328,AUTHORITY["EPSG","9122"]],
      AUTHORITY["EPSG","4269"]],
    UNIT["metre",1,AUTHORITY["EPSG","9001"]],
    PROJECTION["Transverse_Mercator"],
    PARAMETER["latitude_of_origin",0],
    PARAMETER["central_meridian",-75],
    PARAMETER["scale_factor",0.9996],
    PARAMETER["false_easting",500000],
    PARAMETER["false_northing",0],
    AUTHORITY["EPSG","26918"],
    AXIS["Easting",EAST],
    AXIS["Northing",NORTH]]

Si vous ouvrez le fichier ``nyc_neighborhoods.prj`` du répertoire data, vous verrez la même définition.

Un problème auquel se confronte la plupart des débutants en PostGIS est de savoir quel SRID il doit utiliser pour ses données. Tout ce qu'ils ont c'est un fichier ``.prj``. Mais comment un humain peut-il reconnaitre le numéro de SRID correct en lisant le contenu du fichier ``.prj`` ?

La réponse simple est d'utiliser un ordinateur. Copiez le contenu du fichier ``.prj`` dans le formulaire du site http://prj2epsg.org. Cela vous donnera le nombre (ou la liste de nombres) qui correspond le plus à votre définition de projection. Il n'y a pas de nombre pour *toutes* les projections de cartes existantes dans le monde, mais les plus courants sont disponibles dans la base de données de prj2epsg.

.. image:: ./screenshots/prj2epsg_01.png

Les données que vous recevez des agences locales de l'état - comme la ville de New York - utilisent la plupart du temps des projections locales noté "state plane" ou "UTM". Dans notre cas, la projection est "Universal Transverse Mercator (UTM) Zone 18 North" soit EPSG:26918.  


Les choses à essayer : Rendre spatiale une base de données existante
--------------------------------------------------------------------

Vous avez déjà vue comment créer une base de données en utilisant le modèle ``postgis_template`` depuis pgAdmin. Néanmoins, lorsque vous installé depuis les sources ou que vous ajoutez le module PostGIS à une base existante, il n'est pas toujours approprié de créer une nouvelle base de données en utilisant le modèle PostGIS.

Votre tâche consiste dans cette section à créer une base de données et à ajouter les types et les fonctions PostGIS ensuite. Les script SQL nécessaires - :file:`postgis.sql` et :file:`spatial_ref_sys.sql` - se trouve dans le répertoire :file:`contrib` de votre installation de PostgreSQL. Pour vous guider, vous pouvez consulter la documentation PostGIS expliquant comment installer PostGIS [#PostGIS_Install]_.

.. note::

   N'oubliez pas saisir le nom de l'utilisateur et le numéro de port losque vous créer une base de données en ligne de commande.
    
Les choses à essayer : Visualiser des données avec uDig
------------------------------------------------------

`uDig <http://udig.refractions.org>`_, (User-friendly Desktop Internet GIS) est un outil bureautique de visualisation/edition SIG permettant de visualiser rapidement se données. Vous pouvez visualiser un grand nombre de formats différents dont les Shapefiles et les bases de données PostGIS. Son interface graphique vous permet d'explorer vos données facilement mais aussi de les tester et les styler rapidement.

Utilisez cette application pour vous connecter à votre base de données PostGIS. L'application est contenu dans le répertoire ``software``.

.. rubric:: Footnotes

.. [#PostGIS_Install] "Chapter 2.5. Installation" PostGIS Documentation. Mai 2010 <http://postgis.org/documentation/manual-1.5/ch02.html#id2786223>

