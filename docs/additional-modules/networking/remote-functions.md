---
title: Remote Functions
---
RemoteFunctions are for two way communicates between the server and client. This means the sender is able to receive a response from the receiver. Flamework's RemoteFunctions implementation use promises which allow you to avoid any dangerous yields, errors, etc. All requests have a timeout of 10 seconds.

Whilst it is not recommended, Flamework does support `ServerToClient` remote functions. Flamework avoids common pitfalls by implementing timeouts and cancellation (such as a player leaving) using promises.

## Creation
You can use the `Networking.createFunction` macro to create your network handler. This will contain all your functions for both server and client and you can also configure your [middleware](./middleware).

```ts
// shared/networking.ts
import { Networking } from "@flamework/networking";

interface ClientToServerFunctions {
	myClientToServerFunction(param1: string): number;
}

interface ServerToClientFunctions {
	myServerToClientFunction(param1: string): number;
}

// Returns an object containing the `createServer` and `createClient` fields.
export const GlobalFunctions = Networking.createFunction<ClientToServerFunctions, ServerToClientFunctions>();

// It is recommended that you call `createServer` and `createClient` on the server and client respectively,
// which will avoid exposing server configuration (including type guards) to the client. See the Using Functions section below.
export const ServerFunctions = GlobalFunctions.createServer({ /* server config */ });
export const ClientFunctions = GlobalFunctions.createClient({ /* client config */ });
```

## Using Functions
Once you've declared all your functions, it's time to use them. You can access your functions by simply indexing the object returned by createServer or createClient respectively.

```ts
// server/networking.ts
import { GlobalFunctions } from "shared/networking";

export const Functions = GlobalFunctions.createServer();
```

```ts
// client/networking.ts
import { GlobalFunctions } from "shared/networking";

export const Functions = GlobalFunctions.createClient();
```

### Invoking Functions
Invoke a request and wait for a response.

```ts
// Invoke a function
Functions.myFunction.invoke("my parameter!").then((value) => ...);

// Shorthand syntax, equivalent to invoke
Functions.myFunction("my parameter!").then((value) => ...);

// Predict, invokes the callback
Functions.myFunction.predict("my parameter!").then((value) => ...);
```

### Handling Functions
You can only connect one handler to each function. Calling `setCallback` more than once will override the existing handler but will result in a warning being outputted.

```ts
// With a normal function
Functions.myFunction.setCallback((param1) => {
	print(param1);
	return math.random(1, 100);
})

// With an async function
Functions.myFunction.setCallback(async (param1) => {
	print(param1);
	return await myAsyncNumberGenerator(1, 100);
})
```

## Errors
Flamework's networking exposes a `NetworkingFunctionError` enum which is used whenever a RemoteFunction request is rejected.

| Name          | Description                                                                    |
|---------------|--------------------------------------------------------------------------------|
| Timeout       | The request surpassed the timeout length.                                      |
| Cancelled     | The request was cancelled by the receiver.                                     |
| BadRequest    | The request was rejected by the receiver due to invalid arguments.             |
| InvalidResult | The request was processed by the receiver, but an invalid result was returned. |
| Unprocessed   | The request was not processed by the receiver.                                 |

### Handling errors
Flamework's RemoteFunctions return promises which allows you to handle them the same as any other promise.
Flamework always passes a `NetworkingFunctionError` as the rejection value, which tells you the reason the request failed.

```ts
Functions.myFunction.invoke()
	.then((value) => print("I successfully got", value))
	.catch((reason) => {
		if (reason === NetworkingFunctionError.Timeout) {
			warn("My request timed out!");
		} else {
			warn("A different error occurred:", reason);
		}
	})
```
