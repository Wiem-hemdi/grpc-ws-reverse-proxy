# gRPC WebSocket Reverse Proxy

## TP5 : Reverse Proxy WebSocket avec microservice gRPC

Ce projet implémente un proxy inverse permettant aux clients WebSocket de communiquer avec un serveur gRPC. Il facilite la messagerie en temps réel via WebSockets tout en interagissant avec un service gRPC pour la gestion des utilisateurs et l'historique des conversations.

## Table des Matières
- [Introduction](#introduction)
- [Prérequis](#prérequis)
- [Installation et Configuration](#installation-et-configuration)
  - [1. Installation des Dépendances](#1-installation-des-dépendances)
  - [2. Configuration du Serveur gRPC](#2-configuration-du-serveur-grpc)
  - [3. Configuration du Serveur WebSocket](#3-configuration-du-serveur-websocket)
  - [4. Lancement de l'Application](#4-lancement-de-lapplication)
- [Utilisation](#utilisation)
- [Gestion des Erreurs](#gestion-des-erreurs)
- [Dépannage](#dépannage)
- [Licence](#licence)

## Introduction

Ce projet inclut :
- Un **serveur gRPC** qui gère les utilisateurs et les messages de chat.
- Un **proxy WebSocket** qui relaie les messages entre les clients WebSocket et le serveur gRPC.

### Fonctionnalités :
- Messagerie en temps réel via WebSockets.
- Récupération de l'historique des messages avec `GetChatHistory`.
- Communication bidirectionnelle en streaming entre les clients et le serveur gRPC.

---

## Prérequis

Avant de commencer, assurez-vous d'avoir les outils suivants installés sur votre système :

- [Node.js](https://nodejs.org/) (Version 16+ recommandée)
- [npm](https://www.npmjs.com/) (fourni avec Node.js)
- gRPC et WebSocket pour Node.js
- Des notions de base sur gRPC, WebSockets et Node.js

---

## Installation et Configuration

### 1. Installation des Dépendances

Clonez le projet et installez les dépendances :

```bash
mkdir grpc-ws-reverse-proxy
cd grpc-ws-reverse-proxy
npm init -y
npm install @grpc/grpc-js @grpc/proto-loader ws
```

---

### 2. Configuration du Serveur gRPC

Créez un fichier `chat.proto` avec le contenu suivant :

```proto
syntax = "proto3";
package chat;

enum UserStatus {
  UNKNOWN = 0;
  ACTIVE = 1;
  INACTIVE = 2;
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
  UserStatus status = 4;
}

message ChatMessage {
  string id = 1;
  string room_id = 2;
  string sender_id = 3;
  string content = 4;
}

message GetUserRequest {
  string user_id = 1;
}

message GetUserResponse {
  User user = 1;
}

message ChatStream {
  oneof payload {
    ChatMessage chat_message = 1;
  }
}

service ChatService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc Chat(stream ChatStream) returns (stream ChatStream);
}
```

Ajoutez ensuite le serveur gRPC dans `server.js`.

---

### 3. Configuration du Serveur WebSocket

Créez `proxy.js` pour gérer les connexions WebSocket et transmettre les messages au serveur gRPC.

---

### 4. Lancement de l'Application

Démarrez les deux serveurs :

```bash
node server.js
node proxy.js
```

Vous devriez voir les messages suivants :

```bash
Serveur gRPC en écoute sur 0.0.0.0:50051
Reverse proxy WebSocket en écoute sur ws://localhost:8080
```

## Utilisation

Les clients WebSocket peuvent se connecter à `ws://localhost:8080` pour envoyer et recevoir des messages en temps réel.

Exemple de client WebSocket :

```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  console.log('Connecté au serveur WebSocket');
  ws.send(JSON.stringify({ get_chat_history: { room_id: 'room1', limit: 10 } }));
};

ws.onmessage = (message) => {
  console.log('Reçu:', message.data);
};

ws.onerror = (error) => {
  console.error('Erreur WebSocket:', error);
};

ws.onclose = () => {
  console.log('Connexion WebSocket fermée');
};
```

---

## Gestion des Erreurs

### Erreur de Port `EADDRINUSE`

Si vous voyez cette erreur :

```bash
Error: listen EADDRINUSE: address already in use :::8080
```

Cela signifie que le port est déjà utilisé. Vous pouvez résoudre le problème en changeant le port de `proxy.js` :

```javascript
const wss = new WebSocket.Server({ port: 8082 });
```

Relancez ensuite `node proxy.js`.

### Erreur `14 UNAVAILABLE`

Si le client WebSocket ne parvient pas à se connecter au serveur gRPC, vérifiez que `server.js` fonctionne bien et écoute sur `0.0.0.0:50051`.

---

## Dépannage

| Problème | Cause Possible | Solution |
|----------|---------------|----------|
| "No connection established" | Le serveur gRPC ne fonctionne pas | Assurez-vous que `server.js` est lancé |
| "Address already in use" | Port 8080 déjà utilisé | Changez le port WebSocket dans `proxy.js` |
| "Invalid JSON format" | Message mal formatté | Vérifiez que le client envoie du JSON valide |

---

## wiem hemdi

