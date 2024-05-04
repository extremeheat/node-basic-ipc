# node-basic-ipc
[![NPM version](https://img.shields.io/npm/v/basic-ipc.svg)](http://npmjs.com/package/basic-ipc)
[![Build Status](https://github.com/extremeheat/basic-ipc/actions/workflows/ci.yml/badge.svg)](https://github.com/extremeheat/basic-ipc/actions/workflows/)
[![Gitpod ready-to-code](https://img.shields.io/badge/Gitpod-ready--to--code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/extremeheat/basic-ipc)

Node.js real-time client <-> server (between backend and browser) communication library for IPC.

Features:
- Send and receive JSON objects or binary data
- Send and receive messages in chunks
- Request-response pattern


## Install
```bash
npm install basic-ipc
```

## Usage
Simple example of server and client communication using WebSocket:
```javascript
const ipc = require('basic-ipc');
const server = ipc.createServer({
  ws: { port: 8091 }
})

server.once('listening', () => {
  const client = ipc.createClient({
    ws: { url: 'ws://localhost:8091' }
  })
  client.once('open', () => {
    client.sendMessage('hello', { world: '!' })
  })
})

server.on('connection', async (client) => {
  client.receive('hello', (message) => {
    console.log('Received hello message:', message)
    server.close()
    client.close()
  })
})
```

## API
```ts
import { WebSocket } from 'ws'

export class Client extends WebSocket {
  // Write a JSON object to the WebSocket
  write(data: object): void

  // Send a JSON object message in full to the server
  sendMessage(messageType: string, contents: object): void
  // Send a JSON object message in chunks to the server
  createMessage(messageType: string): {
    sendChunk(chunk: object): void
    sendResponse(response: object): void
  }

  // Send a binary message in full to the server
  sendBinaryMessage(messageType: string, contents: Buffer): void
  // Send a binary message in chunks to the server
  createBinaryMessage(messageType: string): {
    sendChunk(chunk: Buffer): void
    sendResponse(response: Buffer): void
  }

  // Send a message to server & get response
  request(messageType: string, contents: object, chunkCb?: (obj: object) => void, timeout?: number): Promise<object>
  // Send a message to server & get response in binary
  requestBinary(messageType: string, contents: Buffer, chunkCb?: (obj: Buffer) => void, timeout?: number): Promise<Buffer>

  // Receive a message from the server (one not asked for)
  receive(messageType: string, cb: (obj: object) => void): void
  // Receive a binary message from the server (one not asked for)
  receiveBinary(messageType: string, cb: (obj: Buffer) => void): void
}

export class Server extends WebSocket.Server {
  on(event: 'connection', listener: (client: Client) => void): this
}

export interface WSClientOptions {
  url: string
}
export interface WSServerOptions {
  port: number
}

export function createClient(options: { ws: WSClientOptions }): Client
export function createServer(options: { ws: WSServerOptions }): Server
```

## License
MIT