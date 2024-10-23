# Archway Quickstart

## Installing Dependencies
```shell
npm install @jackallabs/jackal.js
```

## Init

To get started building with Jackal, we first need to create a `StorageHandler`.

```typescript
import type { IClientSetup, IStorageHandler, ClientHandler } from '@jackallabs/jackal.js'

const chainId = 'jackal-1'
const mainnet = {
  chainId,
  endpoint: 'https://rpc.jackalprotocol.com',
    chainConfig: {
        chainId,
        chainName: 'Jackal Mainnet',
        rpc: 'https://rpc.jackalprotocol.com',
        rest: 'https://api.jackalprotocol.com',
        bip44: {
            coinType: 118
        },
        stakeCurrency: {
            coinDenom: 'JKL',
            coinMinimalDenom: 'ujkl',
            coinDecimals: 6
        },
        bech32Config: {
            bech32PrefixAccAddr: 'jkl',
            bech32PrefixAccPub: 'jklpub',
            bech32PrefixValAddr: 'jklvaloper',
            bech32PrefixValPub: 'jklvaloperpub',
            bech32PrefixConsAddr: 'jklvalcons',
            bech32PrefixConsPub: 'jklvalconspub'
        },
        currencies: [
            {
                coinDenom: 'JKL',
                coinMinimalDenom: 'ujkl',
                coinDecimals: 6
            }
        ],
        feeCurrencies: [
            {
                coinDenom: 'JKL',
                coinMinimalDenom: 'ujkl',
                coinDecimals: 6,
                gasPriceStep: {
                    low: 0.002,
                    average: 0.002,
                    high: 0.02
                }
            }
        ],
        features: []
    }
}

export const archwayChainId = 'archway-1'
export const archwayConfig = {
    chainId: archwayChainId,
    chainName: 'Archway',
    rpc: 'https://archway-rpc.polkachu.com',
    rest: 'https://archway-api.polkachu.com',
    bip44: {
        coinType: 118
    },
    stakeCurrency: {
        coinDenom: 'ARCH',
        coinMinimalDenom: 'aarch',
        coinDecimals: 18
    },
    bech32Config: {
        bech32PrefixAccAddr: 'arch',
        bech32PrefixAccPub: 'archpub',
        bech32PrefixValAddr: 'archvaloper',
        bech32PrefixValPub: 'archvaloperpub',
        bech32PrefixConsAddr: 'archvalcons',
        bech32PrefixConsPub: 'archvalconspub'
    },
    currencies: [
        {
            coinDenom: 'ARCH',
            coinMinimalDenom: 'aarch',
            coinDecimals: 18
        }
    ],
    feeCurrencies: [
        {
            coinDenom: 'ARCH',
            coinMinimalDenom: 'aarch',
            coinDecimals: 18,
            gasPriceStep: {
                low: 196000000000,
                average: 225400000000,
                high: 254800000000
            }
        }
    ],
    features: []
}

const setup: IClientSetup = {
    host: {
        chainId: archwayChainId,
        endpoint: 'https://archway-rpc.polkachu.com',
        chainConfig: archwayConfig,
    },
    endpoint: mainnet.endpoint,
    chainConfig: mainnet.chainConfig,
    chainId: mainnet.chainId,
    networks: ["archway", "jackal"],
    selectedWallet: 'keplr',
}



const myClient = await ClientHandler.connect(setup)
const storage: IStorageHandler = await myClient.createWasmStorageHandler()
storage.loadProviderPool() // load the available provider pool
```

## Purchasing Storage
Purchase storage requires an IBC send to make sure the account on Jackal has the correct amount of tokens, in this code snippet we IBC send over the JKL difference needed.

```typescript
async function buyStorage () {
    const gb = SOME_GB_VALUE
    const days = SOME_DAY_COUNT

    const usd = getUSDPrice() // get USD price of storage plan, function body not included here.
    const price = Math.floor(usd / PRICE_OF_JACKAL_TOKEN_IN_USD * 1000000 * 1.15)

    const coin = await myClient.getJklBalance()
    const hostBal = await myClient.getHostNetworkBalance(myClient.getHostAddress(), 'ibc/926432AE1C5FA4F857B36D970BE7774C7472079506820B857B75C5DE041DD7A3')

    let bal = coin.amount
        
    if (bal < price) {
        if (hostBal.amount < price - bal) {
          throw ('You do not have enough JKL on Archway')
        }
        const c: Coin = {
          amount: (price - bal).toString(),
          denom: 'ibc/926432AE1C5FA4F857B36D970BE7774C7472079506820B857B75C5DE041DD7A3'
        }
        await myClient.ibcSend(myClient.getICAJackalAddress(), c, 'channel-14')
    }

    const c = await myClient.getJklBalance()
    bal = c.amount
    while (bal < price) {
        await new Promise(r => setTimeout(r, 1000))
        const c = await myClient.getJklBalance()
        bal = c.amount
    }
      
    await storage.purchaseStoragePlan({
        gb,
        days,
        broadcastOptions: {
          monitorTimeout: 5 * 60 // 5 minute timeout since we're using IBC
        }
    })
}
```

## Continue...
Continue normally from here with [Jackal.js Quickstart](devs/jjs-quickstart.md).