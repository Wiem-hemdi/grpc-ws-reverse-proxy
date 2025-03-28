# gRPC WebSocket Reverse Proxy

This project implements a reverse proxy that connects WebSocket clients to a gRPC server using a WebSocket server to facilitate communication. It provides a way to handle real-time messaging through WebSockets while interacting with a gRPC service for user data and chat history.

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [1. Install Dependencies](#1-install-dependencies)
  - [2. Set up the gRPC Server](#2-set-up-the-grpc-server)
  - [3. Set up the WebSocket Server](#3-set-up-the-websocket-server)
  - [4. Run the Application](#4-run-the-application)
- [Usage](#usage)
- [Error Handling](#error-handling)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Introduction

This project includes:
- A **gRPC server** that provides chat functionality and user data.
- A **WebSocket reverse proxy** server that connects WebSocket clients to the gRPC server, allowing real-time communication.

### Features:
- Real-time messaging via WebSockets.
- Fetch chat history using the `GetChatHistory` gRPC method.
- Streaming bidirectional communication between clients and the gRPC server.

---

## Prerequisites

Before you begin, ensure you have the following installed on your system:

- [Node.js](https://nodejs.org/) (Version 16+ recommended)
- [npm](https://www.npmjs.com/) (Comes with Node.js)
- gRPC and WebSocket dependencies for Node.js
- A basic understanding of gRPC, WebSockets, and Node.js

---

## Setup

### 1. Install Dependencies

First, clone the repository to your local machine:

```bash
git clone https://github.com/yourusername/grpc-ws-reverse-proxy.git
cd grpc-ws-reverse-proxy
npm install
```

### 2. Set up the gRPC Server

The gRPC server is defined in `server.js`. It listens for requests from the reverse proxy and handles chat and user retrieval functionalities.

- The gRPC server responds to `GetChatHistory` and `Chat` RPC calls.
- It stores the chat history in memory.

**gRPC Port:** The server listens on port `50051`. If needed, you can change the port by modifying the line:

```javascript
server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => { ... });
```

### 3. Set up the WebSocket Server

The WebSocket server is defined in `proxy.js`. It acts as a reverse proxy, handling WebSocket connections from clients and forwarding requests to the gRPC server.

- It listens for WebSocket connections on port `8080` (or `8082` if port `8080` is unavailable).
- It facilitates communication between WebSocket clients and the gRPC server by acting as a bridge.

**WebSocket Port:** The server listens on port `8080`. If this port is already in use, change it to another available port (like `8082`).

```javascript
const wss = new WebSocket.Server({ port: 8080 });
```

### 4. Run the Application

After setting up both servers, run them individually:

#### Start the gRPC server:

```bash
node server.js
```

The server should log:

```nginx
Serveur gRPC en écoute sur 0.0.0.0:50051
```

#### Start the WebSocket server (reverse proxy):

```bash
node proxy.js
```

The server should log:

```nginx
Reverse proxy WebSocket en écoute sur ws://localhost:8080
```

Now, the WebSocket server is ready to accept WebSocket connections from clients, and the gRPC server is listening for requests.

## Usage

Once both servers are running, WebSocket clients can connect to `ws://localhost:8080` and send/receive messages in real-time.

### WebSocket Client Example

A simple WebSocket client might look like this (using JavaScript for the front-end):

```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  console.log('Connected to WebSocket server');
  // Request chat history from gRPC server
  ws.send(JSON.stringify({ get_chat_history: { room_id: 'room1', limit: 10 } }));
};

ws.onmessage = (message) => {
  console.log('Received:', message.data);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('WebSocket connection closed');
};
```

### gRPC Methods:

- **`GetChatHistory`**: Fetch the last `n` messages from a specific room.

Example Request:

```json
{ "room_id": "room1", "limit": 10 }
```

- **`Chat`**: Allows real-time bidirectional communication. Messages sent via WebSocket are streamed to the gRPC server, and the server replies with an echo message.

---

## Error Handling

Here are some common error codes you may encounter:

- **`EADDRINUSE`**: This error occurs when the WebSocket server port is already in use. Change the port in `proxy.js` and try again.
- **`14 UNAVAILABLE`**: This error occurs when the gRPC server cannot be reached. Ensure the gRPC server is running and accessible on the correct address and port.
- **JSON Parse Errors**: Ensure messages sent from the client to the WebSocket server are valid JSON. Invalid JSON will result in a parsing error.

---

## Troubleshooting

### Issue: "No connection established"
- **Cause**: The WebSocket server cannot connect to the gRPC server.
- **Solution**: Ensure the gRPC server is running at the correct port (`50051` by default). Check for network issues or firewall restrictions.

### Issue: "Address already in use"
- **Cause**: The WebSocket server port is already being used by another process.
- **Solution**: Change the port in `proxy.js` to a free port (e.g., `8082`) and restart the server.

---

## wiem hemdi
