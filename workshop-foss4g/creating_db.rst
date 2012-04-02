.. _creating_db:

Partie 3 : Créer une base de données spatiales
===============================================

Le Dashboard et PgAdmin
-----------------------

Le "Dashboard" est une application centralisant les accès aux différentes parties de l'openGeo Suite.

Lorsque vous démarrez le dashboard pour la première fois, il vous fournit une indication quand au mot de passe par défaut pour accéder à GeoServer.

.. image:: ./screenshots/dashboard_01.png

.. note::

  La base de données PostGIS a été installée sans la moindre restriction d'accès pour les utilisateurs locaux (les utilisateurs se connectant sur la même machine que celle faisant tourner le serveur de base de données). Cela signifie qu'il acceptera *tout* les mots de passe que vous fournirez. Si vous devez vous connecter depuis un ordinateur distant, le mot de passe pour l'utilisateur ``postgres`` a utiliser est : ``postgres``.

Pour ces travaux pratiques, nous n'utilserons que les parties de la section "PostGIS" du Dashboard.

#. Premièrement, nous devons démarrer les serveur de base de données PostGIS. Cliquez sur le bouton vert **Start** en haut à droite de la fenêtre du Dashboard.

#. La première fois que la Suite se démarre, elle initialise un espace de données et met en place des modèles de bases de données. Ceci peut prendre quelque minutes. Une fois la Suite lancée, vous pouvez cliquer sur l'option **Manage** dans le composant *PostGIS*  pour lancer l'outil pgAdmin.

      .. image:: ./screenshots/dashboard_02.png
  
      .. note:: 
  
         PostgreSQL dispose de nombreux outils d'administration différents.  Le premier est `psql <http://www.postgresql.org/docs/8.1/static/app-psql.html>`_ un outil en ligne de commande permettant de saisir des requêtes SQL. Un autre outil d'administation populaire est l'outils graphique libre et gratuit `pgAdmin <http://www.pgadmin.org/>`_. Toutes les requêtes exécutées depuis pgAdmin peuvent aussi être utilisées depuis la ligne de commande avec psql. 

#. Si c'est la première fois que vous lancez pgAdmin, vous devriez avoir une entrée du type **PostGIS (localhost:54321)** déjà configurée dans pgAdmin. Double cliquez sur cet élément, et entrez le mot de passe de votre choix pour vous connecter au serveur.

    .. image:: ./screenshots/pgadmin_01.png

    .. note::

      Si vous aviez déjà une installation pgAdmin sur votre ordinateur, vous n'aurez pas l'entrée **(localhost:54321)**. Vous devrez donc créer une nouvelle connexion. Allez dans *File > Add Server*, puis enregistrez un nouveau serveur pour **localhost** avec le port **54321** (notez que numéro de port n'est pas standard) afin de vous connecter au serveur PostGIS installé à l'aide de l'OpenGeo Suite.

Créer une base de données
-------------------------

PostgreSQL fournit ce que l'on appèle des modèles de bases de données qui peuvent être utilisés lors de la création d'une nouvelle base. Cette nouvelle base contiendra alors une copie de tout ce qui est présent dans le modèle. Lorsque vous installez PostGIS, une base de données appelée ``template_postgis`` a été crée. Si nous utilisons ``template_postgis`` comme modèle lors de la création de notre nouvelle base, la nouvelle base sera une base de données spatiales.

#. Ouvrez l'arbre des bases de données et regardez quelles sont les bases de données disponibles. La base ``postgres`` est la base de l'utilisateur (par défaut l'utilisateur postgres, donc pas très interressante pour nous). La base ``template_postgis`` est celle que nous utiliserons pour créer des bases de données spatiales.

#. Cliquez avec le clic droit sur l'élément ``Databases`` et selectionnez ``New Database``.

   .. image:: ./screenshots/pgadmin_02.png

   .. note:: Si vous recevez un message d'erreur indiquant que la base de données (``template_postgis``) est utilisée par d'autre utilisateurs, cela signifie que vous l'avez activé par inadvertance. Utilisez alors le clic droit sur l'élément ``PostGIS (localhost:54321)`` puis sélectionnez ``Disconnect``.  Double cliquez sur le même élément pour vous reconnecter et essayez à nouveau.

#. Remplissez le formulaire ``New Database`` puis cliquez sur **OK**.  

   .. list-table::

      * - **Name**
        - ``nyc``
      * - **Owner**
        - ``postgres``
      * - **Encoding**
        - ``UTF8``
      * - **Template**
        - ``template_postgis``

   .. image:: ./screenshots/pgadmin_03.png

#. Selectionnez la nouvelle base de données ``nyc`` et ouvrez là pour consulter son contenu. Vous verrez le schéma ``public``, et sous cela un ensemble de tables de métadonnées spécifiques à PostGIS -- ``geometry_columns`` et ``spatial_ref_sys``.

   .. image:: ./screenshots/pgadmin_04.png

#. Cliquez sur le bouton SQL query comme présenté ci-dessous (ou allez dans *Tools > Query Tool*).

   .. image:: ./screenshots/pgadmin_05.png

#. Saisissez la requête suivante dans le champ prévu à cet effet :

   .. code-block:: sql

      SELECT postgis_full_version();

   .. note::
   
      C'est notre première requête SQL.  ``postgis_full_version()`` est une fonction d'administration qui renvoit le numéro de version et les options de configuration utilisées lors de la compilation. 
      
#. Cliquez sur le bouton **Play** dans la barre d'outils (ou utilisez la touche **F5**) pour  "exécuter la requête." La requête retournera la chaîne de caractères suivante, confirmant que PostGIS est correctement activé dans la base de données.

   .. image:: ./screenshots/pgadmin_06.png
   
Vous venez de créer une base de données PostGIS avec succès !

Liste des fonctions
-------------------

`PostGIS_Full_Version <http://postgis.org/documentation/manual-svn/PostGIS_Full_Version.html>`_: Retourne les informations complètes relatives à la version et aux options de compilation de postgis.
