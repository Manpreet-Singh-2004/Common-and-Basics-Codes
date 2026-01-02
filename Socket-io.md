# Web Socket and Socket.io

# Difference between Socket and HTTP
Http is one way half duplex mode of communication, there is req and res. the client requests and server gives responses.
But in Socket, it is a full duplex mode of communication, meaning the server dosent have to wait for client to request something again, it can just send the data on updation.

# Archeticture
Since the socket keeps listening to server for any changes, i believe that it is best suited for long running servers or a server arch. Meaning it is best paired with express backend application, though it can be used with other frameworks like Next Js

# Socket Io
## Socket vs IO
Socket is an individual client where as io reffers to the entire server/circuit. Which means if you do `io.emit()` the message is sent to all the connected clients in the circuit. And when we use `socket.emit()` the message is sent only to that specific client, and each individual socket has its own socket.id.

## Emit vs On

`emit`: Used to trigger an event and optionally send data along with it.

`on`: Used to listen for a specific event and execute a function when that event is triggered, receiving any data sent with it.

**Server**
```js
io.emit("event-one", "Hi");
``` 
Emits "event-one" to all connected clients with data "Hi".

**Client**
```js
socket.on("event-one", (message) => {
console.log(message); // Receives "Hi"
});
```
emit can also be used from the client side to send data to the server

*Client emit and Server on:*

```js
// Client Side 
socket.emit("btn-event", someRandomNumber); // Emits "btn-event" to the server with data

// Server Side 
socket.on("btn-event", (n) => {
// Do something with 'n'
});
```

## socket.broadcast.emit:

Used to send a message to all clients except the sender. If a client (socket) triggers an event, and the server broadcasts that event, the original sender will not receive it, but all other connected clients will.

## socket.to().emit:

Enables private messaging or sending messages to specific groups. Every individual user (socket) is considered to be in their own default "room" with their socket.id as the room name.

`socket.to(recipientSocketId).emit("message", "Hello!");`

You can also explicitly join multiple sockets into a custom room.

```js
socket.join("room-name");
io.to("room-name").emit("group-message", "Welcome to the group!");
```

# Initialization

For Express app the boiler plate is as following :

```js
import express from 'express'
import { Server } from 'socket.io'
import {createServer} from 'http'

const port = 3000

const app = express()

const server = createServer(app)

const io = new Server(server)

io.on("connection", (socket) =>{

    console.log("user connected");
    console.log(`Id: ${socket.id}`)
})

app.get("/", (req, res) => {
    res.send("Hello World")
})

app.listen(port, () =>{
    console.log(`Server is listening on port ${port}`)
})
```
