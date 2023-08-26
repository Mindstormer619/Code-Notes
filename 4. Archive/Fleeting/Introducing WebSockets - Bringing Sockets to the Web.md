# Introducing WebSockets - Bringing Sockets to the Web
> Created: 2023-06-14 16:48

The [WebSocket](http://dev.w3.org/html5/websockets/) specification defines an API establishing "socket" connections between a web browser and a server. In plain words: There is an persistent connection between the client and the server and both parties can start sending data at any time. 

It avoids the overhead of HTTP, which is a problem for techniques like long-polling. This makes WS well-suited to low-latency applications.

## Opening a connection

```js
var connection = new WebSocket('ws://html5rocks.websocket.org/echo', ['soap', 'xmpp']);
```

Notice the `ws:`. This is the new URL schema for WebSocket connections. There is also `wss:` for secure WebSocket connection the same way `https:` is used for secure HTTP connections.

Attaching some **event handlers** immediately to the connection allows you to know when the connection is opened, received incoming messages, or there is an error.

The second argument accepts optional sub-protocols. It can be a string or an array of strings. Each string should represent a sub-protocol name and server accepts only one of passed sub-protocols in the array. Accepted sub-protocol can be determined by accessing `protocol` property of WebSocket object.

```js
// When the connection is open, send some data to the server
connection.onopen = function () {
	connection.send('Ping'); // Send the message 'Ping' to the server
};

// Log errors
connection.onerror = function (error) {
	console.log('WebSocket Error ' + error);
};

// Log messages from the server
connection.onmessage = function (e) {
	console.log('Server: ' + e.data);
};
```

## Proxy Servers

The WebSocket protocol uses the HTTP upgrade system (which is normally used for HTTP/SSL) to 'upgrade' an HTTP connection to a WebSocket connection. Some proxy servers do not like this and will drop the connection. Thus, even if a given client uses the WebSocket protocol, it may not be possible to establish a connection.



----

## References
1. https://web.dev/websockets-basics/