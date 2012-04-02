.. _joins_exercises:

Partie 13 : Exercices sur jointures spatiales
=============================================

Voici un petit rappel de certaines des fonctions vues précédemment. Elles seront utiles pour les exercices !

 * :command:`sum(expression)` agrégation retournant la somme d'un ensemble
 * :command:`count(expression)` agrégation retournant le nombre d'éléments d'un ensemble
* :command:`ST_Area(geometry)` retourbe l'aire d'un polygone
* :command:`ST_AsText(geometry)` returns WKT ``text``
* :command:`ST_Contains(geometry A, geometry B)` retourne vrai si la géométrie A contient la géométrie B 
* :command:`ST_Distance(geometry A, geometry B)` retourne la distance minimum entre deux géométries
* :command:`ST_DWithin(geometry A, geometry B, radius)` retourne vrai si la A est distante d'au plus radius de B
* :command:`ST_GeomFromText(text)` returns ``geometry``
* :command:`ST_Intersects(geometry A, geometry B)` returns the true if geometry A intersects geometry B
* :command:`ST_Length(linestring)` retourne la longueur d'une linestring
* :command:`ST_Touches(geometry A, geometry B)` retourne vrai si le contour extérieur de A touche B
* :command:`ST_Within(geometry A, geometry B)` retourne vrai si A est hors de B

Souvenez-vous aussi des tables à votre disposition : 

 * ``nyc_census_blocks`` 
 
   * name, popn_total, boroname, the_geom
 
 * ``nyc_streets``
 
   * name, type, the_geom
   
 * ``nyc_subway_stations``
 
   * name, routes, the_geom
 
 * ``nyc_neighborhoods``
 
   * name, boroname, the_geom

Exercices
---------

 * **"Quelle station de métros se situe dans le quartier 'Little Italy' ? Quelle est l'itinéraire de métro à emprunter ?"**
 
   .. code-block:: sql
 
     SELECT s.name, s.routes 
     FROM nyc_subway_stations AS s
     JOIN nyc_neighborhoods AS n 
     ON ST_Contains(n.the_geom, s.the_geom)  
     WHERE n.name = 'Little Italy';

   :: 
  
       name    | routes 
    -----------+--------
     Spring St | 6
     
 * **"Quels sont les quartiers desservis pas le train numéro 6 ?"** (Astuce: la colonne ``routes`` de la table ``nyc_subway_stations`` dispose des valeurs suivantes: 'B,D,6,V' et 'C,6')
 
   .. code-block:: sql
  
    SELECT DISTINCT n.name, n.boroname 
    FROM nyc_subway_stations AS s
    JOIN nyc_neighborhoods AS n 
    ON ST_Contains(n.the_geom, s.the_geom)  
    WHERE strpos(s.routes,'6') > 0;
    
   ::
  
            name        | boroname  
    --------------------+-----------
     Midtown            | Manhattan
     Hunts Point        | The Bronx
     Gramercy           | Manhattan
     Little Italy       | Manhattan
     Financial District | Manhattan
     South Bronx        | The Bronx
     Yorkville          | Manhattan
     Murray Hill        | Manhattan
     Mott Haven         | The Bronx
     Upper East Side    | Manhattan
     Chinatown          | Manhattan
     East Harlem        | Manhattan
     Greenwich Village  | Manhattan
     Parkchester        | The Bronx
     Soundview          | The Bronx

   .. note::
  
     Nous avons utilisé le mot clef ``DISTINCT`` pour supprimer les répétitions dans notre ensemble de résultats où il y avait plus d'une seule station de métro dans le quartier. 
        
 * **"Après le 11 septembre, le quartier de 'Battery Park' était interdit d'accès pendant plusieurs jours. Combien de personnes ont dû être évacuées ?"**
 
   .. code-block:: sql
 
     SELECT Sum(popn_total)
     FROM nyc_neighborhoods AS n
     JOIN nyc_census_blocks AS c 
     ON ST_Intersects(n.the_geom, c.the_geom)  
     WHERE n.name = 'Battery Park';
   
   :: 

     9928
    
 * **"Quelle est la densité de population (personne / km^2) des quartiers de 'Upper West Side' et de 'Upper East Side' ?"** (Astuce: il y a 1000000 m^2 dans un km^2.)
 
   .. code-block:: sql
   
     SELECT 
       n.name, 
       Sum(c.popn_total) / (ST_Area(n.the_geom) / 1000000.0) AS popn_per_sqkm
     FROM nyc_census_blocks AS c
     JOIN nyc_neighborhoods AS n
     ON ST_Intersects(c.the_geom, n.the_geom)
     WHERE n.name = 'Upper West Side'
     OR n.name = 'Upper East Side'
     GROUP BY n.name, n.the_geom;
     
   ::
   
           name       |  popn_per_sqkm   
     -----------------+------------------
      Upper East Side | 47943.3590089405
      Upper West Side | 39729.5779474286

     
