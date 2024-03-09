---
description: Documentation for Getting Started with contract interactions within your app.
---

# Getting Started

If you haven't already read through the [Protocol and Contracts](broken-reference) section you may find that helpful context before working through this section.&#x20;

When building on the Intuition protocol there are offchain interactions (covered in the [API](broken-reference) section) and onchain interactions, which are covered in this section. The user journeys within our Portal app combine offchain and onchain interactions. Combining both of these types of flows are helpful to understand, even if you want to only integrate one type.&#x20;

This section covers core onchain interactions that correspond to [Creating an Identity](../api/identities/create-an-identity.md) and [Creating a Claim](../api/claim/create-a-claim.md) as well as creating onchain [Attestations](../primitives-and-interactions/primitives/attestations.md).

The examples and reference included in this section are included in our [Remix Starter Template](https://github.com/0xIntuition/app-template-remix) and are directly informed by how we use them in the Portal. Additionally, the examples used in our Starter (and covered in this section) can serve as a general guide for onchain interactions using Remix. You can follow the same patterns used and switch out the ABI and contract interactions at any point.&#x20;

### Manual Installation

This section includes _general installation instructions_ if you choose to not directly clone or fork our Remix starter. If you're using our starter you can skip over this content.

#### Installation

You'll need to install [`wagmi`](https://wagmi.sh/react/getting-started) and [`viem`](https://viem.sh/). Our starter currently uses `npm` but you should use whichever package manager you prefer.

{% hint style="info" %}
We are currently using `viem` v1.x and haven't fully tested `viem` 2. x yet. Once we do, we'll update our starters and our docs.
{% endhint %}

{% tabs %}
{% tab title="npm" %}
`npm install wagmi viem`
{% endtab %}

{% tab title="yarn" %}
`yarn add wagmi viem`
{% endtab %}
{% endtabs %}

#### Environment Variables

You'll need to set the following environment variables in a `.env` . You can reference these in the Remix Starter Templates [.env.example](https://github.com/0xIntuition/app-template-remix/blob/dev/.env.example)

<pre class="language-typescript" data-title=".env"><code class="lang-typescript"><strong>// include your values for these
</strong><strong>ALCHEMY_API_KEY=
</strong>ALCHEMY_RPC_URL=
WALLETCONNECT_PROJECT_ID=
</code></pre>

### Scaffolding the Interactions

The code examples used in this section are pulled from our [Remix Starter Template](https://github.com/0xIntuition/app-template-remix). If you're building these interactions yourself outside of our starter, you can structure the files however you prefer.&#x20;

#### Folder Conventions

Our starter and Portal use the following conventions:

* `lib` folder contains `abis`, `hooks` , and `utils` folders
* `abis` folder contains a viem compatible mapping of our [`EthMultiVault`](../protocol-and-contracts/contract-documentation.md) ABI
  * Name this file `ethMultiVault.ts`&#x20;
  * You can include other ABIs in this same structure, or completely swap out our contract for another!
* `utils` folder contains a `constants.ts` and a `viem.ts` configuration file

#### ConfigurationFiles

{% code title="ethMultiVault.ts" %}
```typescript
export const multivaultAbi = [{...}]
```
{% endcode %}

* The key takeaway is that this needs to be a `.ts` file and exported as `const` for viem's type inference and ABI support.
* Note: The actual ABI values are included inside the `{...}` in the above example.

{% code title="constants.ts" %}
```typescript
export const MULTIVAULT_CONTRACT_ADDRESS =
  '0x51eB0DCf0411714a3E126376a700532E6FC2e7d4'
```
{% endcode %}

{% code title="viem.ts" %}
```typescript
import { createPublicClient, getContract, http } from 'viem'
import { mainnet, optimismSepolia } from 'viem/chains'
import { MULTIVAULT_CONTRACT_ADDRESS } from './constants'
import { multivaultAbi } from '../abis/ethMultiVault'

const alchemyRpcUrl = process.env.ALCHEMY_RPC_URL
const alchemyMainnetRpcUrl = process.env.ALCHEMY_MAINNET_RPC_URL

export const publicClient = createPublicClient({
  batch: {
    multicall: true,
  },
  chain: optimismSepolia,
  transport: http(alchemyRpcUrl),
})

export const mainnetClient = createPublicClient({
  chain: mainnet,
  transport: http(alchemyMainnetRpcUrl),
})

export const getMultivaultContract = getContract({
  address: MULTIVAULT_CONTRACT_ADDRESS,
  abi: multivaultAbi,
  publicClient,
})

export const multiVaultContract = {
  address: MULTIVAULT_CONTRACT_ADDRESS,
  abi: multivaultAbi,
} as const

```
{% endcode %}

* This file is critical as it provides `viem` with the ABI (`multiVaultAbi`) ,  RPC information, and the `MULTIVAULT_CONTRACT_ADDRESS` from `constants.ts`
* Be sure to include the `export const multiVaultContract = {} as const`. This is necessary for viem contract interactions.
* You'll also need to check that `ALCHEMY_RPC_URL` and `ALCHEMY_MAINNET_RPC_URL` in your `.env`

#### useMultiVaultContract Hook

Each page in this section goes into more detail about specific hooks for specific contract interactions. This section includes an overview of our helper hooks for contract interactions. This is all included in our Remix Starter.

These hooks are included in `/lib/hooks` in our folder convention.

{% code title="useMultiVaultContract.tsxtsx" %}
```tsx
import { multivaultAbi } from '@/lib/abis/ethMultiVault'
import { getContract } from 'viem'
import { optimismSepolia } from 'viem/chains'
import { usePublicClient, type Address } from 'wagmi'
import { MULTIVAULT_CONTRACT_ADDRESS } from '../utils/constants'

export const MULTIVAULT_ADDRESS: Record<number, string> = {
  [optimismSepolia.id]: MULTIVAULT_CONTRACT_ADDRESS,
}

export const getMultivaultContractConfig = (chainId?: number) => ({
  address: (chainId && chainId in MULTIVAULT_ADDRESS
    ? MULTIVAULT_ADDRESS[chainId]
    : '') as Address,
  abi: multivaultAbi,
})

export function useMultivaultContract(chainId?: number) {
  const publicClient = usePublicClient({ chainId })

  return getContract({
    ...getMultivaultContractConfig(chainId || publicClient.chain.id),
    publicClient,
  })
}
```
{% endcode %}

* This hook provides scaffolding for using our contract interaction hooks
* Note: You'll need to create a `publicClient` with wagmi's [usePublicClient](https://wagmi.sh/react/api/hooks/usePublicClient) hook
* Once you've created this hook you can then use it in the other contract interactions that call functions

### Contract Reads

Once you've set up all of the configuration from the previous sections, you can use them for contract reads. The following example is Remix specific and is used in both our Portal app and our Remix Starter. You can generalize this for inclusion in a React framework of your choice.

We're using a [Remix resources route](https://remix.run/docs/en/main/guides/resource-routes) with a [loader](https://remix.run/docs/en/main/guides/resource-routes) to read from our contract and provide data to one of our Remix routes.

{% code title="/routes/resources+/create-identity.tsx" %}
```tsx
import { getMultivaultContract } from '@/lib/utils/viem'
import { type LoaderFunctionArgs, json } from '@remix-run/node'

export type CreateIdentityLoaderData = {
  atomCost: string
  atomCreationFee: string
}

export async function loader({ request }: LoaderFunctionArgs) {
  const [[atomCost, atomCreationFee]] = (await Promise.all([
    getMultivaultContract.read.atomConfig(),
  ])) as [[BigInt, BigInt]]

  return json({
    atomCost: atomCost.toString(),
    atomCreationFee: atomCreationFee.toString(),
  } as CreateIdentityLoaderData)
}

```
{% endcode %}

* This uses the `getMultivaultContract.read` to read the `atomConfig()` value from our contract.
* We then get the `atomCost` and `atomCreationFee` from the atomConfig() which we are returning as `json` for use in another route.&#x20;

This approach leverages Remix's data loading conventions to provide data from our contract read to a Route via a `useFetcher` on the route:

{% code title="sample-route.tsx" %}
```tsx
  // resource loader fetcher for fees
  const loaderFetcher = useFetcher<CreateIdentityLoaderData>()
  const loaderFetcherUrl = '/resources/create-identity'
  const loaderFetcherRef = React.useRef(loaderFetcher.load)

  React.useEffect(() => {
    loaderFetcherRef.current = loaderFetcher.load
  })

  React.useEffect(() => {
    loaderFetcherRef.current(loaderFetcherUrl)
  }, [loaderFetcherUrl])

  const { atomCost: atomCostAmount } =
    (loaderFetcher.data as CreateIdentityLoaderData) ?? {
      atomCost: BigInt(0),
      atomCreationFee: BigInt(0),
    }
```
{% endcode %}

This example is Remix specific, but you can integrate the contract reads in however way you need for your use case.

### Contract Writes

Once you've set up all of the configuration from the previous sections, you can use them for contract writes. The following example is used in both our Portal app and our Remix Starter.  There are some Remix specific parts, but you can generalize this for inclusion in a React framework of your choice.

The next pages in this section will go deeper into specific interactions. Let's look at two hooks used in our scaffolding for contract writes.

{% code title="useContractWriteAndWait.tsx" %}
```tsx
// this hook uses several lifecycle hooks from wagmi and types from viem

import { type WriteContractMode } from '@wagmi/core'
import { useEffect, useRef } from 'react'
import type { Abi, TransactionReceipt } from 'viem'
import {
  useContractWrite,
  usePrepareContractWrite,
  useWaitForTransaction,
  type UseContractWriteConfig,
  type UsePrepareContractWriteConfig,
} from 'wagmi'

type Handler = (receipt: TransactionReceipt) => void

type UseContractWriteAndWaitConfig = {
  onReceipt?: Handler
}

export function usePreparedContractWriteAndWait<
  TAbi extends Abi | readonly unknown[],
  TFunctionName extends string,
  TChainId extends number,
>(
  prepareContractWriteConfig: UseContractWriteAndWaitConfig &
    UsePrepareContractWriteConfig<TAbi, TFunctionName, TChainId>,
) {
  const { config } = usePrepareContractWrite<TAbi, TFunctionName, TChainId>(
    prepareContractWriteConfig,
  )

  return useContractWriteAndWait<TAbi, TFunctionName, 'prepared'>(config)
}

export function useContractWriteAndWait<
  TAbi extends Abi | readonly unknown[],
  TFunctionName extends string,
  TMode extends WriteContractMode,
>(
  contractWriteConfig: UseContractWriteAndWaitConfig &
    UseContractWriteConfig<TAbi, TFunctionName, TMode>,
) {
  const {
    data,
    isIdle,
    isLoading: awaitingWalletConfirmation,
    isError,
    writeAsync,
    reset,
  } = useContractWrite<TAbi, TFunctionName, TMode>(contractWriteConfig)

  const transactionSettledHandler = useRef<Handler>(() => {})

  const {
    data: receipt,
    isLoading: awaitingOnChainConfirmation,
    isSuccess,
  } = useWaitForTransaction({
    hash: data?.hash,
  })

  const onReceipt = useRef((handler: Handler) => {
    transactionSettledHandler.current = handler
  })

  useEffect(() => {
    if (receipt) {
      transactionSettledHandler.current(receipt)
    }
  }, [receipt])

  return {
    data,
    isIdle,
    isError,
    awaitingWalletConfirmation,
    awaitingOnChainConfirmation,
    receipt,
    writeAsync,
    reset,
    isSuccess,
    onReceipt: onReceipt.current,
  }
}

```
{% endcode %}

This hook is critical for contract writes.  Once this is in place, you can use it within other hooks that'll be covered in the next pages.

An optional enhancement that we've provided is our transaction state reducer that we use for more granular state control during interactions.

{% code title="useGenericTxReducer.tsx" %}
```tsx
import { type WriteContractMode } from '@wagmi/core'
import { useEffect, useRef } from 'react'
import type { Abi, TransactionReceipt } from 'viem'
import {
  useContractWrite,
  usePrepareContractWrite,
  useWaitForTransaction,
  type UseContractWriteConfig,
  type UsePrepareContractWriteConfig,
} from 'wagmi'

type Handler = (receipt: TransactionReceipt) => void

type UseContractWriteAndWaitConfig = {
  onReceipt?: Handler
}

export function usePreparedContractWriteAndWait<
  TAbi extends Abi | readonly unknown[],
  TFunctionName extends string,
  TChainId extends number,
>(
  prepareContractWriteConfig: UseContractWriteAndWaitConfig &
    UsePrepareContractWriteConfig<TAbi, TFunctionName, TChainId>,
) {
  const { config } = usePrepareContractWrite<TAbi, TFunctionName, TChainId>(
    prepareContractWriteConfig,
  )

  return useContractWriteAndWait<TAbi, TFunctionName, 'prepared'>(config)
}

export function useContractWriteAndWait<
  TAbi extends Abi | readonly unknown[],
  TFunctionName extends string,
  TMode extends WriteContractMode,
>(
  contractWriteConfig: UseContractWriteAndWaitConfig &
    UseContractWriteConfig<TAbi, TFunctionName, TMode>,
) {
  const {
    data,
    isIdle,
    isLoading: awaitingWalletConfirmation,
    isError,
    writeAsync,
    reset,
  } = useContractWrite<TAbi, TFunctionName, TMode>(contractWriteConfig)

  const transactionSettledHandler = useRef<Handler>(() => {})

  const {
    data: receipt,
    isLoading: awaitingOnChainConfirmation,
    isSuccess,
  } = useWaitForTransaction({
    hash: data?.hash,
  })

  const onReceipt = useRef((handler: Handler) => {
    transactionSettledHandler.current = handler
  })

  useEffect(() => {
    if (receipt) {
      transactionSettledHandler.current(receipt)
    }
  }, [receipt])

  return {
    data,
    isIdle,
    isError,
    awaitingWalletConfirmation,
    awaitingOnChainConfirmation,
    receipt,
    writeAsync,
    reset,
    isSuccess,
    onReceipt: onReceipt.current,
  }
}

```
{% endcode %}

While this is optional, we use it in our Portal transaction states and we've included it in our Remix Starter.&#x20;
