# Web5 JS SDK
Making developing with Web5 components at least 5 times easier to work with.

⚠️ WEB5 JS SDK IS IN A PRE-BETA STATE ⚠️

## Introduction

Web5 consists of the following components:
- Decentralized Identifiers
- Verifiable Credentials
- DWeb Node personal datastores

The SDK sets out to gather the most oft used functionality from all three of these 
pillar technologies to provide a simple library that is as close to effortless as 
possible. 

## Docs

[API Reference](https://tbd54566975.github.io/web5-js/)

### **`new Web5([options])`**

Creates an isolated API object for doing Web5 things, split into three main top-level objects: `did`, `dwn`, and `vc`.

#### **`options`**

When creating a `Web5` instance the following options can be provided:

##### **`dwn`**

This object contains options related to `web5.dwn`.

- **`node`**  - *`Dwn`*: A customizable `Dwn` instance to use instead of a default one created by `web5.dwn`. This can be used to customize the storage location/structure/etc. of the `Dwn`, how DIDs are resolved, etc..

### **`web5.did.create(method, options)`**

The `create` method under the `did` scope enables generation of DIDs for a supported set of DID Methods. The output is method-specific, and handles things like key generation and assembly of DID Documents that can be published to decentralized DID networks.

#### **Example**

```javascript
const myDid = await web5.did.create('ion');
```

### **`web5.did.manager.set(did, parameters)`**

The package provides a DID manager mechanism that handles interactions with a DID that is being 'managed' by the local code (be it in a web page, agent app, or elsewhere). By adding a DID to the manager, the library ensures that all interactions with that DID, including things like signing and encryption, are handled automatically.

#### **Example**

Assuming the `myDid` instance is present from the example above, you would pass the following values to add it to the DID manager:

```javascript
await web5.did.manager.set(myDid.id, {
    connected: true,
    endpoint: 'app://dwn', //this points to the user's local DWN
    keys: {
        '#dwn': {
          keyPair: myDid.keys.find(key => key.id === 'dwn').keyPair,
        }
    }
});
```

### **`web5.dwn.records.query(target, request)`**

Method for querying the DWeb Node of a provided `target` DID.

#### **`request`**

The query `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.

#### **Example** 

```javascript
const web5 = new Web5();
const response = await web5.dwn.records.query('did:example:bob', {
  author: 'did:example:alice',
  message: {
    filter: {
      schema: 'https://schema.org/Playlist',
      dataFormat: 'application/json'
    }
  }
});

console.log(response.entries) // logs array of Record class instances
```

### **`web5.dwn.records.create(target, request)`**

Method for writing a record to the DWeb Node of a provided `target` DID.

#### **`request`**

The write `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
- **`data`**  - *`blob | stream | file`*: The data object of the bytes to be sent.

#### **Example** 

```javascript
const web5 = new Web5();
const { record } = await web5.dwn.records.create('did:example:alice', {
  author: 'did:example:alice',
  data: 'Hello World!',
  message: {
    dataFormat: 'text/plain'
  }
});

console.log(record.data.text()) // logs "Hello World!" in console
```

### **`Record` response instances**

Every modifying method (`create`, `write`, etc.) and query entries return an instance of `Record` class, which is a representation of the Record(s) involved. `Record` class instances have the following properties:

- **`recordId`**  - *`string`*: The unique identifier based on the record entry's composition. Note: all entries across all records are deterministically unique.
- **`descriptor`**  - *`object`*: The descriptor object for the constructed DWeb Node message.
- **`data`**  - *`object`*: a object with the following convenience methods that read out the data of the record entry in the following formats:
  - **`text`**  - *`function`*: produces a textual representation of the data.
  - **`json`**  - *`function`*: if the value is JSON data, this method will return a parsed JSON object.
  - **`stream`**  - *`function`*: returns the raw stream of bytes for the data.

### **`web5.dwn.records.write(target, request)`**

Method for writing a record to the DWeb Node of a provided `target` DID.

#### **`request`**

The write `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
- **`data`**  - *`blob | stream | file`*: The data object of the bytes to be sent.

#### **Example** 

For this example we assume we have created the record and are using the output `Record` instance from above to overwrite the data of the record with a subsequent `write`:

```javascript
const web5 = new Web5();
const { record } = await web5.dwn.records.write('did:example:alice', {
  author: 'did:example:alice',
  data: 'Hello again!',
  message: {
    recordId: record.recordId,
    dataFormat: 'text/plain'
  }
});

console.log(record.data.text()) // prints "Hello again!" in console
```

### **`web5.dwn.records.read(target, request)`**

Method for writing a record to the DWeb Node of a provided `target` DID.

#### **`request`**

The write `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
  - **`recordId`**  - *`string`*: the required record ID string that identifies the record data you are fetching.

#### **Example** 

```javascript
const web5 = new Web5();
const response = await web5.dwn.records.read('did:example:alice', {
  author: 'did:example:alice',
  message: {
    recordId: 'bfw35evr6e54c4cqa4c589h4cq3v7w4nc534c9w7h5'
  }
});
```

### **`web5.dwn.records.delete(target, request)`**

Method for deleting a record in the DWeb Node of a provided `target` DID.

#### **`request`**

The delete `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
    - **`recordId`**  - *`string`*: the required record ID string that identifies the record being deleted.

#### **Example** 

```javascript
const web5 = new Web5();
const response = await web5.dwn.records.delete('did:example:alice', {
  author: 'did:example:alice',
  message: {
    recordId: 'bfw35evr6e54c4cqa4c589h4cq3v7w4nc534c9w7h5'
  }
});
```

### **`web5.dwn.protocols.query(target, request)`**

Method for querying the protocols that a `target` DID has added configurations for in their DWeb Node.

#### **`request`**

The query `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
  - **`filter`**  - *`object`*: an object that defines a set of properties to filter results.
      - **`protocol`**  - *`URI string`*: a URI that represents the protocol being queried for.

#### **Example** 

```javascript
const web5 = new Web5();
const response = await web5.dwn.protocols.query('did:example:alice', {
  author: 'did:example:alice',
  message: {
    filter: {
      protocol: "https://decentralized-music.org/protocol",
    }
  }
});
```

### **`web5.dwn.protocols.configure(target, request)`**

Method for deleting a record in the DWeb Node of a provided `target` DID.

#### **`request`**

The configuration `request` must contain the following:

- **`author`**  - *`string`*: The decentralized identifier of the DID signing the query. This may be the same as the `target` parameter if the target and the signer of the query are the same entity, which is common for an app querying the DWeb Node of its own user.
- **`message`**  - *`object`*: The properties of the DWeb Node Message Descriptor that will be used to construct a valid DWeb Node message.
    - **`protocol`**  - *`URI string`*: a URI that represents the protocol being configured via the `definition` object.
    - **`definition`**  - *`object`*: an object that defines the ruleset that will be applied to the records and activities under the protocol.
        - **`labels`**  - *`object`*: an object that defines the composition of records that will be used in the `records` tree below.
        - **`records`**  - *`object`*: a recursive object that defines the structure of an app, including data relationships and constraints on which entities can perform various activities.

#### **Example** 

```javascript
const web5 = new Web5();
const response = await web5.dwn.protocols.configure('did:example:alice', {
  author: 'did:example:alice',
  message: {
    protocol: "https://decentralized-music.org/protocol",
    definition: {
      "labels": {
        "playlist": {
          "schema": "https://decentralized-music.org/protocol/playlist",
          "dataFormat": [ "application/json" ]
        },
        "track": {
          "schema": "https://decentralized-music.org/protocol/track",
          "dataFormat": [ "application/json" ]
        },
        "audio": {
          "schema": "https://decentralized-music.org/protocol/audio",
          "dataFormat": [ "audio/aac", "audio/mp4" ]
        }
      },
      "records": {
        "playlist": {},
        "track": {
          "records": {
            "audio": {}
          }
        }
      }
    }
  }
});
```

### **`web5.connect([options])`**

Enables an app to request connection to a user's local identity app, or generate an in-app DID to represent the user (e.g. if the user does not have an identity app).

> NOTE: This method **_MUST_** be invoked within the scope of a 'trusted user action', which is something the browser/OS decides. For browsers this is generally some direct user intent, like clicking a link or button.

#### **`options`**

The `connect` method optionally accepts an object with the following properties:

##### **`storage`**

Used by `connect` to store connection data/keys/etc. for reuse when calling `connect` again (e.g. during another session).

If provided, `storage` must be an object that has the same methods as [`Storage`](/TBD54566975/web5-js/tree/main/src/storage/Storage.js).

If not provided, an instance of [`LocalStorage`](/TBD54566975/web5-js/tree/main/src/storage/LocalStorage.js) is used instead.

##### **`connectionLocation`**

Controls where in `storage` the connection data is stored.

Defaults to `'web5-connection'`.

##### **`keysLocation`**

Controls where in `storage` the connection keys are stored.

Defaults to `'web5-keys'`.

##### **`silent`**

Controls whether to prompt the user to start a new connection if a connection has not already been established.

Defaults to `false`.

#### **Events**

After calling `connect`, at least one of the following events will be dispatched on the `Web5` instance:

##### **`'open'`**

A `'open'` event is dispatched if a local identity app is found, containing a verification pin and port number used for the connection.

- **`event.detail.pin`**  - *`number`*: 1-4 digit numerical code that must be displayed to the user to ensure they are connecting to the desired local identity app.
- **`event.detail.port`** - *`number`*: The port number used by the the local identity app to listen for connections.

##### **`'connection'`**

A `'connection'` event is dispatched if the user accepts the connection, containing the DID the user selected in the local identity app.

- **`event.details.did`**  - *`string`*: The DID selected by the user in the local identity app.

##### **`'close'`**

A `'close'` event is dispatched if the user refuses to accept or respond to the connection attempt for any reason.

##### **`'error'`**

An `'error'` event is dispached if anything goes wrong (e.g. `Web5` was unable to find any local identity apps).

#### **Example** 

```javascript
document.querySelector('#connect_button').addEventListener('click', async event => {

  event.preventDefault();

  const web5 = new Web5();

  web5.addEventListener('open', (event) => {
    const { pin } = event.detail
    document.querySelector('#pin_code_text').textContent = pin;
  });

  web5.addEventListener('connection', (event) => {
    const { did } = event.detail
    alert('Connection succeeded! Connected to DID: ' + did);
  });

  web5.addEventListener('close', (event) => {
    alert('Connection was denied');
  });

  web5.addEventListener('error', (event) => {
    console.error(event);
  });

  web5.connect();
});
```

## Project Resources

| Resource                                   | Description                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------------ |
| [CODEOWNERS](./CODEOWNERS)                 | Outlines the project lead(s)                                                   |
| [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) | Expected behavior for project contributors, promoting a welcoming environment |
| [CONTRIBUTING.md](./CONTRIBUTING.md)       | Developer guide to build, test, run, access CI, chat, discuss, file issues     |
| [GOVERNANCE.md](./GOVERNANCE.md)           | Project governance                                                             |
| [LICENSE](./LICENSE)                       | Apache License, Version 2.0                                                    |