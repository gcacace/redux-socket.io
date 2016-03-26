# redux-socket.io
An enhanced version of the opinionated connector between socket.io and Redux.

Philosophy
-------------
### Client → Server
Messages are sent to the server by dispatching actions to Redux store, where the action `type` has a specific prefix, `toSocket/` as default.

### Server → Client
Messages are listened from the server by dispatching actions to Redux store, where the action `type` has a specific prefix, `fromSocket/` as default.

> **Note:** This middleware must be instructed with the list of events coming from the server.

How to use
-------------
### Installation
```
npm install --save git+https://git@github.com/gcacace/redux-socket.io#1.0.0
```

### Example usage
This will create a middleware that sends actions to the server when the action type starts with `toSocket/`.
When the socket.io client receives a message with event name starting with `fromSocket/`, it will dispatch the action to the store.

The result of running this code from the client is a request to the server and a response from the server, both of
which go through the redux store's dispatch method.

####Client side:
```js
import { createStore, applyMiddleware } from 'redux';
import createSocketIoMiddleware from 'redux-socket.io';
import io from 'socket.io-client';
let socket = io('http://localhost:3000');
let socketIoMiddleware = createSocketIoMiddleware(socket, 'fromSocket/', 'toSocket/', ['fromSocket/pong']);

function reducer(state = {}, action) {
  switch(action.type) {
  
    // This event is dispatched from the redux-socket.io middleware only if listed into the configuration array
    // defined above (4th argument of the `createSocketIoMiddleware` function
    case 'fromSocket/pong':
      return Object.assign({}, { message: action.payload });
      
    default:
      return state;
  }
}
let store = applyMiddleware(socketIoMiddleware)(createStore)(reducer);

// This will send a message to the socket.io server, with event name `ping` and data `This is a ping!`
store.dispatch({type: 'toServer/ping' , payload: 'This is a ping!' });
```

####Server side:
```js
var http = require('http');
var server = http.createServer();
var socket_io = require('socket.io');
server.listen(3000);
var io = socket_io();
io.attach(server);

io.on('connection', function(socket) {
  console.log("Socket connected: " + socket.id);
  
  socket.on('ping', (message) => {
    console.log('Got ping data!', message);
    socket.emit('pong', message);
  });
});
```

License
-------------
Copyright © 2016 Gianluca Cacace

Copyright for portions of this project are held by Ian Taylor, 2015.

Permission is hereby granted, free of charge, to any person
obtaining a copy of this software and associated documentation
files (the “Software”), to deal in the Software without
restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.
