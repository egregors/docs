# Core Concepts of Logux

Logux synchronizes action log between peer-to-peer nodes.

You can use Logux to connect clients with a server, clients with clients,
servers with servers or for mesh-networks. There is no big difference between
clients and server in Logux. So we will call them **nodes**.

```haskell
[Client 1] ⇆ [Client 2] ⇆ [Server A] ⇆ [Server B]
```

However, we created Logux for standard applications with multiple
web and mobile clients and several servers. Next chapter read will show
how core concepts work in the standard case.

Each node has **action** log (list of operations). When you want to change
the application state, you add an action to the log. An action is a JSON object
that describes what happened. Every action must have `type` property.

```js
{ type: 'user/rename' userId: 386, name: 'New name' }
```

Action **log** is append-only. You can only add actions, but can’t change
or delete them. However, you can compress log by cleaning old actions,
which are not actual anymore.

```js
app.log.add(action)
```

Each action in the log has own **meta** with:

* `meta.id`: unique action ID.
* `meta.time`: action creation time. It uses local client’s time
  and could be different on different clients.
* `meta.added`:
* `meta.reasons`: an array of the strings. Each string is a code of some
  “reason” why action is still actual. When application will remove all reasons,
  Logux will clean the action from the log.
* Applications could add own data to **meta**.

```js
[action, {
  // Core meta
  id: '1553821137583 388:mgxhClZT:mAKgAtBF 0',
  time: 1553821137582,
  added: 56,
  reasons: ['user:388:lastName'],
  // Custom meta
  channels: ['users/388']
}]
```

Differences between action and meta:

* **Actions are immutable.** But you can change meta
  (except `meta.id`, `meta.added` and `meta.time`).
* Logux synchronizes actions, but **only few meta keys will be synchronized**.
  Each Logux implementation decides what meta keys it will synchronize.

You can use any method to connect Logux nodes. Logux use
**Web Socket** only as default way.

During the synchronization Logux **guarantee**:

* Each action will be synchronized **only once**.
* Actions will have **the same order** on each node.

Logux us based on the **Optimistic UI** idea. When a node creates action,
it applies it immediately to own application state. In the background,
Logux will synchronize this new action with other nodes.