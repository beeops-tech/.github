# Fonctionnement basique

![Design de l'application](../archi-design.png)

# Repository : 
- backend-spring (java)
- bee (golang)
- frontend (typescript)
- contracts (deprecated) Utilisé a la base pour partager en tant que module, les clients/serveurs grpc
- backend (deprecated) Version initiale du backend écrit en go, depuis migré en java


# Ouverture
## Problématique
A cause des sessions de server sent event, notre backend devient staftefull

## Solution
![Solution eda](../stateless.png)
Mettre un RabbitMQ entre un service de server sent event (qui lui est stafull)
Ce rabbitMq est branché au backend qui va envoyer les messages dans le bus, afin qu'ils soient consommés par le service de server sent event

## Pourquoi on le met pas en place
Manque d'utilisateur => Pas nécessaire de pouvoir scaler le backend
Complexité du SI: Passage de 3 services a 6.