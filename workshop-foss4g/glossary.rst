.. _glossary:

Annexes B : Glossaire
====================

.. glossary::

    CRS
        Un "système de références spatiales". La combinaison d'un système de coordonnée géographiques et un système de projection.

    GDAL
        `Geospatial Data Abstraction Library <http://gdal.org>`_, prononcé "GéDAL", une bibliothèque open source permettant d'accéder aux données rasters supportant un grand nombre de formats, utilisé largement à la fois dans les applications open source et propriétaires.

    GeoJSON
        "Javascript Object Notation", un format texte qui est très rapide et qui permet de représenter des objet JavaScript. En spatial, la spécification étendue `GeoJSON <http://geojson.org>`_ est courramment utilisée.
    
    SIG
        `Système d'Information Géographique <http://en.wikipedia.org/wiki/Geographic_information_system>`_ capture, stock, analyse, gère, et présente les données qui sont reliées à la zone géographique.
    
    GML
        `Geography Markup Language <http://www.opengeospatial.org/standards/gml>`_.  Le GML est un format standard XML :term:`OGC` pour représenter les données géographiques.

    JSON
        "Javascript Object Notation", un format text qui est très rapide permettant de stocker les objets JavaScript. Au niveau spatial, la spécification étendu `GeoJSON <http://geojson.org>`_ est courramment utilisé.

    JSTL
        "JavaServer Page Template Library", est une bibliothèque pour :term:`JSP` qui encapsule plusieurs fonctionalités de bases géré en JSP (requête de bases de données, itération, conditionnel) dans un syntaxe tièrce.

    JSP
        "JavaServer Pages" est un système de script pour les serveur d'applications Java qui permet de mixer du code XML et du code Java.

    KML
        "Keyhole Markup Language", le format XML utilisé par Google Earth. Google Earth. Il fût à l'origine développé par la société "Keyhole", ce qui expliqe sa présence (maintenant obscure)  dans le nom du format.

    OGC
        Open Geospatial Consortium <http://opengeospatial.org/> (OGC) est une organisation qui développent des spécifications pour les services spatiaux.

    OSGeo
         Open Source Geospatial Foundation <http://osgeo.org> (OSGeo) est une association à but non lucratif dédié à la promotion et au support des logiciels cartographiques open source.

    SFSQL
        La spécification `Simple Features for SQL <http://www.opengeospatial.org/standards/sfs>`_ (SFSQL) de l':term:`OGC` définit les types et les fonctions qui doivent être disponibles dans une base de données spatiales.

    SLD
        Les spécifications `Styled Layer Descriptor <http://www.opengeospatial.org/standards/sld>`_ (SLD) de l':term:`OGC` définissent un format permettant de décrire la manière d'afficher des donnéesvectorielles.

    SRID
        "Spatial reference ID" est un identifiant unique assigné à un système de coordonnées géographique particulier. La table PostGIS **spatial_ref_sys** contient un loarge collection de valeurs de SRID connus.

    SQL
        "Structured query language" est un standard permettant de requêter les bases de données relationnelle. Référence http://en.wikipedia.org/wiki/SQL.

    SQL/MM
        `SQL Multimedia <http://www.fer.hr/_download/repository/SQLMM_Spatial-_The_Standard_to_Manage_Spatial_Data_in_Relational_Database_Systems.pdf>`_; includes several sections on extended types, including a substantial section on spatial types.

    SVG
        "Scalable vector graphics" est une famille de spécifications basé sur le format XML pour décrire des objet graphiques en 2 dimensions, aussi bien statiques que dynamiques (par exemple interactive ou animé). Réference : http://en.wikipedia.org/wiki/Scalable_Vector_Graphics.

    WFS
        Les spécifications `Web Feature Service <http://www.opengeospatial.org/standards/wfs>`_ (WFS) de l':term:`OGC` définit une interface pour lire et écrire des données géographiques à travers internet.

    WMS
        Les spécifications `Web Map Service <http://www.opengeospatial.org/standards/wms>`_ (WMS) de l':term:`OGC` définit une interface pour requêter une carte à travers internet.

    WKB
        "Well-known binary". Fait référence à la représentation binaire desgéométries comme décrit dans les spécifications Simple Features for SQL (:term:`SFSQL`).
        
    WKT
        "Well-known text". Fait référence à la représentation textuelle de géométries, avec des chaînes commençant par "POINT", "LINESTRING", "POLYGON", etc. Il peut aussi faire référence à la représentation textuelle d'un :term:`CRS`, avec une chaîne commençant par "PROJCS", "GEOGCS", etc.  Les représentations au format Well-known text sont des standards de l':term:`OGC`, mais n'ont pas leur propres documents de spécifications. La première description du WKT (pour les géométries et pour les CRS) apparaissent dans les spécifications :term:`SFSQL` 1.0.
        

  
