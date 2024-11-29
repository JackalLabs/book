# Developer Quickstart

## Installing Dependencies
```shell
npm install @jackallabs/jackal.js
```

## Init
To get started building with Jackal, we first need to create a `StorageHandler`.

```typescript
import { ClientHandler } from '@jackallabs/jackal.js'
import type { IClientSetup, IStorageHandler } from '@jackallabs/jackal.js'

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

const setup: IClientSetup = {
    selectedWallet: 'keplr',
    ...mainnet
}

const myClient = await ClientHandler.connect(setup)
const storage: IStorageHandler = await myClient.createStorageHandler()
storage.loadProviderPool() // load the available provider pool
```

## Purchasing Storage
Purchase storage if needed.

```typescript
// 1 TB for 1 year
const options = {
  gb: 1000,
  days: 365
}
await storage.purchaseStoragePlan(options)
```

## Upload Files
Use your storage account to upload files.

```typescript
// unlock full feature set
await storage.upgradeSigner()

// if first time using account, initialize
await storage.initStorage()

// load root directory if not already loaded
await storage.loadDirectory('Home')

// upload encrypted file
/* get your file into the browser */
const myFiles = [/* Files */]
await storage.queuePrivate(myFiles)
await storage.processAllQueues()

// upload public (unencrypted) file
/* get your file into the browser */
const myPublicFiles = [/* Files */]
await storage.queuePublic(myPublicFiles)
await storage.processAllQueues()
```

## Downloading Files
Download your file.
```typescript
// create a tracker to monitor download progress
const tracker = { progress: 0, chunks: [] }
const myFileName = 'myFileName.txt'

// Home is the default root folder for all Jackal.js accounts
const myFile = await storage.downloadFile(`Home/${myFileName}`, tracker)

// do something with myFile
```

## More!
Check out a deep dive on Jackal.js [here](https://jackallabs.github.io/jackal.js/).
