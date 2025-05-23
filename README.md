# Corestore

### [See the full API docs at docs.space.bsv.direct](https://docs.space.bsv.direct/helpers/corestore)

Corestore is a Spacecore factory that makes it easier to manage large collections of named Spacecores.

Corestore provides:
1. __Key Derivation__ - All writable Spacecore keys are derived from a single master key and a user-provided name.
2. __Session Handling__ - If a single Spacecore is loaded multiple times through the `get` method, the underlying resources will only be opened once (using Spacecore 10's new session feature). Once all sessions are closed, the resources will be released.
3. __Storage Management__ - Spacecores can be stored in any random-access-storage instance, where they will be keyed by their discovery keys.
4. __Namespacing__ - You can share a single Corestore instance between multiple applications or components without worrying about naming collisions by creating "namespaces" (e.g. `corestore.namespace('my-app').get({ name: 'main' })`)

### Installation
`npm install corestore`

### Usage
A corestore instance can be constructed with a random-access-storage module, a function that returns a random-access-storage module given a path, or a string. If a string is specified, it will be assumed to be a path to a local storage directory:
```js
const Corestore = require('spacecorestore')

const store = new Corestore('./my-storage')
const core1 = store.get({ name: 'core-1' })
const core2 = store.get({ name: 'core-2' })
```

### API
#### `const store = new Corestore(storage)`
Create a new Corestore instance.

`storage` can be either a random-access-storage module, a string, or a function that takes a path and returns an random-access-storage instance.

#### `const core = store.get(key | { name: 'a-name', ...spacecoreOpts})`
Loads a Spacecore, either by name (if the `name` option is provided), or from the provided key (if the first argument is a Buffer or String with hex/z32 key, or if the `key` options is set).

If that Spacecore has previously been loaded, subsequent calls to `get` will return a new Spacecore session on the existing core.

All other options besides `name` and `key` will be forwarded to the Spacecore constructor.

#### `const stream = store.replicate(optsOrStream)`
Creates a replication stream that's capable of replicating all Spacecores that are managed by the Corestore, assuming the remote peer has the correct capabilities.

`opts` will be forwarded to Spacecore's `replicate` function.

Corestore replicates in an "all-to-all" fashion, meaning that when replication begins, it will attempt to replicate every Spacecore that's currently loaded and in memory. These attempts will fail if the remote side doesn't have a Spacecore's capability -- Corestore replication does not exchange Spacecore keys.

If the remote side dynamically adds a new Spacecore to the replication stream, Corestore will load and replicate that core if possible.

Using [Spaceswarm](https://github.com/samooth/spaceswarm) you can easily replicate corestores

``` js
const swarm = new Spaceswarm()

// join the relevant topic
swarm.join(...)

// simply pass the connection stream to corestore
swarm.on('connection', (connection) => store.replicate(connection))
```

#### `const storeB = storeA.session()`
Create a new Corestore session. Closing a session will close all cores made from this session.

#### `const store = store.namespace(name)`
Create a new namespaced Corestore session. Namespacing is useful if you're going to be sharing a single Corestore instance between many applications or components, as it prevents name collisions.

Namespaces can be chained:
```js
const ns1 = store.namespace('a')
const ns2 = ns1.namespace('b')
const core1 = ns1.get({ name: 'main' }) // These will load different Spacecores
const core2 = ns2.get({ name: 'main' })
```

#### `await store.close()`
Fully close this Corestore instance.

### License
MIT
