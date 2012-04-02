.. _tuning:

Partie 21 : Paramétrer PostgreSQL pour le spatial
=================================================

PostgreSQL est une base de données très versatile, capable de tourner dans des environnements ayant des ressources très limités et partageant ces ressources avec un grand nombre d'autres applications. Afin d'assurer qu'il tournera convenablement dans ces environnements, la configuration par défaut est très peu consomatrice de ressource mais terriblement innadapaté pour des bases de données hautes-performances en production. Ajoutez à cela le fait que les base de données spatiales ont différent type d'utilisation, et que les données sont généralement plus grandes que les autres types de données, vous en arriverez à la conclusion que les parètres par défaut ne sont pas approprié pour notre utilisasion.

Tout ces paramètres de configuration peuvent être édités dans le fichier de configuration de la base de données : :file:`C:\\Documents and Settings\\%USER\\.opengeo\\pgdata\\%USER`.  Le contenu du fichier est du texte et il peut donc être ouvert avec l'outils d'édition de fichiers de votre choix (Notepad par exemple). Les modifications apportées à ce fichier ne seront effectives que lors du redémarrage du serveur.

.. image:: ./tuning/conf01.png

Une façon simple d'éditer ce fichier de configuration est d'utiliser l'outils nommé : "Backend Configuration Editor".  Depuis pgAdmin, allez dans *File > Open postgresql.conf...*. Il vous sera demandé le chemin du fichier, naviguez dans votre arborescence jusqu'au fichier :file:`C:\\Documents and Settings\\%USER\\.opengeo\\pgdata\\%USER`.

.. image:: ./tuning/conf02.png

.. image:: ./tuning/conf03.png

Cette partie décrit certains des paramètres de configuration qui doivent être modifiés pour la mise ne place d'une base de données spatiale en production. Pour chaque partie, trouvez le bon paramètres dans la liste et double cliquez dessus pour l'éditer. Changez le champs *Value* par la valeur que nous recommendons, assurez-vous que le champs est bien activé pui cliquez sur **OK**.

.. note:: Ces valeurs sont seulement celles que nous recommendons, chaque environnement differera et tester les différents paramétrages est toujours nécessaire pour s'assurer d'utiliser la configuration optimale. Mais dans cette partie nous vous fournissons un bon point de départ.

shared_buffers
--------------

Alloue la quantité de mémoire que le serveur de bases de données utilise pour ses segments de mémoires partagées. Cela est partagé par tout les processus serveur, comme sont nom l'indique. La valeur par défaut est affligeante et inadaptée pour une base de données en production.

  *Valeur par défaut* : typiquement 32MB

  *Valeur recommandée* : 75% de la mémoire de la base de données (500MB)

.. image:: ./tuning/conf04.png

work_mem
--------

Définit la quantité de mémoire que les opération interne d'ordonnancement et les tables de hachages peuvent consommer avec le serveur passe à des fichiers sur le disque. Cette valeur définit la mémoire disponible pour chaque opération complexe, les requêtes complexes peuvent avoir plusieurs ordres ou opération de hachage tournant en parallèle, et chaque client s connecté peut exécuter une requête. 

Vous devez donc considérer combient de connexions et quel complexité est attendu dans les requêtes avant d'augmenter cette valeur. Le bénéfice acquis par l'augmentation de cette valeur est que la plupart des opération de classification,dont les clause ORDER BY et DISTINCT, les jointures, les agrégation basé sur les hachages et l'exécution de requête imbriquées, pourront être réalisé sans avoir à passer par un stockage sur disque.

  *Valeur par défaut* : 1MB

  *Valeur recommandée* : 16MB

.. image:: ./tuning/conf05.png

maintenance_work_mem
--------------------

Définit la quantité de mémoire utilisé pour les opération de maintenances, dont le néttoyage (VACUUM), les indexes et la création de clef étrangères. Comme ces opération sont courremment utilisées, la valeur par défaut devrait être acceptable. Ce paramètre peut être augmenté dynamiquement à l'exécution depuis une connexion au serveur avant l'exécution d'un grand nombre d'appels à :command:`CREATE INDEX` ou :command:`VACUUM` comme le montre la commande suivante.

  .. code-block:: sql

    SET maintenance_work_mem TO '128MB';
    VACUUM ANALYZE;
    SET maintenance_work_mem TO '16MB';

  *Valeur par défaut* : 16MB

  *Valeur recommendée* : 128MB

.. image:: ./tuning/conf06.png

wal_buffers
-----------

Définit la quantité de mémoire utilisé pour l'écriture des données dans le journal respectant la règle du défaire (WAL). Elle indique que les informations pour annuler les effets d'une opération sur un objet doivent être écrites dans le journal en mémoire stable avant que l'objet modifié ne migre sur le disque. Cette règle permet d'assurer l'intégrité des données lors d'une reprise après défaillance. En effet,il suffiré de lire le journal pour retrouver l'état de la base lors de sont arrêt brutal. 

La taille de ce tampon nécessite simplement d'être suffisament grand pour stoquer les données WAL pour une seule transaction. Alors que la valeur par défaut est généralement siffisante, les données spatiales tendent à être plus large. Il est donc recommendé d'augmenter la taille spécifiée dans ce paramètre.

  *Valeur par défaut* : 64kB

  *Valeur recommendée* : 1MB

.. image:: ./tuning/conf07.png

checkpoint_segments
-------------------

Cette valeur définit le nombre maximum de segements des journaux (typiquement 16MB) qui doit être remplit entre chaque point de reprises WAL. Un point de reprise WAL est une partie d'une séquence de transactions pour lequel on garanti que les fichiers de données ont été mis à jour avec toutes les requêtes précédent ce point. À ce moment-là toutes les pages sont punaisées sur le disque et les point de reprises sont écrit dans le fichier de journal. Cela permet au precessus de reprise après défaillance de trouver les dernierspoints de reprises et applique toute les lignes suivantes pour récupérer l'état des données avant la défaillance.

Étant donnée que les point de reprises nécessitent un punaisage de toutes le pages ayant été modifiée sur le disque, cela va créer une charge d'entrées/sorties significative. Le même arguement que précédemment s'applique ici, les données spatiales sont assez grandes pour contrebalancer l'optimisation de données non spatiales. Augmenter cette valeur limitera le nombre de points de reprise, mais impliquera un plus redémarrage en cas de défaillance.

  *Valeur par défaut* : 3

  *Valauer recommendée* : 6

.. image:: ./tuning/conf08.png

random_page_cost
----------------

Cette valeur sans unité représente le coût d'accès alléatoire au page du disque. Cete valeur est relative au autres paramètres de coût notemment l'accès séquentiel au pages, et le coût des opération processeur. Bien qu'il n'y ai pas de valeur magique ici, la valeur par défaut est généralement trop faible. Cette valeur peut être affectée dynamiquement par session en utilisant la commande ``SET random_page_cost TO 2.0``.

  *Valeur par défaut* : 4.0

  *Valeur recommandée* : 2.0

.. image:: ./tuning/conf09.png
   
seq_page_cost
-------------

C'est une paramètre qui controle le coût des accès séquentiel au pages. Il n'est généralement pas nécessaire de modifier cette valeur maus la différence entre cette valeur et la valeurs ``random_page_cost`` affecte drastiquement le choix fait par le plannificateur de requêtes. Cette valeur peut aussi être affectée depuis une session.

  *Valeur par défaut* : 1.0

  *Valeur recommandée* : 1.0

.. image:: ./tuning/conf10.png

Recharger la configuration
--------------------------

Après avoir réalisé ces changements mentioné dans cette partie sauvez-les puis rechargez la configuration.

 * Ceci se fait en cliquant avec le bouton droit sur le nom du serveur (``PostgreSQL 8.4 on localhost:54321``) depuis pgAdmin, selectionnez *Disconnect*. 
 * Cliquez sur le bouton *Shutdown* depuis le Dashboard OpenGeo, puis cliquez sur *Start*. 
 * Pour finir reconnectez-vous au serveur depuis pgAdmin (cliquez avec le bouton droit sur le serveur puis sélectionnez *Connect*).
 
 
 
