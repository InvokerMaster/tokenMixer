# An Ethereum mixer

This is the repo for all code and documentation for a noncustodial Ethereum
mixer. 

A mixer moves ETH or ERC20 tokens from one address to another in a way that
nobody except the sender can know for sure that these addresses are linked.
This mixer lets a user deposit fixed amounts of ETH into a contract, and when
the pool is large enough, anonymously submit zero-knowledge proofs which show
that the submitter had previously made a deposit, thus authorising the contract
to release funds to the recipient.

As a transaction relayer pays the gas of this transaction, there is no certain
on-chain connection between the sender and recipient. Although this relayer is
centralised, the mixer is noncustodial and no third party can exit with users'
funds.

This mixer is highly experimental and not yet audited. Do not use it to mix
real funds yet. It only supports Kovan ETH for now. Get Kovan ETH from a faucet

## Supported features

The current version of this mixer is a simple MVP for desktop Chrome, Brave, or
Firefox. You should also have [MetaMask](https://metamask.io/) installed, and
some Kovan ETH. You need at least 0.11 KETH to mix 0.1 ETH, and 20 Kovan DAI
and 0.01 ETH to mix Kovan DAI. You can generate Kovan DAI using MakerDAO's CDP
creation tool [here](https://cdp.makerdao.com).

It has the following features:

1. A user interface which allows:

    - One deposit per day.

    - One-click withdrawal once UTC midnight has passed.

    - Immediate self-withdrawals in case the user wants their funds back at the
      cost of privacy.
    
    - Immediate withdraw requests if the user wishes the operator to mix the
      funds immediately, which also comes at the cost of some privacy.

2. A backend server with one JSON-RPC 2.0 endpoint, `mixer_mix()`, which:
    
    - Accepts, verifies, and submits a zk-SNARK proof (generated in the user's
      browser) to the mixer contract.

3. Ethereum contracts:

    - The
      [Semaphore](https://github.com/kobigurk/semaphore/) zero-knowledge
      signalling system as a base layer.

    - A Mixer contract with functions which
        
        - Accepts ETH or ERC20 token deposits.

        - Accepts mix requests. Each request comprises of a zk-SNARK proof that
          a deposit had been made in the past and has not already been claimed.
          If the proof is valid, it transfers funds to the recipient and takes an
          operator's fee.
        
        - Allows the operator to withdraw all accurred fees.

## Local development and testing

These instructions have been tested with Ubuntu 18.0.4 and Node 11.14.0.

### Requirements

- Node v11.14.0.

- [`etcd`](https://github.com/etcd-io/etcd) v3.3.13
    - The relayer server requires an `etcd` server to lock the account nonce of
      its hot wallet.

### Local development

Install `npx` and `http-server` if you haven't already:

```bash
npm install -g npx http-server
```

```bash
cd mixer && \
git submodule update --init
```
Download the circuit, keys, and verifier contract. Doing this instead of
generating your own keys will save you about 20 minutes. Note that these are
not for production use as there is no guarantee that the toxic waste was
discarded.

```bash
./scripts/downloadSnarks.sh
```

<!--Next, download the `solc` [v0.4.25-->
<!--binary](https://github.com/ethereum/solidity/releases/tag/v0.4.25) make it-->
<!--executable, and rename it.-->

<!--```bash-->
<!--chmod a+x solc-static-linux && # whatever its name is-->
<!--mv solc-static-linux solc-0.4.25-->
<!--```-->

<!--Take note of the filepath of `solc-0.4.25` as you will need to modify the next-->
<!--command to use it.-->

Create a file named `hotWalletPrivKey.json` in a location outside this
repository with a private key which will serve as the operator's hot wallet.
The following private key corresponds to the address
`0x627306090abab3a6e1400e9345bc60c78a8bef57`, the first Ethereum address which
can be derived from the well-known `candy maple cake sugar...` mnemonic. Don't
use this in production.

```json
{
    "privateKey": "0xc87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3"
}
```

You can now run the frontend at http://localhost:1234.

To automatically compile the TypeScript source code whenever you change it,
first make sure that you have `npm run watch` running in a terminal. For
instance, while you edit `backend/ts/index.ts`, have a terminal open at
`backend/` and then run `npm run watch`.

If you use a terminal multiplexer like `tmux`, your screen might now look like this:

Clockwise from top right:

1. Ganache (`runGanache.sh`)
2. Frontend (`npm run watch`)
3. Deployed contracts (`npm run deploy`)
4. HTTP server (`http-server`)
5. Backend (`npm run server`)

## Testing

### Unit tests

#### Contracts

In the `mixer/contracts/` directory:

1. Run `npm run build` if you haven't built the source already
2. Run `npm run testnet`
3. In a separate terminal: `npm run test`

#### Backend

In the `mixer/contracts/` directory:

1. Run `npm run build` if you haven't built the source already
2. Run `npm run testnet`
3. Run `npm run deploy`

In the `mixer/backend/` directory:

1. Run `npm run build` if you haven't built the source already
2. Run `npm run test`


### Directory structure

- `frontend/`: source code for the UI
- `contracts/`: source code for mixer contracts and tests
- `semaphore/`: a submodule for the [Semaphore code](https://github.com/weijiekoh/semaphore)

### Frontend

See the frontend documentation [here](./frontend).