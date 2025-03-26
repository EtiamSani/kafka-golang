# Système de Commande de Café avec Kafka et Go

Ce projet est une démonstration d'un système de traitement de commandes de café utilisant Kafka comme broker de messages et Go comme langage de programmation.

## Architecture du Système

```
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│               │     │               │     │               │
│   Producer    │────▶│     Kafka     │────▶│    Worker     │
│  (API REST)   │     │    Broker     │     │ (Consommateur)│
│               │     │               │     │               │
└───────────────┘     └───────────────┘     └───────────────┘
```

Le système est composé de trois composants principaux :

1. **Producer** : API REST qui reçoit les commandes de café et les envoie au broker Kafka
2. **Kafka Broker** : Gère la file d'attente de messages
3. **Worker** : Consomme les messages de la file d'attente et traite les commandes

## Flux de Données

```
┌─────────┐     ┌────────────┐     ┌───────────────┐     ┌────────────┐     ┌─────────┐
│         │     │            │     │               │     │            │     │         │
│ Client  │────▶│ HTTP POST  │────▶│  Topic Kafka  │────▶│  Consumer  │────▶│ Brewing │
│         │     │ /order     │     │coffee_orders  │     │            │     │ Coffee  │
│         │     │            │     │               │     │            │     │         │
└─────────┘     └────────────┘     └───────────────┘     └────────────┘     └─────────┘
```

## Composants du Projet

### Producer (API REST)

Le producer est une API REST écrite en Go qui expose un endpoint `/order` pour recevoir les commandes de café. Lorsqu'une commande est reçue, elle est sérialisée en JSON et envoyée à Kafka sous le topic `coffee_orders`.

**Structure d'une commande** :
```json
{
  "customer_name": "Jean Dupont",
  "coffee_type": "Espresso"
}
```

### Kafka Broker

Kafka agit comme un intermédiaire entre le producer et le worker. Il stocke les messages dans un topic nommé `coffee_orders`.

### Worker (Consommateur)

Le worker est un service Go qui écoute le topic `coffee_orders` et traite les commandes de café au fur et à mesure qu'elles arrivent.

## Installation et Démarrage

### Prérequis

- Docker
- Go (version 1.16 ou supérieure)

### Démarrer le système

Pour lancer l'ensemble du système, utilisez Docker Compose :

```bash
docker pull apache/kafka:3.7.0
docker run -p 9092:9092 apache/kafka:3.7.0
```

Cela démarrera trois conteneurs :
- Kafka broker
- Producer (API REST)
- Worker (Consommateur)

### Accéder à l'API

Une fois le système démarré, l'API REST est accessible à l'adresse `http://localhost:3000`.

## Utilisation

### Passer une commande

Pour passer une commande de café, envoyez une requête POST à l'endpoint `/order` :

```bash
curl -X POST http://localhost:3000/order \
  -H "Content-Type: application/json" \
  -d '{"customer_name":"Jean Dupont","coffee_type":"Espresso"}'
```


## Architecture Détaillée

### Diagramme de Séquence

```
┌─────────┐          ┌────────────┐          ┌────────┐          ┌────────────┐
│ Client  │          │  Producer  │          │ Kafka  │          │   Worker   │
└────┬────┘          └──────┬─────┘          └───┬────┘          └──────┬─────┘
     │                      │                    │                       │
     │ HTTP POST /order     │                    │                       │
     │─────────────────────>│                    │                       │
     │                      │                    │                       │
     │                      │ Send to topic      │                       │
     │                      │────────────────────>                       │
     │                      │                    │                       │
     │ JSON Response        │                    │                       │
     │<─────────────────────│                    │                       │
     │                      │                    │                       │
     │                      │                    │ Consume message       │
     │                      │                    │───────────────────────>
     │                      │                    │                       │
     │                      │                    │                       │ Process
     │                      │                    │                       │ order
     │                      │                    │                       │
└────┴────┘          └──────┴─────┘          └───┴────┘          └──────┴─────┘
```

## Structure du Code

### Producer (`producer/main.go`)

- **API REST** basée sur la bibliothèque standard `net/http`
- Endpoint `/order` pour recevoir les commandes
- Utilisation de la bibliothèque `sarama` pour interagir avec Kafka
- Sérialisation JSON des commandes

### Worker (`worker/main.go`)

- Consommateur Kafka basé sur `sarama`
- Consomme les messages du topic `coffee_orders`
- Traite les commandes (simulation de préparation de café)
- Gestion des signaux pour arrêt propre 