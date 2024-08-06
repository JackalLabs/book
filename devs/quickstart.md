# Developer Quickstart

## Installing Dependencies
```shell
npm install @jackallabs/jackal.js
```

To get started building with Jackal, we first need to create a `StorageHandler`.
```ts
import type { IClientSetup, ClientHandler } from '@jackallabs/jackal.js'

const signerChain: string = 'jackal-1'
const mainnet = {
    signerChain,
    enabledChains: [signerChain],
    queryAddr: 'https://grpc.jackalprotocol.com',
    txAddr: 'https://rpc.jackalprotocol.com',
    chainConfig: {
        chainId: signerChain,
        chainName: 'Jackal Mainnet',
        rpc: 'https://rpc.jackalprotocol.com',
        rest: 'https://api.jackalprotocol.com',
        bip44: {
            coinType: 118
        },
        coinType: 118,
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
    endpoint: mainnet.txAddr,
    chainConfig: mainnet.chainConfig,
    chainId: mainnet.signerChain
}

const myClient = await ClientHandler.connect(setup)
const selectedWallet = myClient.getSelectedWallet()
const storage = await myClient.createStorageHandler()
```

Now to upload a file. 