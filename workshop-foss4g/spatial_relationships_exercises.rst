.. _spatial_relationships_exercises:

Partie 11 : Exercises sur les relations spatiales
===========================================

Voici un rappel des fonctions que nous avons vu dans les parties précédentes. Elles seront utiles pour les exercices !

 * :command:`sum(expression)` agrégation retournant la somme d'un ensemble
 * :command:`count(expression)` agrégation retournant le nombre d'éléments d'un ensemble
* :command:`ST_Contains(geometry A, geometry B)` retourne vrai si la géométrie A contient la géométrie B 
* :command:`ST_Crosses(geometry A, geometry B)` retourne vrai si la géométrie A croise la géométrie B
* :command:`ST_Disjoint(geometry A , geometry B)` retourne vrai si les géométrie ne s'intersectent pas
* :command:`ST_Distance(geometry A, geometry B)` retourne la distance minimum entre deux géométries
* :command:`ST_DWithin(geometry A, geometry B, radius)` retourne vrai si la A est distante d'au plus radius de B
* :command:`ST_Equals(geometry A, geometry B)` retourne vrai si A est la même géométrie que B
* :command:`ST_Intersects(geometry A, geometry B)` retourne vrai si A intersecte B
* :command:`ST_Overlaps(geometry A, geometry B)` retourne vrai si A et B on un espace en commun, mais ne sont pas complétement inclus l'un dans l'autre.
* :command:`ST_Touches(geometry A, geometry B)` retourne vrai si le contour extérieur de A touche B
* :command:`ST_Within(geometry A, geometry B)` retourne vrai si A est hors de B

Souvenez-vous les tables à votre disposition :

 * ``nyc_census_blocks`` 
 
   * name, popn_total, boroname, the_geom
 
 * ``nyc_streets``
 
   * name, type, the_geom
   
 * ``nyc_subway_stations``
 
   * name, the_geom
 
 * ``nyc_neighborhoods``
 
   * name, boroname, the_geom

Exercices
---------

 * **"Quel est la valeur géométrique de la rue nommée  'Atlantic Commons' ?"**
 
   .. code-block:: sql

     SELECT the_geom
       FROM nyc_streets
       WHERE name = 'Atlantic Commons';

   ::
   
     01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141
     
 * **"Quel sont les quartiers et villes qui sont dans Atlantic Commons ?"**
     
   .. code-block:: sql

     SELECT name, boroname 
     FROM nyc_neighborhoods 
     WHERE ST_Intersects(
       the_geom,
       '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141'
     );

   ::
     
          name    | boroname 
      ------------+----------
       Fort Green | Brooklyn
     

 * **"Quelles rues touchent Atlantic Commons ?"** 
 
   .. code-block:: sql

     SELECT name 
     FROM nyc_streets 
     WHERE ST_Touches(
       the_geom, 
       '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141'
     );
    
   ::
  
          name      
     ---------------
      S Oxford St
      Cumberland St

   .. image:: ./spatial_relationships/atlantic_commons.jpg
  

 * **"Approximativement combien de personnes vivent dans (ou dans une zone de 50 metres autour d') Atlantic Commons ?"**
 
   .. code-block:: sql

     SELECT Sum(popn_total)
       FROM nyc_census_blocks
       WHERE ST_DWithin(
        the_geom,
        '01050000202669000001000000010200000002000000093235673BE82141F319CD89A22E514170E30E0ADFE82141CB2D3EFFA52E5141',
        50
        );
        
   :: 
   
     1186 
   
