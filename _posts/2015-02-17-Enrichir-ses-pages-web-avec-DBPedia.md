---
layout: post
title: Enrichir ses pages web avec DBPedia
author: Mileem
categories:
- blog
---

Cet article présente comment enrichir ses pages web avec DBpedia simplement avec quelques lignes de javascript. En utilisant DBpedia, vous pourrez donc ajouter de la plus value à vos articles publiés sur le web.

----------

# Avec des vrais morceaux de code dedans ...

----------
![Manipuler un hibou]({{ site.url }}/assets/img/owl_01.jpg)


## Principes de base
A quoi peut servir la base de données DBPedia ?
Comment en extraire facilement du contenu intéressant et pertinent ?

Ces questions m'ont poussé à réaliser ce tutoriel qui explique très simplement comment récupérer des données sur DBPedia et les intégrer dans une page web.

Pour ce faire nous utiliserons du HTML et du Javascript uniquement. En effet, nous pouvons facilement récupérer des informations depuis DBPedia depuis des requêtes AJAX.

## URI ou URL ou les deux ? ou les TROIS ?
<!-- A REVOIR -->
Avant de commencer, nous allons voir ensemble la différence subtile entre URI et URL. Cette notion est importante à connaitre car on va beaucoup utiliser les URI par la suite .

>URI (ou Uniform Resource Identifier), soit identifiant uniforme de ressource est une chaîne de caractères servant d'identifiant à une ressource sur le réseau (par exemple une ressource Web sur le réseau internet).

Une URI un peu particulière :

>URL (ou Uniform Resource Location), soit localisateur uniforme de ressource est enfaite un URI de type "locator". C'est ce qu'on appelle communément une adresse web. Une URL nous permet non seulement d'identifier une page ou un site web mais également d'y accéder.

Par conséquent toute URL est une URI. Mais attention, toute URI n'est pas forcément une URL.

#### Prenons deux exemples concrets.
Si je veux la page DBPedia contenant toutes les informations à propos de la ville de Rennes, il faut que je me rende à l'**URL** suivante :

```
http://fr.dbpedia.org/page/Rennes
```

Si je veux récupérer toutes ces ressources, il va falloir que je me rende à cette autre adresse :

```
http://fr.dbpedia.org/resource/Rennes
```

Vous avez-vous vu ? Lorsqu'on clique sur le deuxième lien, l'adresse change dans la barre d'adresse et on se retrouve sur la même page que le premier lien. Bizarre, non ?
Enfaite, non. C'est logique. Notre navigateur sait pertinemment que ce à quoi nous voulons accéder, c'est la page sur la ville de Rennes et non les ressources sur la ville de Rennes.
Or, vous verrez que dans nos requêtes SPARQL, nous nous servirons de l'**URI** et non de l'**URL**.

<!-- Si vous voulez vous faire peur suivez le lien suivant :
[http://fr.dbpedia.org/data/Rennes.json]({http://fr.dbpedia.org/data/Rennes.json})

Oui, on va bientôt récupérer des données dans ce format là. -->


## La requête SPARQL
Tout d'abord commençons par étudier la requête SPARQL à envoyer au serveur.

Pour préparer notre requête, nous allons tout d'abord identifier ce que nous voulons récupérer.
Pour chaque ville nous avons son nom maintenant, nous voulons récupérer :


-Un bref' résumé sur la ville

-Une image de la ville ou d'un de ses monuments (et la légende qui l'illustre)

Bien sûr on peut récupérer bien d'autres informations mais on réalise cela à titre d'exemple.



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


## Demander les résultats uniquement en français
Le filtre que nous allons appliquer permet de récupérer uniquement les résultats en français.

```
FILTER( lang(?label) = "fr" && lang(?abstract) = "fr")
```

## Préparation de la requête en AJAX
Si vous n'avez jamais fait de requête en AJAX ou si vous ne savez tout simplement ce que ça signifie (non je ne parle pas de produit ménager), je vous conseille de faire un tour par ici :
[https://developer.mozilla.org/en-US/docs/AJAX/Getting_Started]({https://developer.mozilla.org/en-US/docs/AJAX/Getting_Started})

De manière plus générale, lorsque vous avez besoin d'une précision en HTML ou en JavaScript, je vous conseille fortement d'aller faire un tour sur le MDN (Mozilla Developer Network).

## Parser le résultat de la requête en JSON
Les données que nous récupérons depuis DBPedia sont au format JSON comme nous l'avons précisé dans l'URL de requête. Cependant, XMLHttpRequest prévoit seulement de récupérer les données au format XML (responseXML) ou au format texte (responseText). Nous devons donc parser nos données pour les récupérer dans un format JSON tout propre :

```
 var doc = JSON.parse(req.responseText);
```

Simple, n'est-ce pas ?
Maintenant, nous allons pouvoir manipuler facilement nos données. C'est parti !

## Afficher nos données
DOM, attention, nous voilà !!

Afin de créer une application <s>propre</s>, nous allons créer deux fichiers.
Le premier étant un fichier HTML, que nous allons nommer index.html
{% highlight html %}
   <!DOCTYPE html>
    <html xmlns="http://www.w3.org/1999/xhtml" lang="fr" xml:lang="fr" >
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
      <title>Titre de la page</title>
      <link rel="stylesheet" href="css/style.css">
      <!--[if lt IE 9]>
      	<script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
      <![endif]-->
    </head>
    <body>
    	<button id="target">Rennes</button>
    	<div id="abstract"></div>
    	<div id="thumbnail"></div>
    </body>
    <script type="text/javascript" src="script.js"></script>
    </html>
{% endhighlight %}

Et un deuxième fichier nommé script.js
{% highlight javascript %}
/**
 * Récupération de données depuis DBPedia dès qu'on clique sur le bouton
 * Puis affichage des données récupérées sur la page
 */
document.getElementById("target").onclick = function() {
	//URI vers notre ressource (ici la ville de Rennes)
	var uri = "http://fr.dbpedia.org/resource/Rennes";

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

	function myCode() {
	   if (req.readyState == 4) {
	      var doc = JSON.parse(req.responseText);
        var abstract =  doc.results.bindings[0].abstract.value;
        var thumbnail =  doc.results.bindings[0].thumbnail.value;
        var label = doc.results.bindings[0].label.value;

        var img = new Image();
        img.src = thumbnail;
        img.alt = label;
        document.getElementById("abstract").innerHTML = abstract;
        document.getElementById("thumbnail").appendChild(img);
	   }
	}
};
{% endhighlight %}

## Suite
Dans un prochain article, je vous propose d'améliorer ce code et de récupérer l'URI directement depuis le fichier HTML.
On pourra donc récupérer plus simplement les informations sur différentes villes et réaliser un code plus "propre".


Crédit photo : Antonio Cinotti
