.. _about_data:

Partie 5 : A propos de nos données
==================================

Les données utilisées dans ces travaux pratiques sont quatre shapefiles de la ville de New York, et une table attributaire des variables socio-démographiques de la ville. Nous les avons charger sous forme de tables PostGIS et nous ajouterons les données  socio-démographiques plus tard.

Cette partie fournit le nombre d'enregistrements et les attributs de chacun de nos ensembles de données. Ces valeurs attributaires et les relations sont essentielles pour nos futures analyses.

Pour visualiser la nature de vos tables depuis pgAdmin, cliquez avec le bouton droit sur une table et sélectionnez **Properties**. Vous trouverez un résumé des propriétés de la table, incluant la liste des attributs d'une table dans l'onglet **Columns**.

nyc_census_blocks
-----------------

Un bloc ressencé est la plus petite entité géographique pour laquelle un ressencement est raporté. Toutes les couches représentant les niveaux suppérieurs (régions, zones de métro, comtés) peuvent être contruites à partir de ces blocs. Nous avons attaché des données démographiques aux blocs.

Nombre d'enregistrements : 36592

.. list-table::
   :widths: 20 80 

   * - **blkid**
     - Un code à 15 chiffres qui permet d'identifier de manière unique chaque bloc **block**. Eg: 360050001009000
   * - **popn_total**
     - Nombre total de personnes dans le bloc
   * - **popn_white**
     - Nombre de personnes se déclarant comme de couleur blanche
   * - **popn_black**
     - Nombre de personnes se déclarant comme de couleur noire
   * - **popn_nativ**
     - Nombre de personnes se déclarant comme natif d'amérique du nord
   * - **popn_asian**
     - Nombre de personnes se déclarant comme asiatique
   * - **popn_other**
     - Nombre de personnes se déclarant comme faisant partie d'une autre catégorie
   * - **hous_total**
     - Nombre de pièces dans le bloc
   * - **hous_own**
     - Nombre de propriétaires occupant le bloc
   * - **hous_rent**
     - Nombre de locataires occupant le bloc
   * - **boroname**
     - Nom du quartier (Manhattan, The Bronx, Brooklyn, Staten Island, Queens)
   * - **the_geom**
     - Polygone représentant les contours d'un bloc

.. figure:: ./screenshots/nyc_census_blocks.png
   
   *Pourcentage de la population qui est de couleur noire* 

.. note:: 

    Pour disposer des données d'un recensement dans votre SIG, vous avez besoin de joindre deux informations: Les données socio-démographiques et les limites géographiques des blocs/quartiers. Il existe plusieurs moyen de se les procurer, dans notre cas elles ont été récupérées sur le site Internet du Census Bureau's `American FactFinder <http://factfinder.census.gov>`_. 
    
nyc_neighborhoods
-----------------

Les quartiers de New York 

Nombre d'enregistrements: 129

.. list-table::
   :widths: 20 80 

   * - **name**
     - Nom du quartier
   * - **boroname**
     - Name de la section dans New York (Manhattan, The Bronx, Brooklyn, Staten Island, Queens)
   * - **the_geom**
     - Limite polygonale du quartier
   
.. figure:: ./screenshots/nyc_neighborhoods.png

    *Les quartiers de New York* 

nyc_streets
-----------

Les rues de New York

Nombre d'enregistrements: 19091

.. list-table::
   :widths: 20 80 

   * - **name**
     - Nom de la rue
   * - **oneway**
     - Est-ce que la rue est à sens unique ? "yes" = yes, "" = no
   * - **type**
     - Type de voie (Cf: primary, secondary, residential, motorway)
   * - **the_geom**
     - Ligne du centre de la rue.
   
.. figure:: ./screenshots/nyc_streets.png

     *Les rues de New York (les rues principales apparaissent en rouge)*

   
nyc_subway_stations
-------------------

Les stations de métro de New York

Nombre d'enregistrements: 491

.. list-table::
   :widths: 20 80

   * - **name**
     - Nom de la station
   * - **borough**
     - Nom de la section dans New York (Manhattan, The Bronx, Brooklyn, Staten Island, Queens)
   * - **routes**
     - Lignes de métro passant par cette station
   * - **transfers**
     - Lignes de métro accessibles depuis cette station
   * - **express**
     - Stations ou le train express s'arrête, "express" = yes, "" = no
   * - **the_geom**
     - Localisation ponctuelle de la station

.. figure:: ./screenshots/nyc_subway_stations.png

    *Localisation ponctuelle des stations de métro de New York*

nyc_census_sociodata
--------------------

Données socio-démographiques de la ville de New York

.. note::

   La donnée ``nyc_census_sociodata`` est une table attributaire. Nous devrons nous connecter aux géométries correspondant à la zone du recenssement avant de conduire toute analyse spatiale .
   
.. list-table::
   :widths: 20 80

   * - **tractid**
     - Un code à 11 chiffre qui identifie chaque secteur de recessement. **tract**. Eg: 36005000100
   * - **transit_total**
     - Nombre de travailleurs dans le secteur
   * - **transit_public**
     - Nombre de travailleurs dans le secteur utilisant les transports en commun
   * - **transit_private**
     - Nombre de travailleurs dans le secteur utilisant un véhicule privé
   * - **transit_other**
     - Nombre de travailleurs dans le secteur utilisant un autre moyen de transport 
   * - **transit_time_mins**
     - Nombre total de minutes passées dans les transports par l'ensemble des travailleurs du secteur (minutes)
   * - **family_count**
     - Nombre de familles dans le secteur
   * - **family_income_median**
     - Revenu médiant par famille du secteur (dollars)
   * - **family_income_aggregate**
     -  Revenu total de toutes les familles du secteur (dollars)
   * - **edu_total**
     - Nombre de personnes ayant un parcours scolaire
   * - **edu_no_highschool_dipl**
     - Nombre de personnes n'ayant pas de diplôme d'éducation secondaire
   * - **edu_highschool_dipl**
     - Nombre de personnes ayant un diplôme d'éducation secondaire
   * - **edu_college_dipl**
     - Nombre de personnes ayant un diplôme de lycée
   * - **edu_graduate_dipl**
     - Nombre de personnes ayant un diplôme de collège 

