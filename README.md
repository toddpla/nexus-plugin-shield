# nexus-plugin-shield

Unlock the power of [graphql-shield](https://github.com/maticzav/graphql-shield) in your nexus app

## Installation

```
npm install nexus-plugin-shield
```

## Known limitations

- fragments not supported

## Example Usage

### Setup

```typescript
// app.ts

import { use } from 'nexus'
import { shield, rule, deny, not, and, or } from 'nexus-plugin-shield'

const isAuthenticated = rule({ cache: 'contextual' })(
  async (parent, args, ctx: NexusContext, info) => {
    return ctx.user !== null
  }
)

const isAdmin = rule({ cache: 'contextual' })(
  async (parent, args, ctx: NexusContext, info) => {
    return ctx.user.role === 'admin'
  }
)

const isEditor = rule({ cache: 'contextual' })(
  async (parent, args, ctx: NexusContext, info) => {
    return ctx.user.role === 'editor'
  }
)

const permissions = shield({
  rules: {
    Query: {
      frontPage: not(isAuthenticated),
      fruits: and(isAuthenticated, or(isAdmin, isEditor)),
      customers: and(isAuthenticated, isAdmin),
    },
    Mutations: {
      addFruitToBasket: isAuthenticated,
    },
    Fruit: isAuthenticated,
    Customer: isAdmin,
  },
  options: {
    fallbackRule: deny,
  },
})

use(permissions)
```

### `shield({rules: rules, options: options})`

#### `rules`

A rule map must match your schema definition.

[graphql-shield documentation](https://github.com/maticzav/graphql-shield#shieldrules-options)

#### `options`

[graphql-shield documentation](https://github.com/maticzav/graphql-shield#options)

### Context type

Nexus provide a global `NexusContext` interface you can use in your rules:

```typescript
rule()(async (parent, args, context: NexusContext, info) => {
  // logic
})
```
