# Architecture CollabScore

Voici une proposition pour l'architecture CollabScore, résumée globalement par la figure suivante qui décompose les fonctionnalités en modules. La couleur du cadre autour d'un module indique qu'il est sous la responsabilité du partenaire correspondant (cf. légende).

![Architecture CollabScore](/figures/architecture.png)

## Le serveur CollabScore (Cnam)

Le Cnam met en place un serveur CollabScore constitué

 1. d'un dépôt de partitions multimodales
 2. de fonctionnalités internes développées par le Cnam visant notament à permettre la réconciliation de corrections collaboratives
 3. d'une interface Web pour les interactions utilisateurs, 
 4. et d'une interface de services REST pour les communications avec tous les modules développés par les autres partenaires du projet.

L'architecture à base de services permet à chaque partenaire de développer ses outils indépendamment, la seule contrainte technique étant
d'établir un canal de communication avec le serveur CollabScore via ses services.

### Les partitions multimodales

La notion de partition multimodale est centrale dans le projet (figure ci-dessus). Elle est basée sur la *partition pivot*, un codage de
la notation musicale décrivant le contenu d'une œuvre musicale (sous forme de notation) indépendamment des paramètres de présentation. 
En première approche on peut considérer qu'il s'agit d'un document MEI dépouillé autant que faire se peut de toute instruction de rendu. Le Cnam fournira
une spécification précise prochainement.

Dans ce document les éléments notationels (éléments-pivots) relatifs au contenu (mesures, notes, silences et accords, métrique, altérations, peut-être liaisons) 
sont tous identifiables et donc référençables sous la forme d'IRI (cf. les propositions de l'IReMus [1]).
 
![Partition multimodale](/figures/partitionMM.png)

Une partition-pivot est issue d'une *source originale* (une image, scénario principal de CollabScore, mais on pourrait en toute généralité aussi considérer 
un document audio si on faisait de la transcription) à partir de laquelle son contenu est constitué par un processus de numérisation. 
Pour CollabScore, la source originale est une image provenant de la BnF ou de la Fondation Royaumont, et le processus est une combinaison d'OMR 
et de correction collaborative.

### Les annotations

La partition-pivot est complétée par des annotations qui établissent des liens entre les éléments-pivot et des fragments de sources multimodales (images,
audio, vidéos, sources littéraire). La seule condition est que les fragments soient identifiables par une IRI, et que leur granularité 
corresponde à celle des éléments-pivot (en principe, la granularité sera la mesure).

Les annotations seront conformes au modèle du W3C. En particulier, les services REST de CollabScore permettront de déposer / récupérer les annotations
au format JSON-LD.

## Les modules

### Interface (UI) d'alignement entre une partition-image et le pivot (IReMus, fin 2021)

Cette interface permettra de lier les boites englobantes (BB) des mesures sur une partition-image et la partition-pivot. Elle interrogera le serveur
CollabScore pour obtenir la partition-pivot et l'URL d'une source-image, calculera l'ensemble des liens entre une BB de mesure et 
l'élément correspondant dans la partition-pivot, et enverra l'ensemble de ces liens sous forme d'annotation au serveur CollabScore.

**TODO**: 

 - Donner un exemple d'annotation liant une BB et une mesure de la partition-pivot).
 - Est-il nécessaire que la source-image soit accessible via un serveur IIIF ? À éclaircir. L'adressage des  BB devrait certainement être fait en 
   respectant l'adressage IIIF.
 - Concertation préalable avec l'IReMus.

Il serait sans doute très utile de récupérer des outils existants ([3],[4]) ou de collaborer avec des groupes qui ont déjà travaillé sur un
sujet équivalent (cf. Dezrann, développé par AlgoMus [5]).

### Interface (UI) d'alignement entre un audio ou vidéo et le pivot (Cnam, mi-2022)

L'idée est la même mais cette fois il faut associer à chaque mesure de la partition-pivot un fragment (période) d'un document audio ou d'une vidéo. Il
est possible / probable qu'il existe déjà des procédures d'alignement automatique.

**TODO**: essayer de reprendre contact avec Antescofo, sinon étudier l'état de l'art.

### Interface (UI) de synchronisation des sources (Cnam + BnF, fin 2022)

Via la partition-pivot, il sera possible de mettre en correspondance deux sources quelconques. 

   - une source-image et le pivot lui-même (affiché sous forme de partition)
   - une source-image et un document audio (dont, cas particulier, un MIDI produit à partir du pivot)
   - une source-image et une vidéo

Une interface doit permettre une synchronisation de deux sources quelconques. Les scénarios:

  - source image à gauche, partition-pivot à droite. Tout défilement ou action sur l'un (par exemple déclenchement
    d'une écoute MIDI sur la partition pivot) entraîne un alignement avec l'autre (p.e. surlignage des mesures sur l'image).
  - source image à gauche, audio à droite. L'écoute de l'audio entraîne le surlignage des mesures sur l'image
  - même scénario avec un document vidéo à droite.

Le serveur CollabScore fournit la séquence des annotations appariées: pour chaque élément-pivot (mesure), le fragment de la
source A et le fragment de la source B. L'interface doit assurer visuellement l'alignement.

Le Cnam doit réaliser cette interface, mais en vue d'une intégration facile dans Gallica. La BnF dispose d'un petit financement
pour l'adaptation selon les normes IIIF.

**TODO**: 

  - cas des sources littéraires à étudier
  - concertation avec la BnF sur les choix techniques

### Le module OMR (IRISA, toute la durée du projet)

Le module OMR est à peu près entièrement à la charge de l'IRISA. Le principe de l'intégration au reste de 
l'architecture est le suivant: le module OMR interroge le serveur CollabScore pour obtenir des partitions multimodales.
Pour chacune, la source originale peut être récupérée par URL, traitée par l'OMR, avec production d'une partition
pivot (document MEI) et sans doute des annotations indiquant les parties à confirmer ou contrôler. 

La partition pivot et les annotations sont transmises au serveur CollabScore via les services REST.

**TODO**:
  
   - Proposition très générale, à vérifier dans le détail
   - Question: obtiendra-t-on automatiquement les liens entre BB de la source-origine et les éléments-pivot ?
   - Concertation avec l'IRISA sur les détails techniques

### UI correction collaborative (Cnam)

Module à la charge du Cnam. A creuser.

# Références

[1]: https://github.com/Amleth/source-sherlockizer-service(Some technical thoughts on processing MEI sources to facilitate their future scholarly semantic annotation)
[2]: https://www.w3.org/TR/annotation-model/(Web annotation data model)
[3]: https://www.audiolabs-erlangen.de/resources/MIR/2019-ISMIR-LBD-Measures(Démo d'un outil pour oîtes englobantes, ISMIR)
[4]: https://dl.acm.org/doi/10.1145/3429743(Schubert Winterreise Dataset: A Multimodal Scenario for Music Analysis)
[5]: https://gitlab.com/algomus.fr/dezrann/(Dezrann, l'interface d'annotation AlgoMus)

