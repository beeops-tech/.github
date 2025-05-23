# Fonctionnement basique

![Design de l'application](../archi-design.png)

# Repositories de l'organisation : 
- backend-spring (java)
- bee (golang)
- frontend (typescript, vuejs)
- webcomponent-boilerplate (typescript, lit) : Starter permettant de rapidement générer des web components avec lit et vite. Afin de les utiliser dans les dashboards.
- landing-page (astro, vuejs) : Page d'accueil de projet, exposé sur internet
- contracts (deprecated) Utilisé a la base pour partager en tant que module, les clients/serveurs grpc
- backend (deprecated) Version initiale du backend écrit en go, depuis migré en java

# Backend

## Architecture logicielle
Mix entre une architecture hexagonale et une clean architecture.
![Workflow basique de l'architecture back](../workflow_back.png)

## Point notable
Voici les quelques points notables du backend.

### Choix de mongodb et DDD

### Implémentation de server sent event comme vecteur principal de communication backend -> front.
Voir justification front

### Utilisation de l'Aspect Oriented Programming pour le HiveAccessControl
Lors de la connection, on renvoi a l'utilisateur un JWT avec dedans sa HiveId.
Afin de faire en sorte qu'un utilisateur avec un JWT valide ne puisse pas accéder a une autre hive. On a mis en place l'annotation @HiveAccessControl sur les méthodes des controllers concerné.
Sur ces méthodes, grâce a l'[AOP](https://fr.wikipedia.org/wiki/Programmation_orient%C3%A9e_aspect), on va appliquer en ammont de chaque appel de la méthode, le traitement suivant : 

- Récupération du JWT
- Récupération de la HiveId dans le JWT
- Récupération du HiveId de la requête
- Comparaison de ces deux champs.

Cela nous permet d'appliquer ce comportement uniquement en apposant une annotation sur notre endpoint


### GlobalExceptionHandler et ProblemDetail
On a centraliser la gestion des exception dans le fichier application/http/GlobalExceptionHandler.
L'idée est de catcher l'exception noté par @ExceptionHandler, et de renvoyer un message d'erreur.
On fait le choix de renvoyer un ProblemDetail ([RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807)), qui s'annonce comme être un futur standard des messages d'erreurs d'api.

### Génération des deux specs open api.
En tant que backend, on a deux type d'acteurs qui peuvent nous contacter. Les bees, et les clients fronts.

Nous avons fais le choix de séparer nos controllers en deux (application/http/bee & application/http/client pour le front).
La raison de ça c'est de pouvoir générer 2 specs open api spécifiques a chaque acteurs.

Elles sont disponibles sur ce lien ([LienOpen Api en local sur le back](http://localhost:8080/api/v1/api-docs/ui)).
La séparation se fait dans la classe application/http/OpenApiConfig en fonction des routes.


# Ouverture sur l'architecture
## Problématique
A cause des sessions http de server sent event, notre backend devient statefull.
- Impossible donc de scaler des réplicas.

## Solution
![Solution eda](../stateless.png)
- Mettre un RabbitMQ entre un nouveau service pour la notification et notamment le server sent event (qui lui est statefull).
- Ce rabbitMq est branché au backend qui va envoyer les messages dans le bus, afin qu'ils soient consommés par le service de server sent event, et envoyé au front.

## Pourquoi on le met pas en place
- Manque d'utilisateur/charge => Pas nécessaire de pouvoir scaler le backend, car notre service n'est pas sensé recevoir de la charge.
- Complexité du SI: Passage de 3 services a 6, ce qui rend de moins en moins portable le service.
