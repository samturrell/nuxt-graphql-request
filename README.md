# 📡 GraphQL Request Module

[![circleci][circleci-src]][circleci-href]
[![npm version][npm-version-src]][npm-version-href]
[![Dependencies][david-dm-src]][david-dm-href]
[![npm downloads][npm-downloads-src]][npm-downloads-href]
[![code style: prettier](https://img.shields.io/badge/code_style-prettier-1a2b34.svg?style=flat-square)](https://prettier.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg?style=flat-square)](https://opensource.org/licenses/MIT)

> Easy Minimal <a href="https://github.com/prisma-labs/graphql-request">GraphQL</a> client integration with Nuxt.js.

## Features

- Most **simple and lightweight** GraphQL client.
- Promise-based API (works with `async` / `await`).
- Typescript support.
- AST support.
- GraphQL Loader support.

[📖 **Release Notes**](./CHANGELOG.md)

## Setup

Install with yarn:

```bash
yarn add nuxt-graphql-request graphql --dev
```

Install with npm:

```bash
npm install nuxt-graphql-request graphql --save-dev
```

**nuxt.config.js**

```ts
module.exports = {
  buildModules: ['nuxt-graphql-request'],

  graphql: {
    /**
     * Your GraphQL endpoint
     */
    endpoint: 'https://swapi-graphql.netlify.com/.netlify/functions/index',

    /**
     * Options
     * See: https://github.com/prisma-labs/graphql-request#passing-more-options-to-fetch
     */
    options: {},

    /**
     * Optional
     * default: true (this includes cross-fetch/polyfill before creating the graphql client)
     */
    useFetchPolyfill: true,

    /**
     * Optional
     * default: false (this includes graphql-tag for node_modules folder)
     */
    includeNodeModules: true,
  },
};
```

### Runtime Config

If you need to supply your endpoint at runtime, rather than build time, you can use the [Runtime Config](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-runtime-config) to provide this value:

**nuxt.config.js**

```ts
module.exports = {
  publicRuntimeConfig: {
    GRAPHQL_ENDPOINT: '<your endpoint>',
  },
};
```

## Usage

### Component

**`asyncData`**

```ts
import { gql } from 'graphql-request';

async asyncData({ $graphql, params }) {
  const query = gql`
    query planets {
      allPlanets {
        planets {
          id
          name
        }
      }
    }
  `;

  const planets = await $graphql.request(query);
  return { planets };
}
```

**`methods`/`created`/`mounted`/etc**

```ts
import { gql } from 'graphql-request';

methods: {
  async fetchSomething() {
    const query = gql`
      query planets {
        allPlanets {
          planets {
            id
            name
          }
        }
      }
    `;

    const planets = await $graphql.request(query);
    this.$set(this, 'planets', planets);
  }
}
```

### Store actions (including `nuxtServerInit`)

```ts
import { gql } from 'graphql-request';

// In store
{
  actions: {
    async fetchAllPlanets ({ commit }) {
      const query = gql`
        query planets {
          allPlanets {
            planets {
              id
              name
            }
          }
        }
      `;

      const planets = await this.$graphql.request(query);
      commit('SET_PLANETS', planets)
    }
  }
}
```

### GraphQL Request Client

> <a href="https://github.com/prisma-labs/graphql-request#examples">Examples</a> from the official graphql-request library.

#### Authentication via HTTP header

In nuxt.config.ts

```ts
// nuxt.config.ts

module.exports = {
  graphql: {
    endpoint: 'https://swapi-graphql.netlify.com/.netlify/functions/index',
    options: {
      headers: {
        authorization: 'Bearer MY_TOKEN',
      },
    },
  },
};
```

Or using setHeaders / setHeader:

```ts
this.$graphql.setHeaders({ authorization: 'Bearer MY_TOKEN' });

this.$graphql.setHeader('authorization', 'Bearer MY_TOKEN');
```

#### Passing more options to fetch

In nuxt.config.ts:

```ts
// nuxt.config.ts

module.exports = {
  graphql: {
    endpoint: 'https://swapi-graphql.netlify.com/.netlify/functions/index',
    options: {
      credentials: 'include',
      mode: 'cors',
    },
  },
};
```

Or using setHeaders / setHeader:

```ts
this.$graphql.setHeaders({
  credentials: 'include',
  mode: 'cors',
});

this.$graphql.setHeader('credentials', 'include');
this.$graphql.setHeader('mode', 'cors');
```

#### Using variables

```ts
import { gql } from 'graphql-request';

const query = gql`
  query planets($first: Int) {
    allPlanets(first: $first) {
      planets {
        id
        name
      }
    }
  }
`;

const variables = { first: 10 };

const planets = await $graphql.request(query, variables);
```

#### Receiving a raw response

The `request` method will return the `data` or `errors` key from the response. If you need to access the `extensions` key you can use the `rawRequest` method:

```ts
import { gql } from 'graphql-request';

const query = gql`
  query planets($first: Int) {
    allPlanets(first: $first) {
      planets {
        id
        name
      }
    }
  }
`;

const variables = { first: 10 };

const { data, errors, extensions, headers, status } = await $graphql.rawRequest(
  endpoint,
  query,
  variables
);
console.log(JSON.stringify({ data, errors, extensions, headers, status }, undefined, 2));
```

## [FAQ](https://github.com/prisma-labs/graphql-request/blob/master/README.md#faq)

#### Why use `nuxt-graphql-request` over `@nuxtjs/apollo`?

Don't get me wrong, Apollo Client is great and well maintained by the vue / nuxt community, I used Apollo Client for 18months before switching to graphql-request.

However, as I am obsessed with performances, Apollo Client doesn't work for me at all:

- I don't need another state management as the Vue ecosystem is enough (Vuex & Persisted data).
- I don't need an extra ~120kb parsed in my app for fetching my data.
- I don't need subscriptions as I use pusher.com, there are also alternatives for a WS client: [http://github.com/lunchboxer/graphql-subscriptions-client](http://github.com/lunchboxer/graphql-subscriptions-client)

#### Why do I have to install `graphql`?

`graphql-request` uses a TypeScript type from the `graphql` package such that if you are using TypeScript to build your project and you are using `graphql-request` but don't have `graphql` installed TypeScript build will fail. Details [here](https://github.com/prisma-labs/graphql-request/pull/183#discussion_r464453076). If you are a JS user then you do not technically need to install `graphql`. However, if you use an IDE that picks up TS types even for JS (like VSCode) then it's still in your interest to install `graphql` so that you can benefit from enhanced type safety during development.

#### Do I need to wrap my GraphQL documents inside the `gql` template exported by `graphql-request`?

No. It is there for convenience so that you can get the tooling support like prettier formatting and IDE syntax highlighting. You can use `gql` from `graphql-tag` if you need it for some reason too.

### What's the difference between `graphql-request`, Apollo and Relay?

`graphql-request` is the most minimal and simplest to use GraphQL client. It's perfect for small scripts or simple apps.

Compared to GraphQL clients like Apollo or Relay, `graphql-request` doesn't have a built-in cache and has no integrations for frontend frameworks. The goal is to keep the package and API as minimal as possible.

### Does nuxt-graphql-request support mutations?

Sure, you can perform any GraphQL queries & mutations as before 👍

## Development

1. Clone this repository
2. Install dependencies using `yarn install` or `npm install`
3. Start development server using `yarn dev` or `npm run dev`

## Roadmap

- [ ] Support multiple clients
- [ ] Support [WebSocket client](https://github.com/lunchboxer/graphql-subscriptions-client)?
- [ ] Expose `request` function for running GraphQL queries/mutations as a static function.

## 📑 License

[MIT License](./LICENSE)

<!-- Badges -->

[circleci-src]: https://circleci.com/gh/Gomah/nuxt-graphql-request.svg?style=shield
[circleci-href]: https://circleci.com/gh/Gomah/nuxt-graphql-request
[npm-version-src]: https://img.shields.io/npm/dt/nuxt-graphql-request.svg?style=flat-square
[npm-version-href]: https://npmjs.com/package/nuxt-graphql-request
[npm-downloads-src]: https://img.shields.io/npm/v/nuxt-graphql-request/latest.svg?style=flat-square
[npm-downloads-href]: https://npmjs.com/package/nuxt-graphql-request
[david-dm-src]: https://david-dm.org/gomah/nuxt-graphql-request/status.svg?style=flat-square
[david-dm-href]: https://david-dm.org/gomah/nuxt-graphql-request
