# `ts.app`

# AKA `Tradeshift App Messaging Protocol`

## API reference

### `ts.app.connect(clientIds)`

Connect to Server/Broker

`userId`, `companyId` and `appId` are already known by `Tradeshift.Chrome` AKA the Broker.

### `ts.app.onMessage(appId, topic, payload)`

Handle any messages sent to my `appId`

* `appId` (string, required): Incoming message sender
* `topic` (string, required): Incoming message topic
* `payload` (any, optional): Incoming message payload

### `ts.app.publish(message)`

Publish messages

* `message` (`ts.app.Message`, required)
  * `appId` (string, required): Message recipient (wildcards NOT allowed)
  * `topic` (string, required): Message topic (wildcards NOT allowed)
  * `payload` (any, optional): Message payload

### `ts.app.subscribe(subscriptions)`

Subscribe to appIds and/or topics

* `subscriptions` (`Array<ts.app.Subscription>``, required)
  * `appId` (string, optional if `topic` is defined): AppId to subscribe to (wildcards allowed)
  * `topic` (string, optional if `appId` is defined): Topic to subscribe to (wildcards allowed)
  * `handler` (function, required): Handle incoming messages for subscription, see `ts.app.onMessage`

### `ts.app.unsubscribe(unsubscriptions)`

Unsubscribe from appIds and/or topics

> Wildcards are only used to match to the exact subscriptions the Client has.

> If `handler` is supplied, unsubscription only succeeds if it matches the currently subscribed `handler`

* `unsubscriptions` (`Array<ts.app.Subscription>`, required)
  * `appId` (string, optional if `topic` is defined): AppId to unsubscribe from (wildcards allowed)
  * `topic` (string, optional if `appId` is defined): Topic to unsubscribe from (wildcards allowed)
  * `handler` (function, optional): Handler to deregister

### `ts.app.exchange(messages)`

Exchange message with specific apps

* `messages` (`Array<ts.app.Message>`, required)
  * `appId` (string, required): Message recipient (wildcards NOT allowed)
  * `topic` (string, required): Message topic (wildcards NOT allowed)
  * `payload` (any, optional): Message payload

### `ts.app.load(message)`

Load specific app, wait for and handle user interaction

* `message` (`ts.app.Message`, required)
  * `appId` (string, required): Message recipient (wildcards NOT allowed)
  * `topic` (string, required): Message topic (wildcards NOT allowed)
  * `payload` (any, optional): Message payload

## API use-cases

### Send message to a single app

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Tell Tradeshift.RaptorFactory to unleash 300 raptors
		client.publish({
			appId: 'Tradeshift.RaptorFactory',
			topic: 'unleash/raptors',
			payload: 300
		});
	} catch (e) {
		console.error(e);
	}
}
init();
```

### Send message to multiple apps

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Tell every Tradeshift app to unleash 500 godzillas
		client.publish({
			appId: 'Tradeshift.+',
			topic: 'unleash/godzilla',
			payload: 500
		});
	} catch (e) {
		console.error(e);
	}
}
init();
```

### Receive message from any app

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Handle messages directed at my appId
		client.onMessage = (appId, topic, payload) => {
			switch (topic) {
				case 'unleash/raptors':
					console.log(
						`${appId} sent the order: Time to unleash ${payload} raptors!`
					);
					break;
			}
		};
	} catch (e) {
		console.error(e);
	}
}
init();
```

### Receive message from specific apps and/or topics

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
  try {
    // Connect to Server
    const client = await tsApp.connect();

    // Subscribe to specific apps
    client.subscribe([
      {
        appId: 'Tradeshift.+', // /^Tradeshift.[0-9a-zA-Z_-]$/
        topic: '+/godzilla', // /^[0-9a-zA-Z_- ]*\/godzilla$/
        handler
      },
      {
        appId: 'Tradeshift.WizardSchool',
        topic: 'magic/#', // /^magic\/(.*)$/
        handler
      },
      {
        appId: 'Tradeshift.Magicians'
        handler
      }
    ]);
  } catch (e) {
    console.error(e);
  }
}
init();

// Simple message handler
function handler(appId, topic, payload) {
  console.log(`Message from app: ${appId}`);
  console.log(`Topic: ${topic}`);
  console.log(`Payload: ${payload}`);
}
```

### Receive message for specific topics

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Subscribe to specific topics
		client.subscribe([
			{
				topic: '+/spells', // /^[0-9a-zA-Z_- ]*\/spells$/
				handler
			},
			{
				topic: '#/set/#', // /^(.*)\/set\/(.*)$/
				handler
			},
			{
				topic: 'wand/carving',
				handler
			}
		]);
	} catch (e) {
		console.error(e);
	}
}
init();

// Simple message handler
function handler(appId, topic, payload) {
	console.log(`Message from app: ${appId}`);
	console.log(`Topic: ${topic}`);
	console.log(`Payload: ${payload}`);
}
```

### Send message to single app and handle response

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Exchange message with specific app
		const payloads = await client.exchange([
			{
				appId: 'Tradeshift.SpellCaster',
				topic: 'cast/magic-missile',
				payload: {
					level: 3
				}
			}
		]);

		// Handle response
		console.log(
			`Magic Missile dealt ${payloads[0].dmg} points of force damage.`
		);
	} catch (e) {
		console.error(e);
	}
}
init();
```

### Send message to multiple apps and handle response(s)

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Exchange message with multiple apps
		const payloads = await client.exchange([
			{
				appId: 'Tradeshift.Jazz',
				topic: 'music/play',
				payload: {
					smoothness: 8
				}
			},
			{
				appId: 'Tradeshift.Lights',
				topic: 'scene/set',
				payload: {
					relaxing: 42
				}
			}
		]);

		// Handle responses
		console.log(
			`Playing ${payloads[0].track} while setting ${
				payload[1].lights
			} lights to ${payload[1].color}`
		);
	} catch (e) {
		console.error(e);
	}
}
init();
```

### Load app and handle response

```js
import tsApp from '@tradeshift/tradeshift-app';

async function init() {
	try {
		// Connect to Server
		const client = await tsApp.connect();

		// Load app and wait for user selection or some other response
		const payload = await client.load({
			appId: 'Tradeshift.ShipSelector',
			topic: 'ship/select',
			payload: {
				minCapacity: 500,
				minSpeed: 30
			}
		});

		// Handle response
		console.log(
			`Selected ship: ${payload.name}, capacity: ${payload.capacity}, speed: ${
				payload.speed
			}`
		);
	} catch (e) {
		console.error(e);
	}
}
init();
```
