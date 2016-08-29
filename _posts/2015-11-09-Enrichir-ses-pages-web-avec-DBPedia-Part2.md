---
layout: post
title: Enrichir ses pages web avec DBPedia Part2
categories:
- blog
---

Suite de l'article pour manipuler des données extraites de DBpedia.

----------

# C'est parti pour la suite de l'aventure !

----------
![Bon on continue de manière un peu plus propre ?]({{ site.url }}/assets/img/owl_02.jpg)

## Inclure l'URI dans notre code HTML

Pour ce nouvel article, nous allons apprendre comment récupérer une URI depuis le fichier HTML. On part du principe ici que la ressource sur laquelle nous requêtons est une ville (Paris, Berlin, etc.).

Nous allons donc inclure trois buttons, un bouton pour chaque ville (ici Rennes, Paris et Dijon), lorsque l'utilisateur cliqueras dessus, il verra apparaitre les informations sur la ville concernée.

Voici les URI des trois villes :

```
http://fr.dbpedia.org/resource/Rennes
```

```
http://fr.dbpedia.org/resource/Paris
```

```
http://fr.dbpedia.org/resource/Dijon
```


### La requête SPARQL
La requête SPARQL ne va pas réellement différer de la précédente, seulement cette fois-ci nous veillerons à remplacer la ressource par une variable dans notre script.

Pour rappel, nous avions utilisé la requête suivante (avec des résultats uniquement en français) :
{% highlight xml %}
PREFIX dbpedia-owl: <http://dbpedia.org/ontology/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
SELECT distinct ?label ?abstract ?thumbnail ?thumbnailCaption
WHERE {
<http://fr.dbpedia.org/resource/Rennes> rdfs:label ?label.
<http://fr.dbpedia.org/resource/Rennes> dbpedia-owl:abstract ?abstract.

OPTIONAL {
    <http://fr.dbpedia.org/resource/Rennes> dbpedia-owl:thumbnail ?thumbnail.
    <http://fr.dbpedia.org/resource/Rennes> dbpedia-owl:thumbnailCaption ?thumbnailCaption.
  }

FILTER( lang(?label) = "fr" && lang(?abstract) = "fr")
}
{% endhighlight %}


### La page HTML
Comme je vous l'ai dit, nous allons quelque peu changer le code du fichier index html pour obtenir ceci :
{% highlight html %}
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" lang="fr" xml:lang="fr" >
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
  <title>Récupération des données depuis DBPedia</title>
  <link rel="stylesheet" href="css/style.css">
  <!--[if lt IE 9]>
  	<script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
  <![endif]-->
</head>
<body>
	<div class="container">
		<h1 class="text-center">Manipuler les données de DBPedia</h1>
		<div class="row">
			<div class="col-md-6">
				<button id="target" class="btn btn-info" resource="http://fr.dbpedia.org/resource/Rennes" onclick="getInformation(this)">
					Rennes
				</button>
				<button id="target" class="btn btn-info" resource="http://fr.dbpedia.org/resource/Paris" onclick="getInformation(this)">
					Paris
				</button>
				<button id="target" class="btn btn-info" resource="http://fr.dbpedia.org/resource/Dijon" onclick="getInformation(this)">
					Dijon
				</button>
			</div>
			<div id="infos" class="col-md-6">
			</div>
		</div>
	</div>

	<!-- Latest compiled and minified CSS -->
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
	<script type="text/javascript" src="js/script.js"></script>
</body>
</html>
{% endhighlight %}

Etudions de plus près l'élément "button" :
{% highlight html %}
<button id="target" class="btn btn-info" resource="http://fr.dbpedia.org/resource/Rennes" onclick="getInformation(this)">
  Rennes
</button>
{% endhighlight %}

Chaque bouton renseigne une ressource qui correspond à la ressource d'une ville. C'est grâce à cette ressource que nous allons pouvoir faire varier notre requête en fonction de la ville cible.

De plus, à chaque fois qu'on cliqueras sur le bouton, la fonction "getInformation" sera appelée, c'est elle qui va se charger de renvoyer les informations depuis DBpedia.

### Le script
Voici le script qui se charge de récupérer les données depuis DBpedia.

{% highlight javascript %}
function getInformation(button){
  //On recupère l'URI vers notre ressource
  var uri = button.getAttribute("resource");
  //La requête SPARQL à proprement parler
  var querySPARQL=""+
    'PREFIX dbpedia-owl: <http://dbpedia.org/ontology/> '+
    'PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> '+
    'PREFIX foaf: <http://xmlns.com/foaf/0.1/> '+
    'SELECT ?label ?abstract ?thumbnail ?thumbnailCaption '+
    'WHERE { '+
    '    <'+uri+'> rdfs:label ?label . '+
    '    <'+uri+'> dbpedia-owl:abstract ?abstract .'+
    '    OPTIONAL { '+
    '      <'+uri+'> dbpedia-owl:thumbnail ?thumbnail . '+
    '      <'+uri+'> dbpedia-owl:thumbnailCaption ?thumbnailCaption . '+
    '    } '+
    'FILTER( lang(?label) = "fr" && lang(?abstract) = "fr")'+
    '  }';

  // On prépare l'URL racine (aussi appelé ENDPOINT) pour interroger DBPedia (ici en français)
  var baseURL="http://fr.dbpedia.org/sparql";
  // On construit donc notre requête à partir de cette baseURL
  var queryURL = baseURL + "?" + "query="+ encodeURIComponent(querySPARQL) + "&format=json";

  //On crée notre requête AJAX
  var req = new XMLHttpRequest();
  req.open("GET", queryURL, true);
  req.onreadystatechange = myCode;   // the handler
  req.send(null);

  function myCode()
  {
     if (req.readyState == 4)
     {
        initialisation();

        var doc = JSON.parse(req.responseText);
        var label = doc.results.bindings[0].label.value;
        var abstract =  doc.results.bindings[0].abstract.value;
        var thumbnail =  doc.results.bindings[0].thumbnail.value;
        var thumbnailCaption =  doc.results.bindings[0].thumbnailCaption.value;

        //On crée le titre
        newTitle = document.createElement("h2");
        newTitle.innerHTML = label;

        //On crée le résumé
        newAbstract = document.createElement("div");
        newAbstract.innerHTML = abstract;

        //On crée le conteneur de l'image
        newThumbnail = document.createElement("div");
        newThumbnail.id = "thumbnail";
        var img = new Image();
        img.src = thumbnail;
        img.alt = label;

        //On crée la légende de l'image
        newThumbnailCaption = document.createElement("p");
        newThumbnailCaption.id = "thumbnailCaption";
        newThumbnailCaption.innerHTML = thumbnailCaption;

        document.getElementById("infos").appendChild(newTitle);
        document.getElementById("infos").appendChild(newAbstract);
        document.getElementById("infos").appendChild(newThumbnail);
        document.getElementById("thumbnail").appendChild(img);
        document.getElementById("thumbnail").appendChild(newThumbnailCaption);
     }
  }

  function initialisation() {
    // Removing all children from an element
    var element = document.getElementById("infos");
    while (element.firstChild) {
      element.removeChild(element.firstChild);
    }
  }
}
{% endhighlight %}

Pour récupérer l'URI de notre ressource, il suffit d'utiliser le code ci-dessus.
{% highlight javascript %}
  var uri = button.getAttribute("resource");
{% endhighlight %}

La requête SPARQL ne change pas à ceci près que nous avons désormais placé la variable 'uri' en lieu en place de l'URI vers la ressource de Rennes.

Une fois que nous obtenons la réponse de DBpedia, nous utilisons la fonction "initialisation()" pour nous assurer que la "div" ayant l'identifiant "infos" de notre document soit vide (cela engendrerait des conflits sinon).

Puis nous récupérons toutes les données dont nous avons besoin en les plaçant dans des variables pour plus de lisibilité :
{% highlight javascript %}
var doc = JSON.parse(req.responseText);
var label = doc.results.bindings[0].label.value;
var abstract =  doc.results.bindings[0].abstract.value;
var thumbnail =  doc.results.bindings[0].thumbnail.value;
var thumbnailCaption =  doc.results.bindings[0].thumbnailCaption.value;
{% endhighlight %}

Et pour finir nous créons chaque élément contenant les informations sur la ville en prenant soin de les ajouter au DOM.
{% highlight javascript %}
//On crée le titre
newTitle = document.createElement("h2");
newTitle.innerHTML = label;

//On crée le résumé
newAbstract = document.createElement("div");
newAbstract.innerHTML = abstract;

//On crée le conteneur de l'image
newThumbnail = document.createElement("div");
newThumbnail.id = "thumbnail";
var img = new Image();
img.src = thumbnail;
img.alt = label;

//On crée la légende de l'image
newThumbnailCaption = document.createElement("p");
newThumbnailCaption.id = "thumbnailCaption";
newThumbnailCaption.innerHTML = thumbnailCaption;

document.getElementById("infos").appendChild(newTitle);
document.getElementById("infos").appendChild(newAbstract);
document.getElementById("infos").appendChild(newThumbnail);
document.getElementById("thumbnail").appendChild(img);
document.getElementById("thumbnail").appendChild(newThumbnailCaption);
{% endhighlight %}

## Aperçu de notre application
![Infos de Rennes]({{ site.url }}/assets/img/exemple_enrichir_DBpedia2.png)

## Et pour finir
J'espère que cela vous aura donné un premier aperçu de ce qu'il est possible de faire grâce aux données récupérées sur DBpedia. Bien sûr il existe d'autres sources libres de données en RDF ainsi que bien d'autres façon de les utiliser, j'écrirais prochainement un article là-dessus.

### Notes
Le code de cette petite application est disponible sur GitHub :
[Dépôt GitHub](https://github.com/Mileem/enrichir-page-web-DBpedia)

### Exercices
Essayer d'ajouter d'autres villes dans un premier temps. Puis essayez de prendre d'autres type de ressources (planète, region, etc.) afin de varier l'exercice.


[Crédit photo : archangel12](https://www.flickr.com/photos/archangel12/9549651052/in/photolist-fxSsy5-fs72mg-siNvk7-7JvnWT-5kW9Kz-67UYEa-ok6R43-5sv4K2-btSHq7-8aCDnP-6pSW8R-29Rmoo-sQYdho-83hKxA-nVK4Ra-9XQVMX-cTxhoL-cm4mHm-rR5jZN-6y2g9m-s4D2Qx-61ed3M-dsv6PQ-hhUv3p-89BHdq-azJHR1-9ErfFE-nLi7cK-bztqB6-rhAkCC-cKsZG5-4F4ad1-9o39nM-rg6YxL-5wUiYb-cQ8ReE-4h9WgM-83eWFQ-3KLNZ-5BJ5aG-bztpfT-5kkd9o-JRayz-fxSCd5-arqVq9-6HynxT-a8JxmL-s4Ddne-7Sr7bn-9UaZVu )
