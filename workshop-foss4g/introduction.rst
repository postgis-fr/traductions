.. _introduction:

Partie 1 : Introduction 
************************

Qu'est-ce qu'une base de données spatiales ?
============================================

PostGIS est une base de données spatiale. Oracle Spatial et SQL Server 2008 sont aussi des bases de données spatiales. Mais qu'est-ce que cela signifie? Qu'est-ce qui différencie un serveur de base de données spatiales d'un serveur de base de données non spatiale ?

La réponse courte, est ...

**Les base de données spatiales permettent les stockage et la manipulation des objets spatiaux comme les autres objets de la base de données.**

Ce qui suit présente brièvement l'évolution des base de données spatiales, puis les liens
entre les données spatiales et la base de données (types de données, indexes et fonctions).

#. **Types de données spatiales** fait référence aux géométries de type point, ligne et polygone; 
#. L'**indexation spatiale** est utilisée pour améliorer les performances d'exécution des opérations spatiales;
#. Les **fonctions spatiales**, au sens :term:`SQL`, sont utilisées pour accéder à des propriétés ou à des relations spatiales.

Utilisés de manière combinée, les types de données spatiales, les indexes et les fonctions fournissent une structure flexible pour optimiser les performances et les analyses.

Au commencement
----------------

Dans les premières implémentations :term:`SIG`, toutes les données spatiales étaient stockées sous la forme de fichiers plats et certaines applications :term:`SIG` spécifiques étaient nécessaires pour les interpréter et les manipuler. Ces outils de gestion de première génération avaient été conçus pour répondre aux besoins des utilisateurs pour lesquels toute les données étaient localisées au sein de leur agence. Ces outils propriétaires étaient des systèmes specifiquement créés pour gérer les données spatiales.

La seconde génération des systèmes de gestion de données spatiales stockaient certaines données dans une base de données relationelle (habituellement les "attributs" ou autres parties non spatiales) mais ne founissaient pas encore la fléxibilité offerte par une intégration complète des données spatiales.

**Effectivement, les bases de données spatiales sont nées lorsque les gens ont commencé à considérer les objet spatiaux comme les autres objets d'une base de données .**  

Les bases de données spatiales intègrent les données spatiales sous formes d'objets de la base de données relationelles. Le changement opéré passe d'une vision centrée sur le SIG à une vision centrée sur les bases de données.

.. image:: ./introduction/beginning.png

.. note:: Un système de gestion de base de données peut être utilisé dans d'autres cadres que celui des SIG. Les bases de données spatiales sont utilisées dans divers domaines : l'anatomie humaine, les circuits intégrés de grandes envergures, les structures moléculaires, les champs electro-magnétiques et bien d'autre encore.


Les types de données spatiales
------------------------------

Une base de données classique propose par exemple les types chaînes de caractères et date. Une base de données spatiales ajoute les types de données (spatiales) pour représenter les **entités géographiques**. Ces types de données spatiales permettre d'accéder à des propriétés de l'entité géographique comme ses contours ou sa dimension. Pour bien des aspects, les types de données spatiales peuvent être vu simplement comme des formes.

.. image:: ./introduction/hierarchy.png
   :align: center

Les types de données spatiales sont organisés par une hierarchie de type. Chaque sous-types hérite de la structure (les atrributs) et du comportement (les méthodes et fonctions) de son type supérieur dans la hierarchie.


Indexes spatiaux et étendue
---------------------------

Une base de données ordinaire fournit des "méthodes d'accès" -- connues sous le nom d'**index** -- pour permettre un accès efficace et non séquentiel à un sous ensemble de données. L'indexation des types non géographiques (nombre, chaînes de caractères, dates) est habituellement faite à l'aide des index de type `arbres binaires <http://en.wikipedia.org/wiki/B-tree>`__. Un arbre binaire est un partitionnement des données utilisant l'ordre naturel pour stoquer les données hierarchiequement.

L'ordre naturel des nombres, des chaînes de caractères et des dates est assez simple à déterminer -- chaque valeur est inférieure, plus grande ou égale à toutes les autres valeurs. Mais, étant donné que les polygones peuvent se chevaucher, peuvent être contenu dans un autre et sont représenté par un tableau en deux dimensions (ou plus), un arbre binaire ne convient pas pour indexer les valeurs. Les vraies bases de données spatiales fournissent un "index spatial" qui répond plutôt à la question : "quel objet se trouve dans une étendue spécifique ?"

Une **étendue** correspond au rectangle de plus petite taille capable de contenir un objet géographique.

.. image:: ./introduction/boundingbox.png
   :align: center

Les étendues sont utilisées car répondre à la question : "est-ce que A se trouve à l'intérieur de B ? " est une opération couteuse pour les polygones mais rapide dans le cas ou ce sont des rectangles. Même des polygones et des lignes complexes peuvent être représentés par une simple étendue.

Les index spatiaux doivent réaliser leur ordonnancement rapidement afin d'être utiles. Donc au lieu de fournir des résultats exacts, comme le font les arbres binaires, les index spatiaux fournissent des résultats approximatifs. La question "quelles lignes sont à l'intérieur de ce polygone" sera interprétée par un index spatial comme : "quelles lignes ont une étendue qui est contenue dans l'étendue de ce polygone ?"

Les incréments spatiaux réels mis en application par diverses bases de données varient considérablement.
Les index spatiaux actuellement utilisés par les différents systèmes de gestion de bases de données varient aussi considérablement. L'implémentation la plus commune est l'`arbre R <http://en.wikipedia.org/wiki/R-tree>`_ (utilisé dans PostGIS), mais il existe aussi des implémentations de type `Quadtrees <http://en.wikipedia.org/wiki/Quadtree>`_, et des `indexes basés sur une grille <http://en.wikipedia.org/wiki/Grid_(spatial_index)>`_.

Les Fonctions spatiales
-------------------

Pour manipuler les données lors d'une requête, une base de données classique fournit des **fonctions** comme la concaténation de chaînes de caractères, le cacul de la clef md5 d'une chaîne, la réalisation d'opérations mathématiques sur les nombres ou l'extraction d'informations spécifiques sur une date. Une base de données spatiales fournit un ensemble complet de fonctions pour analyser les composants géographiques, déterminer les relations spatiales et manipuler les objets géographiques. Ces fonctions spatiales sont utilisées comme des pièces de légo pour de nombreux projet SIG.

La majorité des fonctions spatiales peuvent être regroupées dans l'une des cinq catégories suivantes :

#. **Conversion**: fonctions qui *convertissent* les données géographiques dans un format externe. 
#. **Gestion**: fonctions qui permettre de *gérer* les informations relatives  aux tables spatiales et l'administration de PostGIS.
#. **Récupération**: fonctions qui permettent de *récupérer* les propriétés et les mesures d'une géométrie. 
#. **Comparaison**: fonctions qui permettent de *comparer* deux géométries en respectant leur relations spatiales. 
#. **Contruction**: fonctions qui permettent de *construire* de nouvelles géométries à partir d'autre.

La liste des fonctions possibles est très vaste, mais un ensemble communs à l'ensemble des implémentation est défini par la spécification term:`OGC` :term:`SFSQL` et sont implémentées (ainsi que certaines supplémentaires) dans PostGIS.


Quest-ce que PostGIS ?
======================

PostGIS confère au `système de gestion de base de données PostgreSQL <http://www.postgresql.org/>`_ le status de base de données spatiales en ajoutant les trois supports suivants : les types de données spatiales, les indexes et les fonctions. Étant donné qu'il est basé sur PostgreSQL, PostGIS bénéficie automatiquement des capacités orienté "entreprise" ainsi que le respect des standards de cette implémentation.

Mais qu'est-ce que PostgreSQL ?
-------------------------------

PostgreSQL est un puissant système de gestion de données relationel à objets (SGBDRO). Il a été publié sous la licence de style BSD et est donc un logiciel libre. Comme avec beaucoup de logiciels libres, PostgreSQL n'est pas controlé par une société unique mais par une communauté de développeurs et de sociétés qui le développe.

PostgreSQL a été conçu depuis le début en conservant à l'esprit qu'il serait potentiellement nécessaire de l'étendre à l'aide d'extensions particulières -- la possibilité d'ajouter de nouveau types, des nouvelles fonctions et des méthodes d'accès à chaud. Grâce à cela, une extension de PostgreSQL peut être développé par une équipe de développement indépendante, bien que le lien soit très fortement lié au coeur de la base de données PostgreSQL.

Pourquoi choisir PostgreSQL ?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Une question que se posent souvent les gens déja familiarisés avec les bases de données libres est : "Pourquoi PostGIS n'a pas été basé sur MySQL ?"

PostgreSQL a:

  * prouvé sa fiabilité et son respect de l'intégrité des données ( propriétés ACID)
  * un support soigneux des standard SQL (respecte la norme SQL92)
  * un support pour le développement d'extensions et de nouvelles fonctions
  * un modèle de développement communautaire 
  * pas de limite sur la taille des colonne (les tuples peuvent être "TOAST"és) pour supporter des objets géographiques
  * un structure d'index générique (GiST) permettant l'indéxation à l'aide d'abres R
  * facilité ajout de fonctions personalisées

Tout ceci combiné, PostgreSQL permet un cheminement simple du développement nécessaire à l'ajout des types spatiaux. Dans le monde propriétaire, seul Illustra (maintenant Informix Universal Server) permet une extension aussi simple. Ceci n'est pas une coincidence, Illustra est une version propriétaire modifiée du code original de PostgreSQL publié dans les années 1980. 

Puisque le cheminement du développement nécessaire à l'ajout de types à PostgreSQL est direct, il semblait naturel de commencer par là. Lorsque MySQL a publié des types de données spatiales de base dans sa version 4.1, l'équipe de PostGIS a jetté un coup d'oeil dans leur code source et cela a confirmé le choix initial d'utiliser PostgreSQL. Puisque les objets géographiques de MySQL doivent être considérés comme un cas particulier de chaînes de caractères, le code de MySQL a été diffus dans l'intégralité du code de base. Le développement de PostGIS version 0.1 a pris un mois. Réaliser un projet "MyGIS" 0.1 aurait pris beaucoup plus de temps, c'est sans doute pourquoi il n'a jamais vu le jour.

Pourquoi pas des fichiers Shapefile ?
------------------------------------

Les fichiers `shapefile <http://en.wikipedia.org/wiki/Shapefile>`_ (et les autres formats) ont été la manière standard de stocker et d'interragir avec les données spatiales depuis l'origine des SIG. Néanmoins, ces fichiers "plats" ont les inconvénients suivants :

* **Les fichier au formats SIG requièrent un logiciel spécifique pour les lire et les écrire.**  Le langage SQL est une abstraction de l'accès alléatoire au données et à leur analyse. Sans cette abstraction, vous devrez développer l'accès et l'anayse par vos propre moyens.
* **L'accès concurent aux données peut parfois entrainer un stockage de données corrompues.** Alors qu'il est possible d'écrire du code supplémentaire afin de garantir la cohérence des données, une fois ce problème solutionné et celui de la performance associée, vous aurez re-écrit la partie la plus importante d'un système de base de données. Pourquoi ne pas simplement utilisé une base de données standard dans ce cas ?
* **Les questions compliquées nécessitent des logiciels compliqués pour y répondre.** Les question intéressantes et compliquées (jointures spatiales, aggrégations, etc) qui sont exprimables en une ligne de SQL grâce à la base de données, nécessitent une centaines de lignes de code spécifiques pour y répondre dans le cas de fichiers.

La plupart des utilisateurs de PostGIS ont mis en place des systèmes où diverses applications sont succeptibles d'accéder aux données, et donc d'avoir les méthodes d'accès SQL standard, qui simplifient le déploiement et le développement. Certains utilisateurs travaillent avec de grands jeux de données sous forme de fichiers, qui peuvent être segmentés en plusieurs fichiers, mais dans une base de données ces données peuvent être stockées dans une seule grande table.

En résumé, la combinaison du support de l'accès concurent, des requêtes complexes spécifiques et de la performance sur de grand jeux de données différencient les bases de données spatiales des systèmes utilisant des fichiers.

Un bref historique de PostGIS
------------------------------

En mai 2001, la société `Refractions Research <http://www.refractions.net/>`_  publie la permière version de PostGIS. PostGIS 0.1 fournissait les objets, les indexes et des fonctions utiles. Le résultat était une base de données permettant le stockage et l'accès mais pas encore l'analyse.

Comme le nombre de fonctions augmentait, le besoin d'un principe d'organisation devint clair. La spécification "Simple Features for SQL" (:term:`SFSQL`) publiée par l'Open Geospatial Consortium fournit une telle structure avec des indications pour le nommage des fonctions et les pré-requis.

Avec le support dans PostGIS de simples fonctions d'analyses et de jointures spatiales, 
`Mapserver <http://mapserver.org/>`_ devint la première application externe permettant de visualiser les données de la base de données.

Au cours de ces dernières années le nombre de fonctions fournies par PostGIS grandit, mais sa puissance restait limité. La plupart des fonctions interressantes (ex : ST_Intersects(), ST_Buffer(), ST_Union()) étaient difficiles à implémenter. Les écrire en repartant du début promettait des années de travail.

Heureusement un second projet, nommé "Geometry Engine, Open Source" ou `GEOS <http://trac.osgeo.org/geos>`_ vit le jour. Cette librairie fournit l'ensemble des algorithmes nécessaires à l'implémentation de la spécification :term:`SFSQL` . En se liant à GEOS, PostGIS fournit alors le support complet de la :term:`SFSQL` depuis la version 0.8.

Alors que les capacités de PostGIS grandissaient, un autre problème fit surface : La représentation utilisée pour stocker les géométries n'était pas assez efficace. Pour de petits objets comme les points ou de courtes lignes, les métadonnées dans la représentation occupaient plus de 300% supplémentaires. Pour des raisons de performance, il fut nécessaire de faire faire un régime à la représentation. En réduisant l'entête des métadonnées et les dimensions requises, l'espace supplémentaire fut réduit drastiquement. Dans PostGIS 1.0, cette nouvelle représentation plus rapide et plus légère devint la représentation par défaut.

Les mises à jour récentes de PostGIS ont permit d'éttendre la compatibilité avec les standards, d'ajouter les géométries courbes et les signatures de fonctions spécifiées dans la norme ISO :term:`SQL/MM`. Dans un soucis de performance, PostGIS 1.4 augmenta considérablement la rapidité d'exécution des fonctions de test sur les géométries.

Qui utilisent PostGIS ?
-----------------------

Pour une liste complète des cas d'utilisation, consultez la page web : `PostGIS case studies <http://www.postgis.org/documentation/casestudies/>`_.

Institut Geographique National, France
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

L'IGN utilise PostGIS pour stocker des cartes topographiques de grande résolution de la France : la "BDUni". La BDUni a plus de 100 millions d'entités, et est maintenue par une équipe de 100 personnes qui vérifie les observations et ajoute quotidiennement de nouvelles données à la base. L'installation de l'IGN utilise le système transactionel de la base de données pour assurer la consistance durant les phases de mises à jour et utilise un `serveur de rtandby par transfert de journaux <http://docs.postgresql.fr/9.1/warm-standby.html>`_ afin de conserver un état cohérent en cas de défaillance du système.

GlobeXplorer
~~~~~~~~~~~~

GlobeXplorer est un service web fournissant un accès en ligne à une imagerie satellite et photos aériennes de plusieures petabytes. GlobeXplorer utilise PostGIS pour gérer les métadonnées associées avec le catalogue d'images. Les requêtes pour accéder aux images recherchent d'abord dans le catalogue PostGIS pour récupérer la localisation des images demandées, puis récupèrent ces images et les retournent au client. Lors du proeccessus de mise en place de leur système, GlobeXplorer a essayé d'autre système de base de données spatiales mais a conserver PostGIS à cause de la combinaison du prix et de la performance qu'il offre.

Quest-ce qu'une application qui supporte PostGIS ?
-------------------------------------------------

PostGIS est devenu une base de données spatiale communément utilisée, et le nombre d'applications tierces qui supportent le stockage ou la récupération des données n'a céssé d'augmenter. `Les application qui supportent PostGIS <http://trac.osgeo.org/postgis/wiki/UsersWikiToolsSupportPostgis>`_  contiennent à la fois des applications libres et des application propriétaires tournant sur un serveur ou localement depuis votre bureau.

La table suivante propose une liste des logiciels qui tirent profit de PostGIS :

+-------------------------------------------------+----------------------------------------------+
| Libre/Gratuit                                   | Fermé/Propriétaire                           |
+=================================================+==============================================+
|                                                 |                                              |   
| * Chargement/Extraction                         | * Chargement/Extraction                      |   
|                                                 |                                              |     
|   * Shp2Pgsql                                   |   * Safe FME Desktop Translator/Converter    |      
|   * ogr2ogr                                     |                                              |        
|   * Dxf2PostGIS                                 |                                              |          
|                                                 | * Basé sur web                               |         
| * Basé sur le web                               |                                              |             
|                                                 |   * Ionic Red Spider (now ERDAS)             |              
|   * Mapserver                                   |   * Cadcorp GeognoSIS                        |            
|   * GeoServer (Java-based WFS / WMS -server )   |   * Iwan Mapserver                           |     
|   * SharpMap SDK - for ASP.NET 2.0              |   * MapDotNet Server                         |      
|   * MapGuide Open Source (using FDO)            |   * MapGuide Enterprise (using FDO)          |   
|                                                 |   * ESRI ArcGIS Server 9.3+                  |         
| * Logiciels bureautiques                        |                                              |           
|                                                 | * Logiciels bureautiques                     |               
|   * uDig                                        |                                              |           
|   * QGIS                                        |   * Cadcorp SIS                              |      
|   * mezoGIS                                     |   * Microimages TNTmips GIS                  |         
|   * OpenJUMP                                    |   * ESRI ArcGIS 9.3+                         |           
|   * OpenEV                                      |   * Manifold                                 |   
|   * SharpMap SDK for Microsoft.NET 2.0          |   * GeoConcept                               |       
|   * ZigGIS for ArcGIS/ArcObjects.NET            |   * MapInfo (v10)                            |           
|   * GvSIG                                       |   * AutoCAD Map 3D (using FDO)               |   
|   * GRASS                                       |                                              |           
|                                                 |                                              |             
+-------------------------------------------------+----------------------------------------------+

