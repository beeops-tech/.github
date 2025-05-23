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



# Ouverture sur l'achitecture
## Problématique
A cause des sessions http de server sent event, notre backend devient staftefull

## Solution
![Solution eda](../stateless.png)
- Mettre un RabbitMQ entre un nouveau service pour la notification et notamment le server sent event (qui lui est statefull).
- Ce rabbitMq est branché au backend qui va envoyer les messages dans le bus, afin qu'ils soient consommés par le service de server sent event, et envoyé au front.

## Pourquoi on le met pas en place
- Manque d'utilisateur/charge => Pas nécessaire de pouvoir scaler le backend, car notre service n'est pas sensé recevoir de la charge.
- Complexité du SI: Passage de 3 services a 6, ce qui rend de moins en moins portable le service.
