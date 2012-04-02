==============================================================================================================
Client en GeoExt
==============================================================================================================

`GeoExt <http://www.geoext.org/>`_ est une librairie JavaScript pour le développement d'applications internet avancées. GeoExt rassemble les capacités d'`OpenLayers <http://www.openlayers.org>`_ avec l'interface utilsiateur savvy de `Ext JS <http://www.sencha.com>`_ pour aider au développement d'applications similaires à des applications bureautiques.

Commençons avec un exemple simple d'utilisation de GeoExt et ajoutons-y les fonctionalités de routage :

.. literalinclude:: ../../web/routing-0.html
	:language: html

Dans l'entête nous chargeons l'ensemble des fichiers javascript et css requis pour l'application. Nous définissons aussi une fonction qui sera exécutée à la fin du chargement de la page (Ext.onReady).

Cette fonction cée une instance de`GeoExt.MapPanel
<http://www.geoext.org/lib/GeoExt/widgets/MapPanel.html>`_ avec une couche de fond OpenStreetMap centrée sur Denver. Dans ce code, aucun objet OpenLayers.Map est explicitemet créé; le GeoExt.MapPanel le fait de façon camouflé : il récupère les options de la carte, le centre, le seuil de zoom et crée une instance de manière appropriée.

Pour permettre à nos utilisateurs de trouver leur direction, nous devons fournir :
 * un moyen de sélectionner un algorithme de routage  (Dijkstra, A* ou Shooting*),
 * un moyen de sélectionner la position de départ et d'arrivée.

.. note:: Ce chapitre présente uniquement des extraits de codes, le code source complet de la page peut être récupérée depuis ``pgrouting-workshop/web/routing-final.html`` qui devrait être sur votre bureau. Le fichier complet se trouve à la fin de ce chapitre.

-------------------------------------------------------------------------------------------------------------
Outil de sélection de la méthode de routage
-------------------------------------------------------------------------------------------------------------

Pour sélectionner une méthode de routage, nous utiliserons un `Ext.form.ComboBox
<http://www.sencha.com/deploy/dev/docs/?class=Ext.form.ComboBox>`_: il se comporte simplement comme un select html mais est plus simple à controler.

Comme pour le GeoExt.MapPanel, nous avons besoin d'un éléments html pour placer notre control, créons un div dans le body (ayant 'method' comme identifiant) :

 .. code-block:: html

   <body>
     <div id="gxmap"></div>
     <div id="method"></div>
   </body>

Créons ensuite le combo :
 .. code-block:: js

   var method = new Ext.form.ComboBox({
       renderTo: "method",
       triggerAction: "all",
       editable: false,
       forceSelection: true,
       store: [
           ["SPD", "Shortest Path Dijkstra"],
           ["SPA", "Shortest Path A*"],
           ["SPS", "Shortest Path Shooting*"]
       ]
   });

Dans l'option ``store``, nous avons défini toutes les valeurs pour les méthodes de routages; les formats sont dans un tableau d'options ou une option est de la forme ``[clef,valeur]``. La ``clef`` sera envoyée au serveur (the script php dans note cas) et la ``valeur`` affichée dans la liste déroulante.

L'option ``renderTo`` défini où la liste déroulante doit être affichée , nous utilisons ici notre élément div.

Et pour finir, une valeur par défaut est définie :
 .. code-block:: js

    method.setValue("SPD");

Cette partie utilise le composant ExtJS : ni code OpenLayers ni GeoExt.

-------------------------------------------------------------------------------------------------------------
Sélectionner un point de départ et d'arrivée
-------------------------------------------------------------------------------------------------------------

Nous souhaitons permettre à nos utilisateurs de déssiner et déplacer un point de départ et d'arrivée. C'est plus ou moins le comportement des applications comme google map et les autres : l'utilsateur choisi un point de départ à l'aide d'un moteur de recherche ou en cliquant sur la carte. L'applicationinterroge ensuite le serveur afin d'afficher le chemin sur la carte. L'utilisateur peut ensuite modifier son opint de départ et d'arrivée afin que le chemin soit mis à jour automatiquement.

Dans ces travaux pratiques, nous implémenterons seulement la saisie de point sur la carte (dessiner des points et les déplacer) mais il est parfaitement possible d'implémenter un moteur de recherche en utilisant un service web dédié tel que `GeoNames <http://www.geonames.org/>`_ ou tout autre service de `geocodage
<http://fr.wikipedia.org/wiki/Geocodage>`_.

Pour faire ceci nous aurons besoin d'un outil permettant de dessiner des points (nous utiliserons le control
`OpenLayers.Control.DrawFeatures
<http://openlayers.org/dev/examples/draw-feature.html>`_) et un outil pour déplacer les points (`OpenLayers.Control.DragFeatures
<http://openlayers.org/dev/examples/drag-feature.html>`_ sera parfait pour ce travail). Comme leur noms le suppose ces controls sont disponibles dans OpenLayers.

Ces deux controls auront besoin d'un emplacement pour afficher et manipuler les points; nous aurons besoin d'une couche `OpenLayers.Layer.Vector
<http://dev.openlayers.org/releases/OpenLayers-2.10/doc/apidocs/files/OpenLayers/Layer/Vector-js.html>`_. Dans OpenLayers, une couche vecteur est un endroit où des élements (une geométrie et des attributs) peuvent être afffichée (contrairement à la couche OSM qui est une couche raster).

Comme les couches vecteurs sont légères, nous en utiliserons une seconde pour afficher la route retourner par le service web. L'initialisation de la couche se fait de la manière suivante :

 .. code-block:: js

    // Création de la couche où le chemin sera affiché
    var route_layer = new OpenLayers.Layer.Vector("route", {
        styleMap: new OpenLayers.StyleMap(new OpenLayers.Style({
            strokeColor: "#ff9933",
            strokeWidth: 3
        }))
    });

``"route"`` est  le nom de la couche, n'importe qu'elle chaîne de caractères peut être utilisée.
``styleMap`` fournis à la couche un mimum de style avec une couleur de contour particulière et un largeur (en pixel).

La seconde couche est simplement initialiser comme suit :

 .. code-block:: js

    // Création de la couche ou seront affiché les points de départ et d'arrivée
    var points_layer = new OpenLayers.Layer.Vector("points");

Les deux couches sont ajoutée à l'objet OpenLayers.Map avec :

 .. code-block:: js

    // Ajouter la cocuhe à la carte
    map.addLayers([points_layer, route_layer]);

Regardons le control pour afficher les points : puisque ce composant à un comportement particulier il est plus simple de créer une nouvelle classe basée sur le control standard
OpenLayers.Control.DrawFeatures. Ce control (nommé DrawPoints) est enregistré dans un fichier javascript à part (``web/DrawPoints.js``):

.. literalinclude:: ../../web/DrawPoints.js
	:language: js

Dans la fonction ``initialize`` (qui est le constructeur de la classe) nous spécifions que ce control peut seulement dessiner des point (la variable handler est OpenLayers.Handler.Point).

Le comportement spécifique est implémenté dans la fonction ``drawFeature`` : puisque nous avons seulement  besoin des points de départ et d'arrivée, le control se desactive automatiquemnt lorsque deux points ont été saisie. La désactivation se fait via ``this.deactivate()``.

Notre control est ensuite créé avec :

 .. code-block:: js

    // Création du control pour dessiner les points (voir le fichier DrawPoints.js)
    var draw_points = new DrawPoints(points_layer);

``points_layer`` est la couche vecteur créée précédemment.

Et maintenant pour le control DragFeature :

 .. code-block:: js

    // Création du control pour déplacer les points 
    var drag_points = new OpenLayers.Control.DragFeature(points_layer, {
        autoActivate: true
    });

Encore une fois, la couche ``points_layer``est de type vecteur, ``autoActivate: true`` dit à OpenLayers que nous voulons que ce control soit automatiquement activé.

 .. code-block:: js

    // Ajouter le control à la carte
    map.addControls([draw_points, drag_points]);

Ajouter le control à la carte.

-------------------------------------------------------------------------------------------------------------
Envoyer et recevoire des données du service web
-------------------------------------------------------------------------------------------------------------

Le cheminement de base pour obtenir un chemin depuis le service web est le suivant :

#. transformer nos points en coordonnées de EPSG:900913 en EPSG:4326
#. appeler le service web avec les arguments correctes (nom de la méthode et deux points)
#. lire le résultat retourné par le service web : transformer le GeoJSON en OpenLayers.Feature.Vector
#. transformer toutes les coordonnées de EPSG:4326 à EPSG:900913
#. ajouter le résultat à la couche vecteur

La première étape est quelque chose de nouveau : notre carte utilise le système de projection EPSG:900913 
(parce que nous utilisons une couche OSM) mais le service web attends des coordonnées en EPSG:4326 : nous devons re-projeter les données avant de les envoyer. Ce n'est pas bien difficile : nous aurons simplement besoin de la ` librairie javascript Proj4js <http://trac.osgeo.org/proj4js/>`_.

(La deuxième étape *appeler le service web* est étudié au chapitre suivant.)

Le service web de routage dans pgrouting.php renvoit un objet  FeatureCollection au format `GeoJSON <http://geojson.org/>`_. Un objet FeatureCollection est simplement un tableau d'élément : un éléments pour chaque tronçon de route. Ceci est très pratique carOpenLayers et GeoExt ont tout ce dont nous avons besoin pour gérer ce format. Pour nous rendre la tâche encore plus simple, nous utiliserons le GeoExt.data.FeatureStore suivant :

 .. code-block:: js

    var store = new GeoExt.data.FeatureStore({
        layer: route_layer,
        fields: [
            {name: "length"}
        ],
        proxy: new GeoExt.data.ProtocolProxy({
            protocol: new OpenLayers.Protocol.HTTP({
                url: './php/pgrouting.php',
                format: new OpenLayers.Format.GeoJSON({
                    internalProjection: epsg_900913,
                    externalProjection: epsg_4326
                })
            })
        })
    });

Un Store est simplement un conteneur qui stocke des informations : nous pouvons y ajouter des éléments et les récupérer.

Expliquons les options :

``layer``: le paramètre est une couche vecteur : en spécifiant une couche, le
FeatureStore affichera automatiquement les données qu'elle contient. C'est exactement ce dont nous avons besoin pour la dernière étape (*ajouter le résultat à la couche vecteur*) dans la liste ci-dessus.

``fields``: liste tout les attributs renvoyés avec la géométrie : pgrouting.php renvoit la longueurdu segment donc nous le spécifions ici. Notez que cette information n'est pas utilisée dans ces travaux pratiques.

``proxy``: le paramètre proxy specifie où nous devons récupérer les données : dans notre cas depuis le serveur HTTP. Le type de proxy est GeoExt.data.ProtocolProxy : cette classe connecte le monde ExtJS (le Store) et le monde OpenLayers (l'objet protocol).

``protocol``: ce composant OpenLayers est capable d'exécuter des requêtes à un``url`` (notre script php) et de lire la réponse (option ``format``). En ajoutant les options ``internalProjection`` et ``externalProjection``, les coordonnées sont reprojetées par l'objet format.

Nous avons maintenant tout ce qu'il nous faut pour gérer les données renvoyées par le service web : le prochain chapitre expliquera comment et quand l'appeler.

-------------------------------------------------------------------------------------------------------------
Déclancher l'appel au service web
-------------------------------------------------------------------------------------------------------------

Nous devons appeler le service web lorsque :
 * les deux points sont dessinés
 * un point à été déplacé
 * la méthode à utiliser a changé 

Notre couche vecteur génère une événement (appelé ``featureadded``) lorsqu'un nouvel élément est ajouté, nous pouvons utiliser cet événement pour appeler la fonction pgrouting (cette fonction sera présenté dans peu de temps) :

 .. code-block:: js

    draw_layer.events.on({
        featureadded: function() {
            pgrouting(store, draw_layer, method.getValue());
        }
    });

.. note:: Avant de continuer quelque mots sur les événements : un événement dans OpenLayers (la même chose s'applique pour ExtJS et les autres frameworks), est un système qui permet à une fonction d'être appelée lorsque *quelquechose* se passe. Par exemple lorsqu'une couche est ajoutée à la carte ou quand la souris se trouve au dessus d'un objet de la carte. Plusieurs fonctions peuvent être liées à un même événement.

Aucune événemenet n'est généré lorsqu'un point est déplacé, heureusement nous pouvons définir une fonction à notre control DragFeature à appeler lorsqu'un point est déplacé :

 .. code-block:: js

    drag_points.onComplete = function() {
        pgrouting(store, draw_layer, method.getValue());
    };

Pour la liste déroulante *method*, nous pouvons ajouter une option `select` au contructeur  (c'est l'événement déclencher lorsqu'un utilisateur change sa sélection) :

 .. code-block:: js

    var method = new Ext.form.ComboBox({
        renderTo: "method",
        triggerAction: "all",
        editable: false,
        forceSelection: true,
        store: [
            ["SPD", "Shortest Path Dijkstra"],
            ["SPA", "Shortest Path A*"],
            ["SPS", "Shortest Path Shooting*"]
        ],
        listeners: {
            select: function() {
                pgrouting(store, draw_layer, method.getValue());
            }
    });

Il est maintenant temps de présenter la fonction pgrouting :

 .. code-block:: js

   // global projection objects (uses the proj4js lib)
   var epsg_4326 = new OpenLayers.Projection("EPSG:4326"), 
       epsg_900913 = new OpenLayers.Projection("EPSG:900913");

   function pgrouting(store, layer, method) {
         if (layer.features.length == 2) {
             // erase the previous route
             store.removeAll();

             // transform the two geometries from EPSG:900913 to EPSG:4326
             var startpoint = layer.features[0].geometry.clone();
             startpoint.transform(epsg_900913, epsg_4326);
             var finalpoint = layer.features[1].geometry.clone();
             finalpoint.transform(epsg_900913, epsg_4326);

             // load to route
             store.load({
                 params: {
                     startpoint: startpoint.x + " " + startpoint.y,
                     finalpoint: finalpoint.x + " " + finalpoint.y,
                     method: method
                 }
             });
        }
    }

La fonction pgrouting appèle me service web à travers l'argument store.

Au début, la fonction vérifie si deux points sont présent dans les paramètres. Ensuite, `select` est appelée pour éffacer le résultat précédent de la couche (souvenez-vous que le Store et la couche vecteur sont lié). Les deux points sont projetés en utilisant une instance de OpenLayers.Projection.

Pour finir, ``store.load()`` est appelée avec un argument ``params`` (ils sont passés via un appèle HTTP utilisant la méthode GET).

-------------------------------------------------------------------------------------------------------------
Que faire maintenant ?
-------------------------------------------------------------------------------------------------------------

Possibles améliorations :
 * Utiliser un service de géocodage pour récupérer le point de départ / d'arrivée
 * Support de plusieurs points 
 * De jolies icônes pour le point de départ et celui d'arrivée
 * Direction du parcour (carte de voyage) : nous avons déjà la distance

-------------------------------------------------------------------------------------------------------------
Code source complet
-------------------------------------------------------------------------------------------------------------

.. literalinclude:: ../../web/routing-final.html
	:language: html

