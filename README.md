[![npm version](https://img.shields.io/npm/v/web3-react/unstable.svg)](https://www.npmjs.com/package/web3-react/v/unstable)
[![Build Status](https://travis-ci.org/NoahZinsmeister/web3-react.svg?branch=unstable)](https://travis-ci.org/NoahZinsmeister/web3-react)
[![Coverage Status](https://coveralls.io/repos/github/NoahZinsmeister/web3-react/badge.svg?branch=unstable)](https://coveralls.io/github/NoahZinsmeister/web3-react?branch=unstable)

![Example GIF](./_assets/example.gif)

## Resources

- Documentation for `web3-react` is [available on Gitbook](https://noahzinsmeister.gitbook.io/web3-react/v/unstable/).

- A live demo of `web3-react` is [available on CodeSandbox](https://codesandbox.io/s/r7wymqkzwn).

## Introduction

`web3-react` is a drop-in solution for building Ethereum dApps in React. Its marquee features are:

- Complete support for commonly used web3 providers including [MetaMask](https://metamask.io/)/[Trust](https://trustwallet.com/)/[Tokenary](https://tokenary.io/), [Infura](https://infura.io/), [Trezor](https://trezor.io/)/[Ledger](https://www.ledger.com/), [Fortmatic](https://fortmatic.com/)/[Portis](https://www.portis.io/), [WalletConnect](https://walletconnect.org/), and more.

- A robust framework which exposes an instantiated [ethers.js](https://github.com/ethers-io/ethers.js/) or [web3.js](https://web3js.readthedocs.io/en/1.0/) instance, the current account and network id, and a variety of helper functions to your dApp via a [React Context](https://reactjs.org/docs/context.html).

- The ability to write custom, fully featured _Connectors_ to manage every aspect of your dApp's connectivity with the Ethereum blockchain and user account(s).

## Quickstart

If you want to cut straight to the chase, check out the CodeSandbox demo!

[![Edit web3-react](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/r7wymqkzwn)

### 1. Install

Ensure you're using the latest `react` and `react-dom` versions (or anything `^18`):

```bash
yarn add react@latest react-dom@latest
```

Next, install either [ethers.js](https://github.com/ethers-io/ethers.js/) or [web3.js](https://web3js.readthedocs.io/en/1.0/), depending on your preference:

```bash
yarn add ethers
```

```bash
yarn add web3
```

Finally you're ready to use `web3-react`:

```bash
yarn add web3-react@unstable
```

### 2. Setup Connectors

Now, you'll need to decide how you want users to interact with your dApp. This is almost always with some combination of MetaMask, Infura, Trezor/Ledger, WalletConnect, etc. For more details on each of these options, see [Connectors.md](./Connectors.md).

```javascript
import { Connectors } from 'web3-react'
const { InjectedConnector, NetworkOnlyConnector } = Connectors

const metaMask = new InjectedConnector({ supportedNetworks: [1, 5] })

const infura = new NetworkOnlyConnector({
  providerURL: 'https://mainnet.infura.io/v3/...'
})

const connectors = { metaMask, infura }
```

### 3. Setup `Web3Provider`

The next step is to setup a `Web3Provider` at the root of your dApp. This ensures that children components are able to take advantage of the `web3-react` context.

```javascript
import React from 'react'
import Web3Provider from 'web3-react'

export default function App () {
  return (
    <Web3Provider
      connectors={...}
      libraryName={'ethers.js'|'web3.js'}
    >
      ...
    </Web3Provider>
  )
}
```

The `Web3Provider` takes 2 props:

1. `connectors: any` (required): An object mapping arbitrary `string` connector names to Connector objects (see [the previous section](#2-setup-connectors) for more detail).

2. `libraryName: string` (optional): `ethers.js` or `web3.js`, depending on which library you wish to use in your dApp. Passing `null` will expose the low-level provider object.

### 4. Activate

Now, you need to decide how/when you would like to activate your Connectors. For all options, please see [the manager functions](#manager-functions) section. The example code below attempts to automatically activate MetaMask, and falls back to infura.

```javascript
import React, { useEffect } from 'react'
import { useWeb3Context } from 'web3-react'

// This component must be a child of <App> to have access to the appropriate context
export default function MyComponent () {
  const context = useWeb3Context()

  useEffect(() => {
    context.setFirstValidConnector(['metaMask', 'infura'])
  }, [])

  if (!context.active && !context.error) {
    // loading
    return ...
  } else if (context.error) {
    //error
    return ...
  } else {
    // success
    return ...
  }
}
```

### 5. Using `web3-react`

Finally, you're ready to use `web3-react`!

#### _Recommended_ - Hooks

The easiest way to use `web3-react` is with the `useWeb3Context` hook.

```javascript
import React from 'react'
import { useWeb3Context } from 'web3-react'

function MyComponent() {
  const context = useWeb3Context()

  return <p>{context.account}</p>
}
```

#### _Conditionally Recommended_ - Render Props

To use `web3-react` with render props, wrap Components in a `Web3Consumer`.

```javascript
import React from 'react'
import { Web3Consumer } from 'web3-react'

function MyComponent() {
  return <Web3Consumer>{context => <p>{account}</p>}</Web3Consumer>
}
```

The component takes 2 props:

1. `recreateOnNetworkChange: boolean` (optional, default `true`). A flag that controls whether child components are completely re-initialized upon network changes.

2. `recreateOnAccountChange: boolean` (optional, default `true`). A flag that controls whether child components are completely re-initialized upon account changes.

#### _Not Recommended_ - HOCs

If you must, you can use `web3-react` with an [HOC](https://reactjs.org/docs/context.html#consuming-context-with-a-hoc).

```javascript
import React from 'react'
import { withWeb3 } from 'web3-react'

function MyComponent({ web3 }) {
  const { account } = web3

  return <p>{account}</p>
}

export default withWeb3(MyComponent)
```

`withWeb3` takes an optional second argument, an object that can set the flags defined above in the [render props section](conditionally-recommended---render-props).

Note: The Component which includes your `Web3Provider` Component **cannot** be wrapped with `withWeb3`.

## Context

Regardless of how you access the `web3-react` context, it will look like:

```typescript
{
  active: boolean
  connectorName?: string
  connector?: any
  library?: any
  networkId?: number
  account?: string | null
  error: Error | null

  setConnector: (connectorName: string, suppressAndThrowErrors?: boolean) => Promise<void>
  setFirstValidConnector: (connectorNames: string[], suppressAndThrowErrors?: boolean) => Promise<void>
  unsetConnector: () => void
  setError: (error: Error, connectorName?: string) => void
}
```

### Variables

- `active`: A flag indicating whether `web3-react` currently has an connector set.
- `connectorName`: The name of the currently active connector.
- `connector`: The instance of the currently active connector.
- `library`: An instantiated [ethers.js](https://github.com/ethers-io/ethers.js/) or [web3.js](https://web3js.readthedocs.io/en/1.0/) instance.
- `networkId`: The current active network ID.
- `account`: The current active account if one exists.
- `error`: The current active error if one exists.

### Manager Functions

- `setConnector(connectorName: string, suppressAndThrowErrors?: boolean)`: Activates a connector by name. The second argument is a flag (`false` by default) that controls whether errors, instead of bubbling up to `context.error`, are instead thrown by this function.
- `setFirstValidConnector(connectorNames: string[], suppressAndThrowErrors: boolean = false)`: Tries to activate each connector in turn by name. The second argument is a flag (`false` by default) that controls whether errors, instead of bubbling up to `context.error`, are instead thrown by this function.
- `unsetConnector()`: Unsets the currently active connector.
- `setError(error: Error, connectorName?: string)`: Sets `context.error`, with an optional connector name.

## Implementations

Projects using `web3-react` include:

- https://hydroblockchain.github.io/snowflake-dashboard/
- https://conlan.github.io/compound-liquidator/
- https://uniswap.info

Open a PR to add your project to the list! If you're interested in contributing, check out [Contributing-Guidelines.md](./docs/Contributing-Guidelines.md).

## Notes

Prior art for `web3-react` includes:

- A pure Javascript implementation with some of the same goals: [web3-webpacked](https://github.com/NoahZinsmeister/web3-webpacked).

- A non-Hooks port of [web3-webpacked](https://github.com/NoahZinsmeister/web3-webpacked) to React that had some problems: [web3-webpacked-react](https://github.com/NoahZinsmeister/web3-webpacked-react).

- A React library with some of the same goals but that uses the deprecated React Context API and does not use hooks: [react-web3](https://github.com/coopermaruyama/react-web3).
