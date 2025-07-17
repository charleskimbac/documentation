---
title: Remote Events
---
RemoteEvents are for one way communication between the server and client. You should use these when you do not need a response from the receiver.

## Creation
You can use the `Networking.createEvent` macro to create your network handler. This will contain all your events for both server and client and you can also configure your [middleware](./middleware).

It should be noted that you cannot return a value from an event and the return type on the interfaces will be ignored.
If you want two way communication then you should use [RemoteFunctions](./remote-functions).

```ts
// shared/networking.ts
import { Networking } from "@flamework/networking";

interface ClientToServerEvents {
	myClientToServerEvent(param1: string): void;
}

interface ServerToClientEvents {
	myServerToClientEvent(param1: string): void;
}

// Returns an object containing the `createServer` and `createClient` fields.
export const GlobalEvents = Networking.createEvent<ClientToServerEvents, ServerToClientEvents>();

// It is recommended that you call `createServer` and `createClient` on the server and client respectively,
// which will avoid exposing server configuration (including type guards) to the client. See the Using Events section below.
export const ServerEvents = GlobalEvents.createServer({ /* server config */ });
export const ClientEvents = GlobalEvents.createClient({ /* client config */ });
```

### Unreliable Events
Flamework supports specifying [unreliable remote events](https://create.roblox.com/docs/reference/engine/classes/UnreliableRemoteEvent).
These events must still follow any limits specified by Roblox (e.g the 1000 byte size limit).

You can specify that an event is unreliable using the `Networking.Unreliable` type.

```ts
interface ClientToServerEvents {
	myUnreliableEvent: Networking.Unreliable<(param1: string) => void>;
}
```

## Using Events
Once you've declared all your events, it's time to use them. You can access your events on the server or client by simply indexing the object returned by `createServer` or `createClient` respectively.

```ts
// server/networking.ts
import { GlobalEvents } from "shared/networking";

export const Events = GlobalEvents.createServer();
export const Functions = GlobalFunctions.createServer();
```

```ts
// client/networking.ts
import { GlobalEvents } from "shared/networking";

export const Events = GlobalEvents.createClient();
export const Functions = GlobalFunctions.createClient();
```


### Firing Events
Send a request between the server and client.

#### Server
```ts
import { Events } from "server/networking.ts";

// Fire to player(s)
Events.myServerToClientEvent.fire(player, ...args);
Events.myServerToClientEvent.fire([player1, player2], ...args);

// Fire to all players except
Events.myServerToClientEvent.except(player, ...args);
Events.myServerToClientEvent.except([player1, player2], ...args);

// Broadcast
Events.myServerToClientEvent.broadcast(...args);

// Predict, fires server event using player as the sender
Events.myServerToClientEvent.predict(player, ...args);

// Shorthand syntax, equivalent to Events.myServerToClientEvent.fire
Events.myServerToClientEvent(player, ...args);
```

#### Client
```ts
import { Events } from "client/networking.ts";

// Fire to server
Events.myClientToServerEvent.fire(...args);

// Predict, fires client event from the client
Events.myClientToServerEvent.predict(...args);

// Shorthand syntax, equivalent to Events.myClientToServerEvent.fire
Events.myClientToServerEvent(...args);
```

### Connecting
Connecting to an event returns a RBXScriptConnection which can be used to disconnect the event at any time.

Connecting events on the server and client is identical except that clients do not have an additional `player` parameter.

#### Server
```ts
// Connect to an event
Events.myServerToClientEvent.connect((player, arg1) => {
	print(player, arg1);
});

// Disconnect an event connection
const myConnection = Events.myServerToClientEvent.connect(() => {});
myConnection.Disconnect();
```
