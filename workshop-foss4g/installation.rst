.. _installation:

Partie 2 : Installation
=======================

Nous utiliserons l'OpenGeo Suite comme application d'installation, car celle-ci contient PostGIS/PostgreSQL dans un seul outil d'installation pour Windows, Apple OS/X et Linux. La suite contient aussi GeoServer, OpenLayers et d'autres outils de visulations sur le web.

.. note::
  
Si vous souhaitez installer simplement PostgreSQL, cela peut se faire en téléchargeant directement le code source ou les binaires de PostgreSQL sur le site du projet http://postgresql.org/download/. Après avoir installé PostgreSQL, utilisez l'outil "StackBuilder" pour ajouter l'extension PostGIS à votre installation.

.. note:: 

Les indications précises de ce document sont propre à Windows, mais l'installation sous OS/X est largement similaire. Une fois la Suite installée, les instructions relatives au système d'exploitation devraient être identiques.

#. Dans le répertoire :file:`postgisintro\\software\\` vous trouverez l'installeur de l'OpenGeo Suite nommé :  :file:`opengeosuite-2.4.3.exe` (sur OS/X, :file:`opengeosuite-2.4.3.dmg`).  Double cliquez sur cet exécutable pour le lancer.

#. Appréciez le message de courtoisie d'OpenGeo, puis cliquez sur **Next**.

   .. image:: ./screenshots/install_01.png


#. L'OpenGeo Suite est publiée sous licence GPL, ce qui est précisé dans la fenêtre de license.  Cliquez sur **I Agree**.

   .. image:: ./screenshots/install_02.png


#. Le répertoire où l'OpenGeo Suite sera installé est généralement le répertoire ``C:\Program Files\``. Les données seront placées dans le répertoire personnel de votre utilisateur, dans le répertoire :file:`.opengeo`.  Cliquez sur **Next**.

   .. image:: ./screenshots/install_03.png


#. L'installeur créera un certain nombre de raccourcis dans le répertoire OpenGeo du menu démarrer. Cliquez sur **Next**.

   .. image:: ./screenshots/install_04.png


#. Tout les composants de la Suite sont obligatoires à ce niveau. Cliquez sur **Next**.

   .. image:: ./screenshots/install_05.png


#. Prêt à installer ! Cliquez sur **Install**.

   .. image:: ./screenshots/install_06.png


#. Le processus d'installation prendra quelques minutes.

   .. image:: ./screenshots/install_07.png


#. Lorsque l'installation est terminée, lancez le Dashboard pour commencer la partie suivante de ces travaux pratiques ! Cliquez sur **Finish**.

   .. image:: ./screenshots/install_08.png


