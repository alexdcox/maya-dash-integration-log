# Maya Dash Integration Log

[Trello](https://trello.com/c/QDnbCI5h/192-maya-protocol-integration)  
[Maya Github](https://github.com/mayachain)  
[Maya Gitlab](https://gitlab.com/mayachain)  
[Maya Whitepaper](https://assets.website-files.com/62a14669b65c6aeed054f32e/62b3648681be198a7fd33120_MAYA%20PROTOCOL%20WHITE%20PAPER_.pdf)  


## Batch One

```
25.01.2023 Friday      7h
30.01.2023 Monday      6h
31.01.2023 Tuesday     6h
01.01.2023 Wednesday   2h
02.01.2023 Thursday    6h
03.02.2023 Friday      6h
07.02.2023 Tuesday     5h
Total                 38h
```

### 25.01.2023 Friday 7h

Starting on further research about what it is I need to do here.

> Maya aims to take the Thorchain model and go a step further through implied
  capital efficiency with more uses for its native currency $CACAO as well as
  being backward compatible with Thorchain

So do they use the same testing suite `heimdall`?

> The Aztech chain is a fork of the Terra blockchain to provide a mature
  infrastructure for smart contract development.

I dare say I'll need to be able to spin up an aztech node then?

The [whitepaper](https://assets.website-files.com/62a14669b65c6aeed054f32e/62b3648681be198a7fd33120_MAYA%20PROTOCOL%20WHITE%20PAPER_.pdf)
says a lot about working together with thorchain, to backup and help develop
the concept of decentralised exchanges. Much like peer testing within the
scientific community.

> Maya takes security very seriously, which is why the most sensitive aspects of
  the THORChain protocol, such as Bifröst, were left untouched.

`Imxu` is their equivalent for `Mimir`.  
Looks like I will need to navigate their fair launch myself for a private chain?

1. Users should only add/withdraw liquidity.
2. Users should not be able to swap or send.
3. Users should not be able to get $CACAO until the end of the Liquidity Auction.

Launches on March 7th. 

> Any and all UI’s supporting the Maya Stagenet—and therefore our Mainnet— can
  host the Liquidity Auction. Code Savvy individuals may also use the
  API/Transaction Memos directly.

They mention three different UIs:
- THORWallet Dex
- Asgardex
- Mayanode cli

> Our first nodes will be called “Genesis Nodes”, and there will be six of them.

Wonder if I have to follow suit?

> Genesis nodes will be approved using a specific custom-made token for this purpose, they will not be entitled to any fees, special allocations or pre-mines of any kind. For more details on our Genesis Nodes please refer to Part 4: Security Nodes of this document.

> As a genesis node, I should be able to be a validator in the chain without contributing economically and without affecting the $CACAO supply. Also, I should not get any sort of pre-mine or reward during this period.

The Maya equivalent (of `bifrost`) is `Yax bridge`.

Differences between $MAYA and $CACAO?

Cacao:
- flagship coin
- 100M initially minted all at once via liquidity auction
- required to run nodes
- earn fees on swaps

Maya:
- used towards protocol total revenues
- 1M
- initial stages funding
- dev incentives
- > they allowed us to financially bootstrap our project in its earliest stages without having to recur to any pre-sales of $CACAO
- > In the specific case of $MAYA tokens, they don’t give any governance rights to their holders, or any other right whatsoever

```
P/E = $MAYA last public price / 10% * Annualized Protocol Fee Revenue
EPS = 10% * Annualized Protocol Fee Revenue / 1,000,000
```

I've been told they're targeting thornode `v104` for the initial release.

Which version did my old branches target?  
thorchain/thornode MR last update:  
Wed Aug 10 09:49:41 2022 -0700  
Wed Aug 10 16:49:41 2022 UTC  
```
TZ=UTC git log $(git rev-parse --short HEAD) --date=local
```

last `origin/develop` commit I have on the `982-add-dash-chain` is: `8f7c5bb8c`
  
Just ran `git fetch --tags` and saw `v1.104.0` the exact version format we're targeting here.  
I think `1.92.0` was the last version my code worked for.

Is there any quickstart guide to getting a local private network running?

They have `cluster-launcher` and `node-launcher`.

`helm-plugins` has been superceeded by `helm plugin`

Alright trying to get `node-launcher` working against the default k8s docker cluster installed with mac. Let's see how painful it is...

I don't want to run `make helm` because my version of helm is managed by homebrew package manager for mac, so that would override my development environment, which I'm already not happy about.

Then they tell me to run `make helm-plugins`.  
Already exists error. That's ok.  

I'd much rather have all these commands in a sequential bash script where I can read down what it's doing to my system and double check, without having to zip around a makefile. Actually this reminds me of my original cluster setup repo which just used docker and didn't modify the host system at all. Might need to revisit that.

Yeah this was that:  
https://github.com/alexdcox/thorchain-cluster

Oh it's all in one convenient docker-compose file. Fuck. Yes. That's how it should be god damn.

I'm going to update that for maya.

Apparently it's `v1.101.0` not 104.

`docker image pull registry.gitlab.com/thorchain/thornode:mocknet-1.101.0`
> Error response from daemon: manifest for registry.gitlab.com/thorchain/thornode:mocknet-1.101.0 not found: manifest unknown: manifest unknown

Changed location? I do remember they were in the process of splitting images out into another repo...  
Ah yeah it was their `node-launcher`:
https://gitlab.com/thorchain/devops/node-launcher/-/tree/master/ci/images

Looking at node-launcher, doge seems to be disabled.  
Is doge running on the current thornode testnet?  
How do I find that out? I can't remember.  
https://app.skip.exchange is HAVING ISSUES  
https://app.thorswap.finance/swap is OK  
https://testnet.app.skip.exchange is BROKEN  
https://testnet.midgard.thorchain.info is DOWN  
https://testnet.seed.thorchain.info is DOWN  
https://testnet.thorswap.finance/ is PASSWORD PROTECTED :(  
https://stagenet-midgard.ninerealms.com is OK  

```bash
curl https://stagenet-midgard.ninerealms.com/v2/pools | jq '.[].asset'
```
```
"BCH.BCH"
"ETH.USDT-0XDAC17F958D2EE523A2206206994597C13D831EC7"
"AVAX.AVAX"
"BNB.BNB"
"GAIA.ATOM"
"BTC.BTC"
"DOGE.DOGE"
"ETH.ETH"
"LTC.LTC"
```

So there you are. Doge is indeed part of the stagenet, and that's the easiest way to tell.

It's a shame they've password protected testnet, I could do with looking at it.

Ahh I'll just rebuild thornode too. I think I used to do that.

`gco release-1.101.0`

How do I rebuild thornode/bifrost for my setup?

I left a hidden/ignored script in the `thornode` repo at `./_adc/docker-build-image.sh` which builds to:  
registry.gitlab.com/alexdcox/thornode:mocknet-v1.101.1

Use the platform first to avoid a load of errors:
```
export DOCKER_DEFAULT_PLATFORM=linux/amd64
./_adc/docker-build-image.sh
```

The nice thing about this script is that it overwrites the original to simply add `bash` and `socat` to facilitate my setup scripts.

Still got this warning in step 11 `RUN make protob install`:
> The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested

Hopefully I can ignore that.

How long does that take to build again? 5 mins for a fail:

```
Step 11/17 : RUN make protob install
 ---> [Warning] The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
 ---> Running in 4474657af929
go install -ldflags '-X gitlab.com/thorchain/thornode/constants.Version=1.101.0 -X gitlab.com/thorchain/thornode/constants.GitCommit=9172986498ea380a616ff599984d23c2c9f16622 -X gitlab.com/thorchain/thornode/constants.BuildTime=2023-01-28_03:52:31 -X github.com/cosmos/cosmos-sdk/version.Name=THORChain -X github.com/cosmos/cosmos-sdk/version.AppName=thornode -X github.com/cosmos/cosmos-sdk/version.Version=1.101.0 -X github.com/cosmos/cosmos-sdk/version.Commit=9172986498ea380a616ff599984d23c2c9f16622 -X github.com/cosmos/cosmos-sdk/version.BuildTags=mocknet' -tags mocknet ./cmd/thornode ./cmd/bifrost ./tools/generate
# gitlab.com/thorchain/thornode/x/thorchain
x/thorchain/querier_quotes.go:168:100: undefined: openapi.QuoteSwapResponse
x/thorchain/querier.go:161:23: undefined: openapi.ThornameAlias
x/thorchain/querier.go:163:37: undefined: openapi.ThornameAlias
x/thorchain/querier.go:171:3: unknown field 'Owner' in struct literal of type openapi.Thorname
x/thorchain/querier.go:172:3: unknown field 'PreferredAsset' in struct literal of type openapi.Thorname
x/thorchain/querier.go:173:3: unknown field 'Aliases' in struct literal of type openapi.Thorname
x/thorchain/querier.go:413:3: unknown field 'VaultsMigrating' in struct literal of type openapi.NetworkResponse
x/thorchain/querier.go:436:20: undefined: openapi.POLResponse
x/thorchain/querier.go:516:4: unknown field 'GlobalTradingPaused' in struct literal of type openapi.InboundAddress
x/thorchain/querier.go:517:4: unknown field 'ChainTradingPaused' in struct literal of type openapi.InboundAddress
x/thorchain/querier.go:517:4: too many errors
make: *** [Makefile:98: install] Error 2
The command '/bin/sh -c make protob install' returned a non-zero code: 2
Sending build context to Docker daemon  66.09kB
Step 1/2 : FROM registry.gitlab.com/alexdcox/thornode:mocknet-1.101.0
manifest for registry.gitlab.com/alexdcox/thornode:mocknet-1.101.0 not found: manifest unknown: manifest unknown
```

`make openapi` didn't work with this error:

```
GNU awk is required.
GNU find is required.
GNU sed is required.

Mac OS can try the following to update native utilities to the latest GNU version (homebrew):
1. Run: brew install coreutils binutils diffutils findutils gnu-tar gnu-sed gawk grep make
2. Follow the instructions from the brew output to update your PATH so the GNU utilities are default
Makefile:7: *** Check environment dependencies..  Stop.
```

Here's how to generate the openapi on m1 mac:

...

Well I thought that would work.  
Gave this weird docker error:  

```
Digest: sha256:310bd0353c11863c0e51e5cb46035c9e0778d4b9c6fe6a7fc8307b3b41997a35
docker: Cannot overwrite digest sha256:310bd0353c11863c0e51e5cb46035c9e0778d4b9c6fe6a7fc8307b3b41997a35.
```

Had to drop the sha256 being included with the image, perhaps this was copypasta which was broken. Probably just a case of 9R going a bit security mad and then not keeping up with their own rules.

```
docker run \
  --rm \
  -v $(pwd)/openapi:/mnt \
  openapitools/openapi-generator-cli:v6.0.0 \
    generate \
      -g go \
      -i /mnt/openapi.yaml \
      -o /mnt/gen
rm openapi/gen/go.mod openapi/gen/go.sum
```

Retrying rebuild...

While that's happening, checking the bifrost client changes between where I stopped developing with dash and the target v1.101.0 (using goland compare branch feature - it's nice).

clients:
- Add `DefaultCoinbase` method, drop constant `DefaultCoinbaseValue`
- Add `DustThreshold` method

signer:
- `DustThreshold`
- `signUTXO` error handling refactored
- broadcast tx failures with txalreadyinchain error are cached for potential future handling
- sign tx calls `utxo.PostKeysignFailure`

Just in case I have to switch back to them in future to see the diff I'm seeing right now, here are the two significant commits:

```
release-1.101.0     917298649 
982-add-dash-chain  64871be0c
```

Plan of attack:
- switch to the dash branch
- merge 1.101.0 in
- update dash in-line with other bifrost clients
- rebuild thornode
- update heimdall
- re-run tests
- tell the world

There's now a `test/smoke` directory.

Does that mean they've moved away from the external `heimdall` and brought it back into the repo? Yeah, and apparently I've already addresses bringing the dash changes in because there's a merge conflict. I'll just have to accept their changes and redo them:

`test/smoke/data/smoke_test_balances.json`  
`test/smoke/data/smoke_test_events.json`  
`test/smoke/thorchain/thorchain.py`  

Right now how do I regen balances/events?

Oh before I do that... bifrost client changes to dash are required.

Is the dash default coinbase 10K satoshis okay?  
See `chain.go` `DefaultCoinbase()` method.  
I've wondered this many times and am yet to find an answer.  
Just scanned the entire dash c++ repo and didn't find any reference to default coinbase value.  

Added commit:  
Update dash bifrost client to use DefaultCoinbase and DustThreshold  

```
go test -tags mocknet ./...
```

Fails.  
What about on the release version?  
Fails.  
Hmm, openapi, protobuf?  

Remember `protogenc.sh` doesn't work because mac `find` isn't the same as GNU find. Use this then checkout protocgen:

```
gsed -i "s+find+gfind+g" ./scripts/protocgen.sh
./scripts/protocgen.sh
```

openapi docker container now hanging for unknown reasons.  
no logs.

```
/usr/local/bin/docker-entrypoint.sh generate \
  -g go \
  -i /mnt/openapi.yaml \
  -o /mnt/gen
```

Yep, hanging. Why?  
Not really sure.  
Let's restart...  
That did the trick for some reason.  
Wait.  
Did it?  
It's stuck at writing `InboundAddresses.md`...  
I'll give it 5 mins...  
I gave it 2 mins.  
Gah.  
I'll try `v6.2.1` / `latest`.  
Yeah that worked fine.  
But tests fail.  
`rm -rf ./openapi/gen`  
Back to v6.  
Ok better.  

Kay go tests...   

release-1.101.0... PASSING.  
mine... seem to be frozen post `bitcoincash`?  
dash tests failed.  
O.  

```
go test -tags mocknet ./bifrost/pkg/chainclients/dash
```

```
FAIL: signer_test.go:312: DashSignerSuite.TestEstimateTxSize

signer_test.go:325:
    c.Assert(size, Equals, int64(417))
... obtained int64 = 451
... expected int64 = 417
```

`OUT:2180B871F2DEA2546E1403DBFE9C26B062ABAFFD979CF3A65F2B4D2230105CF1`  
68 character memo (68 bytes) with 2 txs.  

```
4 + 1 + (147 * len(txes)) + 1 + (34 * assumedOutputs) + 11 + len([]byte(memo)) + 4)

= 5 + 294 + 1 + 68 + 11 + 68 + 4

= 451
```

Test wrong. Code right. How did that slip through?

Tests passing. 47s left of my extra hour on the clock. Nice!

That's all I have time for today.

### 30.01.2023 Monday 6h

Rebuild thorchain...  
Smoke tests...  

What version is my dash node?

```
docker run --rm github.com/alexdcox/dash:latest dash-cli version
```

> Unable to find image 'github.com/alexdcox/dash:latest' locally

Ah. On my old laptop.  
Time to rebuild dash...  

It was `v0.17.0.0-rc4`.  
Latest is `18.2.1` 2 weeks ago.  

```
DOCKER_BUILDKIT=0 docker build -t registry.gitlab.com/alexdcox/dash-core .
```

> Cloning into 'libsodium'...
> fatal: unable to connect to github.com:
> github.com[0: 140.82.113.3]: errno=Connection refused

Wot!?  
https://github.com/jedisct1/libsodium  
... is live and kicking.  

```
docker run --rm -it ubuntu:16.04 bash
apt update && apt install -y git
git clone https://github.com/jedisct1/libsodium.git
```

No issues.  
Wild temporary internet hiccup?  
Same again.  

Okay I'll manually run all those steps and see if I can replicate...  
Ah I think I see, it's using the `git://` url scheme rather than `https://`.  
Yep. Perhaps that's outdated, changed, fixed.  

> Error response from daemon: error parsing HTTP 404 response body: no error details found in HTTP response body: "{\"error\":\"Not Found\"}"

I ought to update all the other nodes I suppose:

This is where we want to source all the container images from.  
https://gitlab.com/thorchain/devops/node-launcher/container_registry/2753226

```bash
registry="registry.gitlab.com/thorchain/devops/node-launcher"
docker image pull $registry:bitcoin-cash-daemon-26.0.0
docker image pull $registry:bitcoin-daemon-24.0.1
```

All containers started. That's good.

> /docker/scripts/orchestrator.sh: line 15: socat: command not found

```
DOCKER_BUILDKIT=0 \
DOCKER_DEFAULT_PLATFORM=linux/arm64 \
docker build \
-t registry.gitlab.com/alexdcox/dash-core:0.18.2.1 \
.

DOCKER_DEFAULT_PLATFORM=linux/arm64 \
docker run \
-it \
--rm \
--name target \
--pull never \
--entrypoint bash \
registry.gitlab.com/alexdcox/dash-core:0.18.2.1

apt install -y socat net-tools

docker commit target registry.gitlab.com/alexdcox/dash-core:0.18.2.1

docker tag registry.gitlab.com/thorchain/devops/node-launcher:dash-daemon-0.18.2.1 registry.gitlab.com/alexdcox/dash-core:0.18.2.1
```

- `registry.gitlab.com/alexdcox/thornode:mocknet-1.101.0`
- `registry.gitlab.com/alexdcox/dash-core:0.18.2.1`

`dash1` is:
> Waiting for node (dash1) verification...

Which never happens, potentially because of this error log:  
> Adding fixed seed nodes as DNS doesn't seem to be available.

Missing script dependency `jq`.  
Now stuck at:  
> Waiting for node (dash1) to reach block 500...

`generate` is now `generatetoaddress`

Now stuck at:

> Waiting for masternode (dash2) to reach MASTERNODE_SYNC_FINISHED state...

It's actually at `MASTERNODE_SYNC_BLOCKCHAIN`.  
Perhaps the sync behaviour has changed.

https://dashcore.readme.io/docs/core-guide-dash-features-masternode-sync

```bash
dash-cli -rpcconnect=dash1 mnsync status
{
  "AssetID": 1,
  "AssetName": "MASTERNODE_SYNC_BLOCKCHAIN",
  "AssetStartTime": 1675144282,
  "Attempt": 0,
  "IsBlockchainSynced": false,
  "IsSynced": false
}

dash-cli -rpcconnect=dash2 mnsync status
{
  "AssetID": 1,
  "AssetName": "MASTERNODE_SYNC_BLOCKCHAIN",
  "AssetStartTime": 1675144294,
  "Attempt": 0,
  "IsBlockchainSynced": false,
  "IsSynced": false
}

dash-cli -rpcconnect=dash1 getblockcount
689
dash-cli -rpcconnect=dash2 getblockcount
690

dash-cli -rpcconnect=dash1 masternode count
{
  "total": 3,
  "enabled": 3
}

dash-cli -rpcconnect=dash2 masternode count
{
  "total": 3,
  "enabled": 3
}

dash-cli -rpcconnect=dash1 masternode status
error code: -32603
error message:
This is not a masternode

dash-cli -rpcconnect=dash2 masternode status
{
  "outpoint": "b1ff8a94bf77cae065e378bc0e777c1a98ec23bad11dd71ea50452233735e66b-1",
  "service": "172.32.62.2:19899",
  "proTxHash": "b1ff8a94bf77cae065e378bc0e777c1a98ec23bad11dd71ea50452233735e66b",
  "collateralHash": "b1ff8a94bf77cae065e378bc0e777c1a98ec23bad11dd71ea50452233735e66b",
  "collateralIndex": 1,
  "dmnState": {
    "service": "172.32.62.2:19899",
    "registeredHeight": 563,
    "lastPaidHeight": 712,
    "PoSePenalty": 0,
    "PoSeRevivedHeight": -1,
    "PoSeBanHeight": -1,
    "revocationReason": 0,
    "ownerAddress": "yYvcdaXN2rPnYpQ6Y4jLfTF2H4baXK3cRf",
    "votingAddress": "ygbUR31HmemCkmYrb5mafCXxJxSQL83aAt",
    "payoutAddress": "yYT4P5xP4jszKM5XJ51QB3PGBjxGih46qo",
    "pubKeyOperator": "88358e58e34ca893602492c37aed2222a0d54168a22e731c8874d1ad6fe5d2ee68ac42af2aa6725c34a30811f9c1177b"
  },
  "state": "READY",
  "status": "Ready"
}

dash-cli -rpcconnect=dash3 masternode status
{
  "outpoint": "bd2c3688d65c75aa5f8fa2c921f5cd38a6e58eac10f7d3e8bc49b7ae8cb4cebe-1",
  "service": "172.32.62.3:19899",
  "proTxHash": "bd2c3688d65c75aa5f8fa2c921f5cd38a6e58eac10f7d3e8bc49b7ae8cb4cebe",
  "collateralHash": "bd2c3688d65c75aa5f8fa2c921f5cd38a6e58eac10f7d3e8bc49b7ae8cb4cebe",
  "collateralIndex": 1,
  "dmnState": {
    "service": "172.32.62.3:19899",
    "registeredHeight": 561,
    "lastPaidHeight": 714,
    "PoSePenalty": 0,
    "PoSeRevivedHeight": -1,
    "PoSeBanHeight": -1,
    "revocationReason": 0,
    "ownerAddress": "ygpr9LqWRxbUZc4Vu4UAGXbp3adPSJEmmv",
    "votingAddress": "yZttYk5Yf48nrUyMEhNNPLNbqqwVHC763K",
    "payoutAddress": "yiFp6Si61ZWrapeNbBpH9SuuRbCHH1DajB",
    "pubKeyOperator": "93bf145b444390c5936b3b1d245b1dd917d4f95f32a8cc9e9880fdf52fe3379d52de4f9fb39921779c80fc961f45035b"
  },
  "state": "READY",
  "status": "Ready"
}

dash-cli -rpcconnect=dash4 masternode status
{
  "outpoint": "7a92375505d09f6e7e9314fa9aeb8a618a0379718c8bf6c117c638c1b0dbeaa0-1",
  "service": "172.32.62.4:19899",
  "proTxHash": "7a92375505d09f6e7e9314fa9aeb8a618a0379718c8bf6c117c638c1b0dbeaa0",
  "collateralHash": "7a92375505d09f6e7e9314fa9aeb8a618a0379718c8bf6c117c638c1b0dbeaa0",
  "collateralIndex": 1,
  "dmnState": {
    "service": "172.32.62.4:19899",
    "registeredHeight": 562,
    "lastPaidHeight": 716,
    "PoSePenalty": 0,
    "PoSeRevivedHeight": -1,
    "PoSeBanHeight": -1,
    "revocationReason": 0,
    "ownerAddress": "ybGvVbR7nLiNr35UA7FB4rrFZgRKrhNFqq",
    "votingAddress": "ycehtEeSTqiz6jxjRjPUCzcMdWseEMWCcy",
    "payoutAddress": "ydCjRJFM9zW8W4S2SDpimTCJKshFQnuqSr",
    "pubKeyOperator": "19d049974abf9e43714314bb75345e3968e88d15dadd4a232a3663a09eb91af5d0189aa412ce2f31a844f67f9c98e5bf"
  },
  "state": "READY",
  "status": "Ready"
}

```

Well everything looks ok.  
What am I not doing?

```
dash-cli -rpcconnect=dash1 getblockchaininfo | jq '.verificationprogress'
1
dash-cli -rpcconnect=dash2 getblockchaininfo | jq '.verificationprogress'
1
dash-cli -rpcconnect=dash3 getblockchaininfo | jq '.verificationprogress'
1
dash-cli -rpcconnect=dash4 getblockchaininfo | jq '.verificationprogress'
1
```

They're all blockchain synced.

`dash-cli -rpcconnect=dash1 getblockchaininfo | jq '.initialblockdownload'`  
Comes back `false`.  
Could that be an issue?  

```bash
dash-cli -rpcconnect=dash2 mnsync status | jq '.AssetName'
"MASTERNODE_SYNC_BLOCKCHAIN"
dash-cli -rpcconnect=dash3 mnsync status | jq '.AssetName'
"MASTERNODE_SYNC_BLOCKCHAIN"
dash-cli -rpcconnect=dash4 mnsync status | jq '.AssetName'
"MASTERNODE_SYNC_BLOCKCHAIN"
```

Can I manually get it started?

```bash
dash-cli -rpcconnect=dash2 mnsync next
sync updated to MASTERNODE_SYNC_GOVERNANCE
dash-cli -rpcconnect=dash2 mnsync next
sync updated to MASTERNODE_SYNC_FINISHED
```

Well that's pure dead brilliant.

Now there's an error activating the sporks.

```
Writing config file to: /root/.dashcore/dash.conf
Waiting for node (dash1) verification...
Dash node ready.
Generating 500 initial blocks...
Waiting for node (dash1) to reach block 500...
Block 500 reached.
Waiting for node (dash2) to reach block 500...
Block 500 reached.
Waiting for masternode (dash2) to reach READY state...
Masternode ready.
Waiting for masternode (dash2) to reach MASTERNODE_SYNC_FINISHED state...
Masternode ready.
Waiting for node (dash3) to reach block 500...
Block 500 reached.
Waiting for masternode (dash3) to reach READY state...
Masternode ready.
Waiting for masternode (dash3) to reach MASTERNODE_SYNC_FINISHED state...
Masternode ready.
Waiting for node (dash4) to reach block 500...
Block 500 reached.
Waiting for masternode (dash4) to reach READY state...
Masternode ready.
Waiting for masternode (dash4) to reach MASTERNODE_SYNC_FINISHED state...
Masternode ready.
Waiting for node (dash1) to establish 3 peer connections...
Node dash1 has 3 peers.
Activating sporks...
success
success
success
Waiting for quorum 'llmq_test' to be established...
Finished setting up the LLMQ in 17 seconds
[orchestrator] Writing 'dash1' ready...
Verifying instantsend and chainlock...
--> tx hash: 63b92b2cffc8c62a37f8d1211f3fcea177ef33fc8e1de4e58aeee7e69b3c18d6
--> instantlock OK
--> chainlock OK
```

A thing of beauty.

How about the other nodes?  
`docker exec -it bitcoincash1 bitcoin-cli getblockcount`  
> error: Could not connect to the server 127.0.0.1:8332

Damn.

> mkdir: cannot create directory '/root': Permission denied

```
image="registry.gitlab.com/thorchain/devops/node-launcher"
docker run \
  --rm \
  --entrypoint bash \
  -it \
    $image:bitcoin-cash-daemon-26.0.0
```

> bash: cd: /root/: Permission denied

They've changed the user to `bitcoin` and set the group directory permission within the bitcoin user home directory `/home/bitcoin/.bitcoin` to be owned by `root:root`. So that's never going to be usable, perhaps unless you overwrite that directory with a docker volume.

According to the docker docs, if a directory exists, a volume will inherit the permissions of that directory. Otherwise I think it just defaults to root. I'll modify the dockerfile to create it and see...

Note to me: The `node-launcher` image build script at `./ci/images/build.sh` doesn't work on mac, nor is it appropriate for offline development. I've modified it and left a usable version at `./_scripts/docker-build-ci-images.sh` (git ignored)

And we wait...

What are the dashboard / explorer repos again?  
How do I run the smoke tests again?  

Need to go through the readme suggested commands, understand them, get them working for m1 mac and my local cluster.

---

```
make reset-mocknet-cluster
```

What does that actually do?

```bash
docker compose \
  -f build/docker/docker-compose.yml \
  --profile mocknet-cluster \
  --profile midgard \
  down -v
  
docker compose \
  -f build/docker/docker-compose.yml \
  --profile mocknet-cluster \
  --profile midgard \
  build
  
docker compose \
  -f build/docker/docker-compose.yml
  --profile mocknet-cluster \
  --profile midgard \
  images
  
docker compose \
  -f build/docker/docker-compose.yml \
  --profile mocknet-cluster \
  --profile midgard \
  ps
```

```bash
docker-compose \
  --env-file $(pwd)/.env \
  -p thorchain \
  -f $(pwd)/docker/docker-compose.yaml ps

composeargs="--env-file $(pwd)/.env -p thorchain -f $(pwd)/docker/docker-compose.yaml"

docker-compose $(echo $composeargs) ps
```

Don't actually need to run any of that.

---

Now this:

```bash
make cli-mocknet
  thornode tx thorchain mimir CHURNINTERVAL 1000 --from dog $TX_FLAGS
```

Not sure what my keychain was called. `thorchain` I think.

```bash
docker exec -it thornode1 bash

thornode tx thorchain mimir CHURNINTERVAL 1000 \
  --from thornode1 \
  --node http://thornode:26657 \
  --keyring-backend file \
  --chain-id thorchain
```

Key not found. Hmm.

Will have to pick up from here tomorrow.

### 31.01.2023 Tuesday 6h

```bash
thornode tx thorchain mimir CHURNINTERVAL 1000 \
  --node http://thornode1:26657 \
  --from thornode1 \
  --chain-id thorchain \
  --keyring-backend file
```

```bash
thornode keys --keyring-backend file list
- name: thorchain
  type: local
  address: tthor1zzja0wl7etldc4caprkfjwd74k68qqvfl32tj5
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AzE2zPSwVl6Dbqq4AFD3amw3m7hQCNgBLIQEIjUeefWH"}'
  mnemonic: ""
```

```bash
thornode tx thorchain mimir CHURNINTERVAL 1000 \
  --node http://thornode1:26657 \
  --from thorchain \
  --chain-id thorchain \
  --keyring-backend file
```
```bash
Enter keyring passphrase:
{"body":{"messages":[{"@type":"/types.MsgMimir","key":"CHURNINTERVAL","value":"1000","signer":"tthor1zzja0wl7etldc4caprkfjwd74k68qqvfl32tj5"}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[],"gas_limit":"200000","payer":"","granter":""}},"signatures":[]}

confirm transaction before signing and broadcasting [y/N]: y
{"height":"1404","txhash":"6F49D17362E7AA4F38D8AAEBAE30D98958622A77DA178C3FB95155BAF445D2E8","codespace":"sdk","code":5,"data":"","raw_log":"failed to execute message; message index: 0: 0rune is smaller than 2000000rune: insufficient funds","logs":[],"info":"","gas_wanted":"200000","gas_used":"74510","tx":null,"timestamp":"","events":[{"type":"tx","attributes":[{"key":"ZmVl","value":"","index":false}]},{"type":"tx","attributes":[{"key":"YWNjX3NlcQ==","value":"dHRob3IxenpqYTB3bDdldGxkYzRjYXBya2Zqd2Q3NGs2OHFxdmZsMzJ0ajUvMA==","index":false}]},{"type":"tx","attributes":[{"key":"c2lnbmF0dXJl","value":"MU9MVmp3Vkk1V1pIYWg4bFFGbFgwWTFBSmxrMCtsMjg1cUp1Y2hmU0RCWVhQWGw2R01kQWwvMlA1UkZtODhlblcwNlZyZ3Z0MGF4a05xZmtoZDdzL3c9PQ==","index":false}]}]}
```

There we go.  
Might have to change this to match - note to self.

---

```
make mocknet-bootstrap
```

I assume they mean `bootstrap-mocknet`.

```
docker run \
  --network host \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(pwd)/test/smoke:/app \
    registry.gitlab.com/thorchain/thornode:smoke \
    python scripts/smoke.py --bootstrap-only=True
```

Don't think I have to do much except change the network.

```bash
docker run \
  --network thorchain_thornode \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(pwd)/test/smoke:/app \
    registry.gitlab.com/thorchain/thornode:smoke \
    python scripts/smoke.py --bootstrap-only=True
```

> ModuleNotFoundError: No module named 'thornode_proto'

```
docker run \
  -it \
  -w /app \
  -v $(pwd)/test/smoke:/app \
  --rm \
  --entrypoint bash \
  registry.gitlab.com/thorchain/thornode:smoke
```

```
pip install -r requirements.txt
python scripts/smoke.py --bootstrap-only=True
```

Same error.

```
cd test/smoke
docker build -t registry.gitlab.com/thorchain/thornode:smoke .
```

I don't like having to actually install and run python shit on my local machine just to have this working. Should be all wrapped up in a container.

I think I'll just do that now.

```
docker run \
  -it \
  --rm \
  --name thornode-proto \
  -w /mnt \
  -v $(pwd):/mnt \
  ubuntu:latest

apt update

# python
apt install software-properties-common
add-apt-repository ppa:deadsnakes/ppa
apt update
apt install -y python3.8

# pip
apt install curl
curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10

# protoc
apt install protobuf-compiler

# thornode
pip3 install --upgrade markupsafe==2.0.1 betterproto[compiler]==2.0.0b4
```

```
docker commit thornode-proto thornode-proto
```

Rebuild python proto using container:

```
docker run \
  -it \
  --rm \
  -w /mnt \
  -v $(pwd):/mnt \
    thornode-proto

rm -rf test/smoke/thornode_proto
mkdir -p test/smoke/thornode_proto
protoc \
  -I ./proto \
  -I ./third_party/proto \
  --python_betterproto_out=test/smoke/thornode_proto \
  $(find ./proto -path -prune -o -name '*.proto' -print0 | xargs -0)

```

### 01.01.2023 Wednesday 2h

Can only spend another 2h working today, had to focus on other things.  
The rest of the time is dedicated to Maya...  
Ok where am I?  

```
docker run \
  -it \
  -w /app \
  -v $(pwd)/test/smoke:/app \
  --rm \
  --entrypoint bash \
  --network thorchain_thornode \
  registry.gitlab.com/thorchain/thornode:smoke
  
# pip install -r requirements.txt
python scripts/smoke.py --bootstrap-only=True
```

`bifrost1` (and 2) keep restarting.  
> unknown shorthand flag: 'c' in -c  

Oh this is the config file argument in `docker-compose`.  
Why have they removed that??  

Looks like all bifrost config now comes from environment variables.

`midgard` is also showing signs of problems.  
Is it just because `thornode1` isn't running yet?  

> missing chain RPC host

I've added `CHAIN_DISABLED` environment variables to everything except DASH and BCH.

> {"level":"error","service":"bifrost","module":"thorchain_client","error":"failed to get node status: failed to get node status: failed to get node account: Status code: 404 Not Found returned","time":"2023-02-02T02:05:42Z","caller":"/app/bifrost/thorclient/thorchain.go:301","message":"observer is not whitelisted , will retry a bit later"}

Hopefully this can be ignored / will self-resolve.

`BTC` tried to load.
They don't allow the following chains to be disabled, currently:
- `BCH`
- `BINANCE` aka `BNB`
- `BTC`
- `ETH`

I've added the ability to disable them, but I dare say they'd argue they never want to allow those to be disabled. For development I'd say be as flexible as possible.


Loads of the configuration values haven't been mapped to environment variables, and seeing as you can no longer provide the configuration file, some parameters can no longer be configured at all.

If it aint broke?

It's important for me to be able to set the database path because I'm intending to debug in goland to iterate quickly and add breakpoints to inspect values. The default is `/var/data` which is not allowed on my machine.

That's it for today.

Remember to change `BINANCE_` to `BNB_` in my cluster repo!

### 02.01.2023 Thursday 6h

> fail to get keyring keybase error="fail to get signer info(thornode1): thornode1.info: key not found

Don't have a keyring for the cosmos sdk.  
What generates that again?  

Okay can I mount my `thornode` and `bifrost` binaries directly inside the container in order to - ah still won't be able to run in debug mode.

Well, looks like my `thornode-start.sh` script still works.  
Bifrost too, with the same error.  

Okay NICE. TanB are stable running on my bare machine metal.  
Until they crash, but that's ok.  
Now with docker compose?  

> 2023-02-02 09:32:39 {"level":"fatal","service":"bifrost","chain":"BNB","time":"2023-02-02T17:32:39Z","caller":"/app/cmd/bifrost/main.go:160","message":"missing chain RPC host"}

I'm noticing the `runner` and `builder` are using `amd64` rather than `arm64`. Can I fix that?

```
cd ci
make builder
# make runner
```

`make runner` broke because they've specified every single apk package version and they're outdated and cannot be found.

Run this instead:
```bash
DOCKER_BUILDKIT=0 docker build . \
  -t registry.gitlab.com/thorchain/thornode:runner-base-v1 \
  -f-<<EOF
FROM alpine:3
RUN apk add --no-cache \
    bind-tools \
    curl \
    gcc \
    jq \
    libffi-dev \
    musl-dev \
    openssl-dev \
    protoc \
    py3-pip \
    python3-dev \
    && pip3 install --no-cache-dir requests==2.22.0 web3==5.29.0
EOF
```

Now you need to remove the `@sha` in `build/docker/Dockerfile` for:
- `runner-base-v1@`
- `builder-v3@`

Better. Much faster.

```
2023-02-02 10:25:46 {"level":"fatal","service":"bifrost","module":"bifrost","error":"fail to create block scanner: Post \"http://dash2:19898\": dial tcp 172.32.62.2:19898: connect: connection refused","chain_id":"DASH","time":"2023-02-02T18:25:46Z","caller":"/app/bifrost/pkg/chainclients/loadchains.go:97","message":"fail to load chain"}
```

Ah `dash2` was down with this error:

```
2023-02-02 10:20:47 Assertion failure:
2023-02-02 10:20:47   assertion: size_t(attempt) < quorum->members.size()
2023-02-02 10:20:47   file: llmq/signing_shares.cpp, line: 805
2023-02-02 10:20:47   function: static CDeterministicMNCPtr llmq::CSigSharesManager::SelectMemberForRecovery(const CQuorumCPtr&, const uint256&, size_t)
2023-02-02 10:20:47 No debug information available for stacktrace. You should add debug information and then run:
2023-02-02 10:20:47 dashd -printcrashinfo=bvcgc43iinzgc43ijfxgm3ybaacwiyltnbspqqltonsxe5djn5xcaztbnfwhk4tfhifcaidbonzwk4tunfxw4oraonuxuzk7oqugc5dumvwxa5bjea6ca4lvn5zhk3jnhzwwk3lcmvzhglttnf5gkkbjbiqcaztjnrstuidmnrwxcl3tnftw42lom5pxg2dbojsxgltdobycyidmnfxgkorahaydkcraebthk3tdoruw63r2ebzxiylunfrsaq2emv2gk4tnnfxgs43unfru2tsdkb2heidmnrwxcor2injwsz2tnbqxezltjvqw4ylhmvzduostmvwgky3ujvsw2ytfojdg64ssmvrw65tfoj4sqy3pnzzxiicdkf2w64tvnvbva5dseywcay3pnzzxiidvnfxhimrvgytcyidtnf5gkx3ufeeqyvboa6v2v777lsb5gbvlvl7763ef2mdkxkx775qcbvagvovp772yj7kank5k777wzfgqa2v2v777wcs5abvlvl777pfumad2xkx777cp76x7777777ya
2023-02-02 10:20:47 dashd: llmq/signing_shares.cpp:805: static CDeterministicMNCPtr llmq::CSigSharesManager::SelectMemberForRecovery(const CQuorumCPtr&, const uint256&, size_t): Assertion `size_t(attempt) < quorum->members.size()' failed.
```

```log
2023-02-02 12:55:32 {"level":"fatal","service":"bifrost","module":"bifrost","error":"fail to register (tthorpub1addwnpepq2g0zq25vw3d65qua24hm96a5ah5wn5vs3rzvdawmaup6e2v3p775y53g5k): -32601: Method not found","chain_id":"DASH","time":"2023-02-02T20:55:32Z","caller":"/app/bifrost/pkg/chainclients/loadchains.go:97","message":"fail to load chain"}
```

```bash
$ dash-cli importaddress 
error code: -32601
error message:
Method not found
```

Right. So the issue is, `importaddress` is part of the wallet api which is disabled for masternodes (to protect the collateral?).

I'll just point to `dash1` - don't see the harm, the quorum health will have been verified as part of the startup script.

Smoking...

Can't connect?  
`bifrost` port `6040` somehow wasn't exposed. Not sure how that ever worked!?

http://localhost:6040/p2pid  
http://localhost:1317/thorchain/inbound_addresses  
http://localhost:1317/thorchain/lastblock  

This is quite useful to alert when mocknet ready for smoke tests:
```bash
timeout 300 bash -c 'while [[ "$(curl --insecure -s -o /dev/null -w ''%{http_code}'' http://localhost:6040/p2pid)" != "200" ]]; do sleep 5; done' && afplay /System/Library/Sounds/Funk.aiff
```

Smoke round 2...

```bash
docker run \
  -it \
  -w /app \
  -v $(pwd)/test/smoke:/app \
  --rm \
  --entrypoint bash \
  --network host \
  registry.gitlab.com/thorchain/thornode:smoke
  
# pip install -r requirements.txt
python scripts/smoke.py --bootstrap-only=True
```

Connection refused, `/wallet`.  
What's that then?  

Shit. Realised something critical and unfortunate at this stage.  
I'm going to need to run every single goddamn node anyway.  
Without each node the smoke test will not be configured correctly.  

Okay I'm going to bruteforce my way into making the makefile work for me.

- `node-launcher`: F&R `x86_64` -> `aarch64`
- `node-launcher/ci/images/build.sh`: `find` -> `gfind`
- `node-launcher/ci/images/build.sh`: Comment out all the docker hub stuff
- Update bch sha
- linux-headers-amd64 -> linux-headers-arm64
- butcher litecoin to work using uphold's dockerfile
- Run that `build.sh`
- Prevent image pull, update `docker-compose` for ci images
- Build mocknet

```bash
DOCKER_BUILDKIT=0 make build-mocknet

make run-mocknet
# AKA
docker compose \
  -f build/docker/docker-compose.yml \
  --profile mocknet \
  --profile midgard \
  up -d

make bootstrap-mocknet

make stop-mocknet
```

Geting on `run-mocknet`:  
> Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "/scripts/entrypoint-mock.sh": stat /scripts/entrypoint-mock.sh: no such file or directory: unknown

bifrost and thornode are still amd64 for some reason??

It's a bit spasm-y without disabling the buildkit.

Dash image doesn't have entrypoint scripts. Oh?

TODO: FIX `Chart.lock`. IT WILL BE WRONG.

Dash config file location changed.  
There's something up with bitcoincash.  

> boost::filesystem::create_directory: Permission denied: "/home/bitcoin/.bitcoin/regtest"

Fix by creating the directory first in the dockerfile as the bitcoin user so that mounted volumes inherit the container directory permissions, allowing the bitcoin user to read/write to the `/home/bitcoin/.bitcoin` directory.

litecoin is seemingly broken too:  
> 2023-02-02 15:21:52 /scripts/entrypoint-mock.sh: line 14: litecoin-cli: not found

```bash
/opt/litecoin $ find / -name 'litecoin-cli'
/usr/local/bin/litecoin-cli

/opt/litecoin $ litecoin-cli
/bin/sh: litecoin-cli: not found

/opt/litecoin $ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

/opt/litecoin $ /usr/local/bin/litecoin-cli
/bin/sh: /usr/local/bin/litecoin-cli: not found
```

Wow. How the hell does that make sense?

```
docker run --rm -it registry.gitlab.com/thorchain/devops/node-launcher:litecoin-daemon-0.21.2 bash

apk add binutils
readelf -a /usr/local/bin/litecoin-cli
apk add libc6-compat
apk add alpine-sdk

apk add gcompat
```

Nope.

Just going to use uphold's dockerfile here, alpine is sketchy on my machine and it's not worth spending the time finding out what I need to install.

https://github.com/uphold/docker-litecoin-core/blob/master/0.21/Dockerfile

> 2023-02-02 16:08:12 12:08AM FTL app/bifrost/pkg/chainclients/loadchains.go:74 > fail to load chain error="fail to register (tthorpub1addwnpepq0djpgp2rhhelx0cckvxpd4k4c8xj2z8l726jlxrrra46zhlg28ws3nfewp): -18: No wallet is loaded. Load a wallet using loadwallet or create a new one with createwallet. (Note: A default wallet is no longer automatically created)" chain_id=LTC module=bifrost service=bifrost

more litecoin issues...

This did the trick:
```
litecoin-cli -regtest -rpcuser=thorchain -rpcpassword=password -rpcport=38443 createwallet ""
```

> Cannot transfer. No LTC UTXO available for rltc1q0s4mg25tu6termrk8egltfyme4q7sg3hemvd64

litecoin again.

```
litecoin-cli -regtest -rpcuser=thorchain -rpcpassword=password -rpcport=38443 generatetoaddress 1 rltc1qj08ys4ct2hzzc2hcz6h2hgrvlmsjynawf4nr3r
```

Not wanting to solve this one either.  
Going to try rolling back to the old version.  
  
Well, the bootstrap inexplicably stopped early with no error.  
I'm not convinced that worked, was expecting more than 100 and it got to 56.  

```
I[2023-02-03 00:28:37,442] 56 PROVIDER-1 => VAULT      [ADD:] 0.10000000 BNB.BNB
```

Is the add supposed the be empty?


```go
2023-02-02 16:27:38 12:27AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=300 chain=LTC module=blockscanner service=bifrost txs=0
2023-02-02 16:27:46 12:27AM INF app/bifrost/pkg/chainclients/litecoin/client.go:1084 > totalTxValue:150000000,total fee and Subsidy:1250100000,confirmation:0 module=litecoin service=bifrost
2023-02-02 16:27:46 12:27AM INF app/bifrost/pkg/chainclients/litecoin/client.go:1099 > confirmation required: 0 module=litecoin service=bifrost
2023-02-02 16:27:46 12:27AM INF app/bifrost/pkg/chainclients/litecoin/client.go:1119 > confirmation required: 0 module=litecoin service=bifrost
2023-02-02 16:27:46 12:27AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=17 service=bifrost
2023-02-02 16:27:46 12:27AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of F7585CB8CAECD558CFCCA2622C0A018B17142E014CA7BB7CE68777AA16C24BFF from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:27:46 12:27AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=F7585CB8CAECD558CFCCA2622C0A018B17142E014CA7BB7CE68777AA16C24BFF
2023-02-02 16:27:50 12:27AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=800 chain=BNB module=blockscanner service=bifrost txs=0
2023-02-02 16:28:01 12:28AM INF app/bifrost/pkg/chainclients/ethereum/smartcontract_log_parser.go:187 > deposit:{To:0x3a8f6CE730bD517Ec00c636C1572677ce02d748f Asset:0x0000000000000000000000000000000000000000 Amount:+400000000000000000 Memo:ADD:ETH.ETH:tthor1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6zdp257} module=SmartContractLogParser service=bifrost
2023-02-02 16:28:01 12:28AM INF app/bifrost/pkg/chainclients/ethereum/smartcontract_log_parser.go:209 > token:0x0000000000000000000000000000000000000000,decimals:0 module=SmartContractLogParser service=bifrost
2023-02-02 16:28:01 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go:785 > tx: 44f7c5f59916dc30df2865c2b4cbc33a72d560eb888d85a6262d76b8a2c1627b, gas price: 2658499, gas used: 63862,receipt status:1 chain=ETH module=block_scanner service=bifrost
2023-02-02 16:28:06 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum.go:815 > totalTxValue:400000000000000000,total fee and Subsidy:3000000000000000000,confirmation:0 module=ethereum service=bifrost
2023-02-02 16:28:06 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum.go:750 > confirmation required: 0 module=ethereum service=bifrost
2023-02-02 16:28:06 12:28AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=18 service=bifrost
2023-02-02 16:28:06 12:28AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of 73D85CECC605A396596C34315DABE83EA87C783B80CA798FBF48BDEE5B684D02 from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:28:06 12:28AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=73D85CECC605A396596C34315DABE83EA87C783B80CA798FBF48BDEE5B684D02
2023-02-02 16:28:16 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go:385 > no tx need to be processed in this block block=54 chain=ETH module=block_scanner service=bifrost
2023-02-02 16:28:20 12:28AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=900 chain=BNB module=blockscanner service=bifrost txs=0
2023-02-02 16:28:21 12:28AM INF app/bifrost/pkg/chainclients/ethereum/smartcontract_log_parser.go:187 > deposit:{To:0x3a8f6CE730bD517Ec00c636C1572677ce02d748f Asset:0x40bcd4dB8889a8Bf0b1391d0c819dcd9627f9d0a Amount:+4000000000000000000 Memo:ADD:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:tthor1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6zdp257} module=SmartContractLogParser service=bifrost
2023-02-02 16:28:21 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go:675 > token:0x40bcd4dB8889a8Bf0b1391d0c819dcd9627f9d0a, decimals: 18 chain=ETH module=block_scanner service=bifrost
2023-02-02 16:28:21 12:28AM INF app/bifrost/pkg/chainclients/ethereum/smartcontract_log_parser.go:209 > token:0x40bcd4dB8889a8Bf0b1391d0c819dcd9627f9d0a,decimals:0 module=SmartContractLogParser service=bifrost
2023-02-02 16:28:21 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum_block_scanner.go:785 > tx: e4f9fecfad95c15e4b768f422f6020fc09e191ac9cf44924b8ca5c943b5c659e, gas price: 1560719, gas used: 92813,receipt status:1 chain=ETH module=block_scanner service=bifrost
2023-02-02 16:28:26 12:28AM ERR app/bifrost/pkg/chainclients/ethereum/ethereum.go:789 > fail to get value for ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A error="pool:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A doesn't exist" module=ethereum service=bifrost
2023-02-02 16:28:26 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum.go:815 > totalTxValue:0,total fee and Subsidy:3000000000000000000,confirmation:0 module=ethereum service=bifrost
2023-02-02 16:28:26 12:28AM INF app/bifrost/pkg/chainclients/ethereum/ethereum.go:750 > confirmation required: 0 module=ethereum service=bifrost
2023-02-02 16:28:26 12:28AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=19 service=bifrost
2023-02-02 16:28:26 12:28AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of 9B92C0A8D50A6D9A1758B14434EBCA2EE889E083DD793301C8F19BA5A16C42A2 from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:28:26 12:28AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=9B92C0A8D50A6D9A1758B14434EBCA2EE889E083DD793301C8F19BA5A16C42A2
2023-02-02 16:28:31 12:28AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=20 service=bifrost
2023-02-02 16:28:31 12:28AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of F064C083AC97841DCE18771A1822B16DDED679A21B805D5CCCB482F18755E9F1 from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:28:31 12:28AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=F064C083AC97841DCE18771A1822B16DDED679A21B805D5CCCB482F18755E9F1
2023-02-02 16:28:41 12:28AM ERR app/bifrost/observer/observe.go:312 > Unable to parse memo: ADD: error="invalid symbol" module=observer service=bifrost
2023-02-02 16:28:41 12:28AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=21 service=bifrost
2023-02-02 16:28:41 12:28AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of 0B90BBF856442317AE72CFCCFC21CC988CE340F35722CAC591D88ECE4CC62F15 from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:28:41 12:28AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=0B90BBF856442317AE72CFCCFC21CC988CE340F35722CAC591D88ECE4CC62F15
2023-02-02 16:28:41 12:28AM ERR app/bifrost/pkg/chainclients/binance/binance.go:627 > fail to parse memo: ADD: error="invalid symbol" module=binance service=bifrost
2023-02-02 16:28:47 12:28AM INF app/bifrost/signer/sign.go:258 > Received a TxOut Array of {47 [{BNB tbnb1mkymsmnqenxthlmaa9f60kd6wgr9yjy9h5mz6q tthorpub1addwnpepqfh3x4l86cjsadpn7rkdwc2n9hqulk79xetejypggv9zevqrpfnazvr4mge 9887500 BNB.BNB REFUND:BA3FFA565E280076DCDFE9E948A649AA2A4D60F9C30758044C6A5BE707381903 [37500 BNB.BNB] 56250 BA3FFA565E280076DCDFE9E948A649AA2A4D60F9C30758044C6A5BE707381903    <nil>}]} from the Thorchain module=signer service=bifrost
2023-02-02 16:28:47 12:28AM INF app/bifrost/signer/sign.go:210 > Signing transaction height=47 module=signer num=0 service=bifrost status=1 tx={"aggregator":"","chain":"BNB","coins":[{"amount":"9887500","asset":"BNB.BNB"}],"gas_rate":56250,"in_hash":"BA3FFA565E280076DCDFE9E948A649AA2A4D60F9C30758044C6A5BE707381903","max_gas":[{"amount":"37500","asset":"BNB.BNB","decimals":8}],"memo":"REFUND:BA3FFA565E280076DCDFE9E948A649AA2A4D60F9C30758044C6A5BE707381903","out_hash":"","to":"tbnb1mkymsmnqenxthlmaa9f60kd6wgr9yjy9h5mz6q","vault_pubkey":"tthorpub1addwnpepqfh3x4l86cjsadpn7rkdwc2n9hqulk79xetejypggv9zevqrpfnazvr4mge"}
2023-02-02 16:28:47 12:28AM INF app/bifrost/pkg/chainclients/binance/binance.go:369 > account info account_number=3 block height=993 module=binance sequence_number=0 service=bifrost
2023-02-02 16:28:47 12:28AM INF app/bifrost/pkg/chainclients/binance/binance.go:539 > broadcast response from Binance Chain,memo:REFUND:BA3FFA565E280076DCDFE9E948A649AA2A4D60F9C30758044C6A5BE707381903 body="{\"hash\":\"BJTUMF66UWDI6MSH4TQF5WO3WTDPB6O71YK07CGA1O67RGSRWMGYA74KBCTRABAX\",\"height\":\"994\",\"logs\":[{\"success\":true,\"log\":\"OK\"}]}" module=binance service=bifrost
2023-02-02 16:28:49 12:28AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=1000 chain=BNB module=blockscanner service=bifrost txs=0
2023-02-02 16:28:51 12:28AM INF app/bifrost/thorclient/broadcast.go:46 > account info account_number=0 module=thorchain_client sequence_number=22 service=bifrost
2023-02-02 16:28:51 12:28AM INF app/bifrost/thorclient/broadcast.go:100 > Received a TxHash of 38F422D74DE7A4B5E01708E30A4C08349769C0469AC5056145152B234AE7B20C from the thorchain module=thorchain_client service=bifrost
2023-02-02 16:28:51 12:28AM INF app/bifrost/observer/observe.go:484 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=38F422D74DE7A4B5E01708E30A4C08349769C0469AC5056145152B234AE7B20C
2023-02-02 16:29:00 12:29AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=400 chain=BCH module=blockscanner service=bifrost txs=0
2023-02-02 16:29:00 12:29AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=400 chain=BTC module=blockscanner service=bifrost txs=0
2023-02-02 16:29:03 12:29AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=1300 chain=DOGE module=blockscanner service=bifrost txs=0
2023-02-02 16:29:03 12:29AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=300 chain=DASH module=blockscanner service=bifrost txs=0
2023-02-02 16:29:19 12:29AM INF app/bifrost/blockscanner/blockscanner.go:237 > scan block block height=1100 chain=BNB module=blockscanner service=bifrost txs=0
```

Just re-ran the update balance/events command, forgot to do that:

```
EXPORT=data/smoke_test_balances.json EXPORT_EVENTS=data/smoke_test_events.json make smoke-unit-test
```

Should be 114 transcations.

The problem with the `make smoke` command is that it restarts the cluster (which I have lovingly ensured is working and is ready) and pulls the smoke image from the thornode registry. Local images please guys.

```bash
docker run \
  --network=host \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(PWD)/test/smoke:/app \
    registry.gitlab.com/thorchain/thornode:smoke \
    python scripts/smoke.py --fast-fail=True
```

16:40  
Letting timer overrun for these tests...  

> I[2023-02-03 00:42:33,656] 39 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 00:42:38,045] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC

Well that's that for today.

I don't think I actually ran the smoke tests over on the `master` branch to
check I'm starting off on the right foot. Do that tomorrow!

### 03.02.2023 Friday 6h

Checked out `release-1.101.0`.  
Clean docker environment.  
Patched `Makefile` (to ignore requirement for GNU find/grep etc.)  
Patched `build/Dockerfile` to use local build/runner.  
Patched `config.go` - probs not necessary  

Build thornode/bifrost images.  
Run mocknet / docker compose up.  
Wait for `localhost:6040/p2pid`.  
Run smoke tests.  

```
I[2023-02-03 18:28:38,575] 36 PROVIDER-1 => VAULT      [SWAP:BTC.BTC:PROVIDER-1] 0.10000000 BTC/BTC
E[2023-02-03 18:28:57,164] out coins not matching 6946933 BTC.BTC != 8383225 BTC.BTC
E[2023-02-03 18:29:42,457] Events mismatch
```

Devastating. Let me try again.

```
I[2023-02-03 18:43:00,044] 35 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 18:43:04,143] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC
E[2023-02-03 18:44:04,196] Events mismatch
```

Hmmmmm. I get the feeling these smoke tests are not particularly strong.  
I ran the events regeneration too, no changes.

Can I make them try a little harder?  
I do have a sneaky suspicion it's because my machine is having to emulate amd on arm and that's causing performance issues which are introducing these test inconsistencies.

Options:  
1. Rebuild these for `arm64/aarch64`:
```
registry.gitlab.com/thorchain/midgard:develop
registry.gitlab.com/thorchain/devops/node-launcher:gaia-daemon-7.0.3
ethereum/client-go:v1.10.25
registry.gitlab.com/thorchain/devops/node-launcher:avalanche-daemon-1.9.0
registry.gitlab.com/thorchain/devops/node-launcher:litecoin-daemon-0.18.1
registry.gitlab.com/thorchain/bepswap/mock-binance
registry.gitlab.com/thorchain/devops/node-launcher:bitcoin-daemon-24.0
registry.gitlab.com/thorchain/devops/node-launcher:bitcoin-cash-daemon-24.0.0
registry.gitlab.com/thorchain/devops/node-launcher:dogecoin-daemon-1.14.6
```
2. Try on a `amd64` machine
3. Spin up an entire aws cluster (very sad times, absolute last resort, this really shouldn't be necessary)
4. Improve the tests (probably the best option, but I'm not familiar with python)

```
I[2023-02-03 18:58:05,588] 35 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 18:58:09,672] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC
E[2023-02-03 18:59:09,687] Events mismatch
```

```
I[2023-02-03 19:06:27,393] 35 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 19:06:31,077] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC
E[2023-02-03 19:07:31,132] Events mismatch
```

Pretty consistently wrong. Does make me wonder.

Okay trying with all arm64 images.

```
2023-02-03 11:31:23 bitcoind: error while loading shared libraries: libgcc_s.so.1: cannot open shared object file: No such file or directory
2023-02-03 11:31:23 bitcoin-cli: error while loading shared libraries: libgcc_s.so.1: cannot open shared object file: No such file or directory
```

Original list for target thornode revision: 
registry.gitlab.com/thorchain/devops/node-launcher:avalanche-daemon-1.9.0 
registry.gitlab.com/thorchain/devops/node-launcher:bitcoin-daemon-24.0 
registry.gitlab.com/thorchain/devops/node-launcher:bitcoin-cash-daemon-24.0.0 
registry.gitlab.com/thorchain/devops/node-launcher:dogecoin-daemon-1.14.6 
registry.gitlab.com/thorchain/devops/node-launcher:gaia-daemon-7.0.3 
registry.gitlab.com/thorchain/devops/node-launcher:litecoin-daemon-0.18.1 

I've had to update bitcoin cash now as I was having the alpine arm issues.  
Let me guess, I'll have the wallet errors again now?  
Oh directory not writable for bch first.  
Now?  

Actually seems ok.  
Sus.  

```
I[2023-02-03 20:07:53,611] 35 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 20:07:57,386] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC
```

Getting exatly the same issue. Both `arm64` and `amd64`, the reason I wasn't sure this was a legit issue is because I got to tx `36` with the first test I ran today.

This means somewhere the fee calculation for BTC is wrong?  
Or the test files aren't being updated properly?

```
rm ./test/smoke/data/smoke_test_balances.json ./test/smoke/data/smoke_test_events.json
```

```
EXPORT=data/smoke_test_balances.json \
EXPORT_EVENTS=data/smoke_test_events.json \
make smoke-unit-test
```

The above is equivalent to:

```bash
docker run \
  --network=host \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(PWD)/test/smoke:/app \
  -e EXPORT=data/smoke_test_balances.json \
  -e EXPORT_EVENTS=data/smoke_test_events.json \
  registry.gitlab.com/thorchain/thornode:smoke \
  sh -c 'python -m unittest tests/test_*'
```

Running that gives me:
```
ERROR: test_smoke (tests.test_smoke.TestSmoke)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/app/tests/test_smoke.py", line 184, in test_smoke
    expected = get_balance(i)  # get the expected balance from json file
  File "/app/tests/test_smoke.py", line 33, in get_balance
    with open(file) as f:
FileNotFoundError: [Errno 2] No such file or directory: 'data/smoke_test_balances.json'
```

Isn't this supposed to be generating that file though?  
Do I bootstrap first?

```
docker run \
  --network=host \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(PWD)/test/smoke:/app \
  -e EXPORT=data/smoke_test_balances.json \
  -e EXPORT_EVENTS=data/smoke_test_events.json \
  registry.gitlab.com/thorchain/thornode:smoke \
  python scripts/smoke.py --bootstrap-only=True
```

```
I[2023-02-03 20:27:54,394]  2     MASTER => PROVIDER-1 [SEED] 10.00000000 BNB.BNB, 800.00000000 BNB.LOK-3C0
I[2023-02-03 20:27:54,406]  5     MASTER => PROVIDER-1 [SEED] 5.00000000 BTC.BTC
I[2023-02-03 20:27:54,430]  7     MASTER => PROVIDER-1 [SEED] 2,000.00000000 DOGE.DOGE
I[2023-02-03 20:27:54,452]  9     MASTER => PROVIDER-1 [SEED] 500,000.00000000 GAIA.ATOM
I[2023-02-03 20:27:58,926] 12     MASTER => PROVIDER-1 [SEED] 2.00000000 BCH.BCH
I[2023-02-03 20:27:58,962] 15     MASTER => PROVIDER-1 [SEED] 2.00000000 LTC.LTC
I[2023-02-03 20:27:58,986] 18     MASTER => PROVIDER-1 [SEED] 400,000,000,000.00000000 ETH.ETH
I[2023-02-03 20:28:04,038] 20     MASTER => PROVIDER-1 [SEED] 4,000,000,000,000.00000000 ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A
I[2023-02-03 20:28:09,021] 22 PROVIDER-1 => VAULT      [ADD:BNB.BNB:PROVIDER-1] 1,000.00000000 THOR.RUNE
I[2023-02-03 20:28:10,211] 23 PROVIDER-1 => VAULT      [ADD:BNB.BNB:PROVIDER-1] 2.50000000 BNB.BNB
I[2023-02-03 20:28:15,008] 24 PROVIDER-1 => VAULT      [ADD:DOGE.DOGE:PROVIDER-1] 10.00000000 THOR.RUNE
I[2023-02-03 20:28:20,207] 25 PROVIDER-1 => VAULT      [ADD:DOGE.DOGE:PROVIDER-1] 1,500.00000000 DOGE.DOGE
I[2023-02-03 20:28:30,140] 26 PROVIDER-1 => VAULT      [ADD:GAIA.ATOM:PROVIDER-1] 10.00000000 THOR.RUNE
I[2023-02-03 20:28:35,235] 27 PROVIDER-1 => VAULT      [ADD:GAIA.ATOM:PROVIDER-1] 1,500.00000000 GAIA.ATOM
I[2023-02-03 20:28:50,085] 33 PROVIDER-1 => VAULT      [ADD:BTC.BTC:PROVIDER-1] 1,000.00000000 THOR.RUNE
I[2023-02-03 20:28:55,294] 34 PROVIDER-1 => VAULT      [ADD:BTC.BTC:PROVIDER-1] 2.50000000 BTC.BTC
I[2023-02-03 20:29:05,284] 41 PROVIDER-1 => VAULT      [ADD:BCH.BCH:PROVIDER-1] 500.00000000 THOR.RUNE
I[2023-02-03 20:29:10,311] 42 PROVIDER-1 => VAULT      [ADD:BCH.BCH:PROVIDER-1] 1.50000000 BCH.BCH
I[2023-02-03 20:29:20,318] 43 PROVIDER-1 => VAULT      [ADD:LTC.LTC:PROVIDER-1] 500.00000000 THOR.RUNE
I[2023-02-03 20:29:25,042] 44 PROVIDER-1 => VAULT      [ADD:LTC.LTC:PROVIDER-1] 1.50000000 LTC.LTC
I[2023-02-03 20:29:35,073] 45 PROVIDER-1 => VAULT      [ADD:ETH.ETH:PROVIDER-1] 500.00000000 THOR.RUNE
I[2023-02-03 20:29:40,369] 46 PROVIDER-1 => VAULT      [ADD:ETH.ETH:PROVIDER-1] 4,000,000,000.00000000 ETH.ETH
I[2023-02-03 20:29:55,276] 47 PROVIDER-1 => VAULT      [ADD:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:PROVIDER-1] 500.00000000 THOR.RUNE
I[2023-02-03 20:30:00,386] 48 PROVIDER-1 => VAULT      [ADD:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:PROVIDER-1] 40,000,000,000.00000000 ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A
I[2023-02-03 20:30:20,302] 49 PROVIDER-1 => VAULT      [ADD:BNB.LOK-3C0:PROVIDER-1] 400.00000000 BNB.LOK-3C0
I[2023-02-03 20:30:25,445] 50 PROVIDER-1 => VAULT      [ADD:BNB.LOK-3C0:PROVIDER-1] 500.00000000 THOR.RUNE
I[2023-02-03 20:30:30,484] 52 PROVIDER-1 => VAULT      [ADD:] 0.10000000 BNB.BNB
```

Seems abrupt, unit tests?

Running with the `python scripts/smoke.py --generate-balances=True` throws this:

```
I[2023-02-03 20:43:40,689] 35 PROVIDER-1 => VAULT      [SWAP:BTC/BTC:PROVIDER-1-SYNTH] 1.00000000 BNB/BNB
E[2023-02-03 20:43:44,357] out coins not matching 53680387 BTC/BTC != 53678432 BTC/BTC
```

I don't know why that's being returned, we're supposed to be generating the correct values to test against right?

This is a good time to try on my amd/linux box just in case it's something funky with my m1 mac.

Found this in `test_smoke.py`:

> This runs tests with a pre-determined list of transactions and an expected balance after each transaction (/data/balance.json). These transactions and    balances were determined earlier via a google spreadsheet:
https://docs.google.com/spreadsheets/d/1sLK0FE-s6LInWijqKgxAzQk2RiSDZO1GL58kAD62ch0/edit#gid=439437407

I don't remember having to look at that.  
Going through all my old notes thoroughly...  

Apparently `docker compose` is not installed by default on linux.  
https://github.com/docker/compose/tree/v2#linux

```
ls -la /usr/local/lib/docker/cli-plugins;
ls -la /usr/local/libexec/docker/cli-plugins;
ls -la /usr/lib/docker/cli-plugins;
ls -la /usr/libexec/docker/cli-plugins;
```

```
mkdir -p /usr/local/lib/docker/cli-plugins && cd $_
wget https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-x86_64
mv docker-compose-linux-x86_64 docker-compose
chmod +x ./docker-compose
docker compose
```

There we go.

Right fresh machine, clean repo, doing as I'm told:

```bash
make smoke-protob-docker
make smoke
```

```
I[2023-02-04 00:36:55,545] 29 PROVIDER-1 => VAULT      [SWAP:BNB/BNB:PROVIDER-1-SYNTH] 500.00000000 THOR.RUNE
E[2023-02-04 00:38:00,043] Events mismatch

E[2023-02-04 00:38:00,045] Smoke tests failed
Traceback (most recent call last):
  File "/app/scripts/smoke.py", line 136, in main
    smoker.run()
  File "/app/scripts/smoke.py", line 618, in run
    self.check_events(events, sim_events)
  File "/app/scripts/smoke.py", line 356, in check_events
    self.error("Events mismatch\n")
  File "/app/scripts/smoke.py", line 235, in error
    raise Exception(err)
Exception: Events mismatch

>>>>>>>>>>>>>>> MISSING SIM EVENT
Event mint_burn | {'supply': 'mint'} {'denom': 'bnb/bnb'} {'amount': '79996667'} {'reason': 'native_tx_out'}
<<<<<<<<<<<<<<< EXTRA SIM EVENT
Event rewards | {'bond_reward': '697615247'}
make: *** [Makefile:195: smoke] Error 1
```

Got to `29`.  
I'll try again.

Ooooo will it........  
It's holding...  
Cummooooon!  
Damn that's gotta be too long.  
I'm scared.  
Praise be to all the deities.  
Well that went unheard, no dice.  

```
I[2023-02-04 00:44:57,786] 29 PROVIDER-1 => VAULT      [SWAP:BNB/BNB:PROVIDER-1-SYNTH] 500.00000000 THOR.RUNE
>>>>>>>>>>>>>>> MISSING SIM EVENT
Event mint_burn | {'supply': 'mint'} {'denom': 'bnb/bnb'} {'amount': '79996667'} {'reason': 'native_tx_out'}
<<<<<<<<<<<<<<< EXTRA SIM EVENT
Event rewards | {'bond_reward': '697615269'}
E[2023-02-04 00:46:02,599] Events mismatch

E[2023-02-04 00:46:02,599] Smoke tests failed
Traceback (most recent call last):
  File "/app/scripts/smoke.py", line 136, in main
    smoker.run()
  File "/app/scripts/smoke.py", line 618, in run
    self.check_events(events, sim_events)
  File "/app/scripts/smoke.py", line 356, in check_events
    self.error("Events mismatch\n")
  File "/app/scripts/smoke.py", line 235, in error
    raise Exception(err)
Exception: Events mismatch

make: *** [Makefile:195: smoke] Error 1
```

Make that a double.  
Why?

Interesting that my mac actually got further.  
Let's compare.

```
mac
commit         9172986498ea380a616ff599984d23c2c9f16622
binance image  6dc9bff7ad02
thornode       0dfc1f11bd03   6 hours ago

linux
commit         9172986498ea380a616ff599984d23c2c9f16622
binance image  6dc9bff7ad02
thornode       c04638b076b5   2 days ago
```

You know what.  
I was using my own images for thornode/bifrost.  
Let's try that.  

```
DOCKER_BUILDKIT=0 make build-mocknet
```

While that builds, it does seem that it's a very bad thing to delete the balance.json file.

- `test/smoke/data/smoke_test_transactions.json` NOT GENERATED
- `test/smoke/data/smoke_test_balances.json` GENERATED
- `test/smoke/data/smoke_test_events.json` GENERATED

Getting openapi errors.

```
make openapi
```

`83` and going strong!  
This could be the one.  
Amazing, done, `106` reached.  
  
Okay that box it is.  

Building from my branch now with the 101 changes merged in.  
I do wonder if I merged the  


### 07.02.2023 Tuesday 5h

Was running out of space on my linux box so restored a snapshot and increased the disk space.

I always forget that for system configs - such as adding a custom bin directory to the system `$PATH` via `/etc/profile` - to effect root users you have to switch via a login shell `su -`. Switching using something like `sudo -s` does not cut it.

To allow that `su -` command without having to constantly enter the password add the following lines underneath the `pam_rootok.so` line in `/etc/pam.d/su`:

```
auth  [success=ignore default=1] pam_succeed_if.so user = root
auth  sufficient                 pam_succeed_if.so use_uid user = adc
```

Right so I've cloned `thornode`.

Had some readonly docker issues. Had to remove the snap version and install using the official guide.

```
make smoke-protob-docker
make openapi
DOCKER_BUILDKIT=0 make build-mocknet
DOCKER_BUILDKIT=0 make smoke
```

Make smoke pulls a docker image for `thorchain/thornode` despite me having already run `make build-mocknet` which is suspicious.
Could be `thorchain/thornode:smoke`?

Okay smokes are rolling.

One thing to note is, `smoke_test_balances.json` and `smoke_test_events.json` are the same in my dash branch as the develop branch. Which means, they haven't been regenerated. `smoke_test_transactions.json`, the one which requires manual tweaking, *has* been changed in my dash branch. Not sure if the smoke tests will pass with this mismatch, but even if they do my next step is to get those files regenerated.

It got to `114`, no success message. Ends like this:
```
I[2023-02-08 04:10:59,126] 114 PROVIDER-1 => VAULT      [WITHDRAW:LTC.LTC] 0.00000001 THOR.RUNE
I[2023-02-08 04:11:19,501] [+] 486.11319757 THOR.RUNE | Fee 0.02000000 THOR.RUNE | Gas 0.02000000 THOR.RUNE
I[2023-02-08 04:11:19,503] [+] 1.24598427 LTC.LTC | Fee 0.00000000 LTC.LTC | Gas 0.00609308 LTC.LTC
```

Is there ever a success message? It would certainly be more reassuring.

Well the last 2 transactions in the json are withdraw BCH and withdraw LTC.  
So yeah looks good.

Ok regen, the readme tells me this:
```
EXPORT=data/smoke_test_balances.json \
EXPORT_EVENTS=data/smoke_test_events.json \
make smoke-unit-test
```

We have changes.  
Now for another smoking...

Got to `114` on the dash branch.  
Beaut, I think we're done.  

Just need to bring those changes back to my machine and push those changes.

```
git diff > /tmp/tc.patch

scp adc-vm:/tmp/tc.patch ./
git apply
```

Now...?  
Quick unit test `go test -tags mocknet ./...`.  
Pass.  

Yeah I think I'm done.  

https://gitlab.com/alexdcox/thornode/-/tree/982-add-dash-chain  
https://gitlab.com/alexdcox/node-launcher/-/tree/add-dash  

Posted on the discord. That'll do for now.

## Batch Two

```
09.02.2023 Thursday 4h
03.03.2023 Friday 7h
04.03.2023 Saturday 2h
06.03.2023 Monday 6h
08.03.2023 Monday 4h
09.03.2023 Thursday 4h
13.03.2023 Monday 14h
14.03.2023 Tuesday 4h

Total 45h
```

### 09.02.2023 Thursday 4h

`Itzamna` said he tried the following command:

```
docker build -t registry.gitlab.com/mayachain/devops/node-launcher:dash-daemon-0.17.0.3 ci/images/dash-daemon
```

And he got the error:

> The command '/bin/sh -c apt-get install -y jq curl iputils-ping net-tools vim socat' returned a non-zero code: 100

Also, there are lint errors.

`make lint`
```
  version: 1.101.0
all modules verified
Linting handlers file
Handler: x/thorchain/handler_add_liquidity.go... OK
Handler: x/thorchain/handler_ban.go... OK
Handler: x/thorchain/handler_bond.go... OK
Handler: x/thorchain/handler_common_outbound.go... OK
Handler: x/thorchain/handler_consolidate.go... OK
Handler: x/thorchain/handler_deposit.go... OK
Handler: x/thorchain/handler_donate.go... OK
Handler: x/thorchain/handler_errata_tx.go... OK
Handler: x/thorchain/handler_ip_address.go... OK
Handler: x/thorchain/handler_leave.go... OK
Handler: x/thorchain/handler_manage_thorname.go... OK
Handler: x/thorchain/handler_migrate.go... OK
Handler: x/thorchain/handler_mimir.go... OK
Handler: x/thorchain/handler_network_fee.go... OK
Handler: x/thorchain/handler_node_pause_chain.go... OK
Handler: x/thorchain/handler_noop.go... OK
Handler: x/thorchain/handler_observed_txin.go... OK
Handler: x/thorchain/handler_observed_txout.go... OK
Handler: x/thorchain/handler_outbound_tx.go... OK
Handler: x/thorchain/handler_ragnarok.go... OK
Handler: x/thorchain/handler_refund.go... OK
Handler: x/thorchain/handler_reserve_contrib.go... OK
Handler: x/thorchain/handler_send.go... OK
Handler: x/thorchain/handler_set_node_keys.go... OK
Handler: x/thorchain/handler_solvency.go... OK
Handler: x/thorchain/handler_swap.go... OK
Handler: x/thorchain/handler_switch.go... OK
Handler: x/thorchain/handler_tss.go... OK
Handler: x/thorchain/handler_tss_keysign.go... OK
Handler: x/thorchain/handler_unbond.go... OK
Handler: x/thorchain/handler_version.go... OK
Handler: x/thorchain/handler_withdraw.go... OK
Handler: x/thorchain/handler_yggdrasil.go... OK
Linting managers.go file

OK
./scripts/lint-erc20s.bash: line 6: jq: command not found
make: *** [Makefile:110: lint] Error 127
```

`apt install -y jq`

```
...
Linting managers.go file

OK
OK
go: downloading golang.org/x/tools v0.1.8
go: downloading golang.org/x/sys v0.0.0-20220520151302-bc2c85ada10a
go: downloading golang.org/x/xerrors v0.0.0-20220517211312-f3a8303e98df
/root/go/pkg/mod/golang.org/x/tools@v0.1.8/internal/gocommand/vendor.go:17:2: missing go.sum entry for module providing package golang.org/x/mod/semver (imported by golang.org/x/tools/internal/gocommand); to add:
  go get golang.org/x/tools/internal/gocommand@v0.1.8
make: *** [Makefile:111: lint] Error 1
```

`go get golang.org/x/tools/internal/gocommand@v0.1.8`

```
main: internal error: package "errors" without types was imported from "gitlab.com/thorchain/thornode/x/thorchain/keeper/types"
exit status 1
make: *** [Makefile:111: lint] Error 1
```

Not entirely sure how to tackle that.. Trying:

`go mod tidy`

```
gitlab.com/thorchain/thornode/bifrost/pkg/chainclients/bitcoin imports
  github.com/btcsuite/btcd/chaincfg/chainhash loaded from github.com/btcsuite/btcd@v0.22.0-beta,
  but go 1.16 would fail to locate it:
  ambiguous import: found package github.com/btcsuite/btcd/chaincfg/chainhash in multiple modules:
  github.com/btcsuite/btcd v0.22.0-beta (/root/go/pkg/mod/github.com/btcsuite/btcd@v0.22.0-beta/chaincfg/chainhash)
  github.com/btcsuite/btcd/chaincfg/chainhash v1.0.1 (/root/go/pkg/mod/github.com/btcsuite/btcd/chaincfg/chainhash@v1.0.1)

To proceed despite packages unresolved in go 1.16:
  go mod tidy -e
If reproducibility with go 1.16 is not needed:
  go mod tidy -compat=1.17
For other options, see:
  https://golang.org/doc/modules/pruning
```

`go mod tidy -compat=1.17`

Comes from here:  
https://go.googlesource.com/tools/+/refs/heads/master/go/packages/packages.go?autodive=0%2F%2F#1027

What part of the lint is actually failing here:

```
./scripts/lint.sh
go run tools/analyze/main.go ./common/... ./constants/... ./x/...
./scripts/trunk check --no-fix --upstream origin/develop
```

It's the analyze tool.  
Searching:  
"go tools analyze errors imported without type"  

https://github.com/golang/go/issues/37617

```
$ go get golang.org/x/tools/go/analysis
go: downloading golang.org/x/tools v0.5.0
go: downloading golang.org/x/mod v0.7.0
go: downloading golang.org/x/net v0.5.0
go: downloading golang.org/x/sync v0.1.0
go: downloading golang.org/x/sys v0.4.0
go: downloading golang.org/x/term v0.4.0
go: downloading golang.org/x/text v0.6.0
go: upgraded golang.org/x/mod v0.6.0-dev.0.20211013180041-c96bc1413d57 => v0.7.0
go: upgraded golang.org/x/net v0.0.0-20220607020251-c690dde0001d => v0.5.0
go: upgraded golang.org/x/sync v0.0.0-20210220032951-036812b2e83c => v0.1.0
go: upgraded golang.org/x/sys v0.0.0-20220520151302-bc2c85ada10a => v0.4.0
go: upgraded golang.org/x/term v0.0.0-20210927222741-03fcf44c2211 => v0.4.0
go: upgraded golang.org/x/text v0.3.7 => v0.6.0
go: upgraded golang.org/x/tools v0.1.8 => v0.5.0
```

```
main: 3879 errors during loading
exit status 1
make: *** [Makefile:111: lint] Error 1
```

Wow. Must be proto.

`protob-docker`

Oops ran a `make test`, may as well let it run to double check.  
That went well, back to linting.  

```
  ISSUES

bifrost/pkg/chainclients/dash/client.go:241:39
 241:39  high  captLocal: `MaximumConfirm' should not be capitalized  golangci-lint/gocritic
 389:18  high  func `(*Client).confirmTx` is unused                   golangci-lint/unused
 738:30  high  G601: Implicit memory aliasing in for loop.            golangci-lint/gosec

bifrost/pkg/chainclients/dash/signer.go:144:4
 144:4  high  assignOp: replace `toSpend = toSpend + item.Amount` with `toSpend += item.Amount`  golangci-lint/gocritic

bifrost/pkg/chainclients/dash/signer_test.go:83:12
 83:12  high  ineffectual assignment to err  golangci-lint/ineffassign

build/scripts/node-status.sh:30:18
 30:18  high  ${$x} is invalid. For expansion, use ${x}. For indirection, use arrays, ${!x} or (for sh) eval.  shellcheck/SC2298
 30:19  high  $ cannot be followed by a word                                                                   shfmt/parse

✖ 7 new blocking issues
Run trunk upgrade to upgrade trunk and 10 linters
make: *** [Makefile:112: lint] Error 1
```

`./scripts/trunk check --no-fix --upstream origin/develop`

Sorted.

Back to checking out the docker image:

Commit `3705ce5` of branch `add-dash` of repo `node-launcher`:

```
docker system prune -a

docker build \
  -t registry.gitlab.com/mayachain/devops/node-launcher:dash-daemon-0.17.0.3 \
  ci/images/dash-daemon
```

```
[+] Building 57.9s (10/10) FINISHED
 => [internal] load build definition from Dockerfile                                                                                                                                                                                          0.1s
 => => transferring dockerfile: 322B                                                                                                                                                                                                          0.0s
 => [internal] load .dockerignore                                                                                                                                                                                                             0.1s
 => => transferring context: 2B                                                                                                                                                                                                               0.0s
 => [internal] load metadata for docker.io/dashpay/dashd:0.17.0.3                                                                                                                                                                             1.8s
 => [auth] dashpay/dashd:pull token for registry-1.docker.io                                                                                                                                                                                  0.0s
 => [1/4] FROM docker.io/dashpay/dashd:0.17.0.3@sha256:ad73808606e69722dd2b3d99b98d39952d2d6f344929295fe43f481a33990649                                                                                                                       6.9s
 => => resolve docker.io/dashpay/dashd:0.17.0.3@sha256:ad73808606e69722dd2b3d99b98d39952d2d6f344929295fe43f481a33990649                                                                                                                       0.1s
 => => sha256:889a7173dcfeb409f9d88054a97ab2445f5a799a823f719a5573365ee3662b6f 189B / 189B                                                                                                                                                    0.6s
 => => sha256:3b53ef3ed25adbeac39a14223f3fa89e2c5c8d6a9680ae068c2d13e2478e10a0 6.86kB / 6.86kB                                                                                                                                                0.0s
 => => sha256:75585a48f11ebe18a2ed7d9dc13bbbaf19e5567f9fe6cd6477bba8dcd8d6641c 1.78kB / 1.78kB                                                                                                                                                0.0s
 => => sha256:4bbfd2c87b7524455f144a03bf387c88b6d4200e5e0df9139a9d5e79110f89ca 26.70MB / 26.70MB                                                                                                                                              2.1s
 => => sha256:d2e110be24e168b42c1a2ddbc4a476a217b73cccdba69cdcb212b812a88f5726 857B / 857B                                                                                                                                                    0.4s
 => => sha256:ad73808606e69722dd2b3d99b98d39952d2d6f344929295fe43f481a33990649 1.08kB / 1.08kB                                                                                                                                                0.0s
 => => sha256:b9bdfcf7f53afda8faa48246303f2e46ebb040b131e425b20eb21412780b7dc1 4.52kB / 4.52kB                                                                                                                                                0.7s
 => => sha256:8b21b36ea40d113912d347f3d1e0b526b99e672a4e3bf10221e769b1ac5f8aa1 2.96MB / 2.96MB                                                                                                                                                2.1s
 => => sha256:4c54e37c7b41ce875e1fe4618f6e6a9b11df8103aa41d82035437c40baa0d070 38.69MB / 38.69MB                                                                                                                                              4.8s
 => => sha256:efabff4cd6928cdaa853eea4ef4063b27d61ef84459e9cefd1714ee1d66d86cb 464B / 464B                                                                                                                                                    2.5s
 => => extracting sha256:4bbfd2c87b7524455f144a03bf387c88b6d4200e5e0df9139a9d5e79110f89ca                                                                                                                                                     2.6s
 => => extracting sha256:d2e110be24e168b42c1a2ddbc4a476a217b73cccdba69cdcb212b812a88f5726                                                                                                                                                     0.0s
 => => extracting sha256:889a7173dcfeb409f9d88054a97ab2445f5a799a823f719a5573365ee3662b6f                                                                                                                                                     0.0s
 => => extracting sha256:b9bdfcf7f53afda8faa48246303f2e46ebb040b131e425b20eb21412780b7dc1                                                                                                                                                     0.0s
 => => extracting sha256:8b21b36ea40d113912d347f3d1e0b526b99e672a4e3bf10221e769b1ac5f8aa1                                                                                                                                                     0.2s
 => => extracting sha256:4c54e37c7b41ce875e1fe4618f6e6a9b11df8103aa41d82035437c40baa0d070                                                                                                                                                     1.0s
 => => extracting sha256:efabff4cd6928cdaa853eea4ef4063b27d61ef84459e9cefd1714ee1d66d86cb                                                                                                                                                     0.0s
 => [internal] load build context                                                                                                                                                                                                             0.1s
 => => transferring context: 1.88kB                                                                                                                                                                                                           0.0s
 => [2/4] RUN apt-get update -y && apt-get upgrade -y                                                                                                                                                                                        34.4s
 => [3/4] RUN apt-get install -y jq curl iputils-ping net-tools vim socat                                                                                                                                                                    13.3s
 => [4/4] COPY ./scripts /scripts                                                                                                                                                                                                             0.3s
 => exporting to image                                                                                                                                                                                                                        1.1s
 => => exporting layers                                                                                                                                                                                                                       1.1s
 => => writing image sha256:99103aaea5a38ba039fb57f0cda4ea22aa217ef77e9133754570020680a32794                                                                                                                                                  0.0s
 => => naming to registry.gitlab.com/mayachain/devops/node-launcher:dash-daemon-0.17.0.3
```

NOTE: The host machine requires `jq` to run lint scripts  
NOTE: The host machine requires protobuf complier for `./scripts/protocgen.sh`

Oh. Now we're miracuously getting an error on tx `30`.

```
1:10AM INF app/x/thorchain/handler_deposit.go:60 > receive MsgDeposit coins=[{"amount":"1000000000","asset":"THOR.RUNE"}] from=tthor1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6zdp257 memo=ADD:DASH.DASH:yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L
1:10AM ERR app/x/thorchain/handler.go:246 > fail to parse memo error="yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L is not recognizable"
1:10AM ERR app/x/thorchain/handler_deposit.go:169 > fail to process native inbound tx error="yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L is not recognizable" tx hash=207472C58B15DE93B3EB9FE9300A83FA20920759D1B96E064C6E6B2C30565D52
```

I think I see what's happening here.

`make smoke` doesn't really test any part of the `thornode` codebase, it pulls
everything from docker hub.

You have to run `make build-mocknet` first to ensure you have local images built
with the current repo, or you get the most recent `:mocknet` thorchain/bifrost
image that was pushed to docker hub.

Explains why my previous tests worked and this time - after running `docker
system prune` - I got the same error on tx `30` as you. We're just using an
image of TC without my changes.

Managed to run the smoke tests to completion (`114`) after `make build-mocknet`.  
If the ci/cd pipeline is not doing this already, it probably needs updating.

- [x] make lint
- [x] make test
- [x] make smoke

### 03.03.2023 Friday 7h

Right, genesis node setup...

https://gitlab.com/mayachain/devops/cluster-launcher  
https://docs.mayaprotocol.com/maya-protocol-devs/mayanodes/kubernetes  
https://docs.mayaprotocol.com/maya-protocol-devs/mayanodes/joining#create-maya-address-and-mnemonic  
https://docs.mayaprotocol.com/maya-protocol-devs/mayanodes/deploying  

```
./mayanode-ops
  |./cluster-launcher
  |./node-launcher
```

ohio still seems to be a decent choice for a cheap region  
us-east-2

repo: `gitlab.com/mayachain/thornode`  
branch: `genesis-nodes`

---

repo: `gitlab.com/mayachain/devops/cluster-launcher`  
branch: `master`

```
export AWS_PROFILE="astound"
make aws
```

That did a `cd aws && terraform init && terraform apply`

```
kubectl -n mayanode-stagenet get pods
kubectl -n mayanode-stagenet exec --stdin --tty bitcoin-daemon-89965f8cf-vmq64 -- bitcoin-cli getblockcount
kubectl -n mayanode-stagenet exec --stdin --tty ethereum-daemon-59989f76cf-v55kx -- bitcoin-cli getblockcount
```

Single large aws node showing up correctly.

NOTE: The `make aws-backups` optional step fails with:
> make: *** No rule to make target `aws-backups'.  Stop.

Doesn't seem to exist anywhere in the project.

---

repo: `gitlab.com/mayachain/devops/node-launcher`
branch: `master`

Updated helm through brew.
Updated helm diff manually.
Installed all the GNU versions of commands and temporarily updated my path to use those in place of the macos default:

```
unalias grep
export PATH="/opt/homebrew/opt/gawk/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/gnu-sed/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/gnu-tar/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/make/libexec/gnubin:$PATH"
export PATH="/opt/homebrew/opt/findutils/libexec/gnubin:$PATH"
```

```
make tools
```

It says:
```
export POD_NAME=$(kubectl get pods -n kube-system -l "app.kubernetes.io/name=kubernetes-dashboard,app.kubernetes.io/instance=kubernetes-dashboard" -o jsonpath="{.items[0].metadata.name}")
  kubectl -n kube-system port-forward $POD_NAME 9090:9090
  echo http://127.0.0.1:9090/
```

```
make install
```

> stagenet  
> validator  
> mayanode-stagenet  

Asked to provide secret. Set in pw valut as: `aws.eks.dashmaya.node`

```
error: unable to upgrade connection: container not found ("mnemonic")
:: Mnemonic generation failed. Please try again.
make: *** [Makefile:61: install] Error 1
```

Think the branch has to be: `genesis-daemons`.

`registry.gitlab.com/mayachain/thornode:stagenet-1.96.1@sha256:479d54749f89661efea35847e4aaca49841710811b35e5ab1e3674712943f501`  
`registry.gitlab.com/mayachain/thornode:stagenet-1.96.1@sha256:9503ce7031cf5a001481cb55f3dab0348ccd4bd8866033146fdcdc6888b9ae5e`

Yeah most recent commits are to that branch. That image exists.  
It looks like we're only running thornode and maya. Everything else is turned off.

```
Setting THORNode as Validator node
Missing PEER / SEEDS
```

```
thornode keys --keyring-backend file add "maya" --output json
```

It's just `BTC` and `ETH` that need to be up and running.

> Yeah, you can start syncing if you want, you'll just have to update mayanode-stack/values.yaml to enable BTC, ETH


> Error: Failed to render chart: exit status 1: Error: template: mayanode-stack/charts/midgard/templates/statefulset.yaml:6:8: executing "mayanode-stack/charts/midgard/templates/statefulset.yaml" at <include "midgard.labels" .>: error calling include: template: mayanode-stack/charts/midgard/templates/_helpers.tpl:40:30: executing "midgard.labels" at <include "daemon.tag" .>: error calling include: template: mayanode-stack/charts/ethereum-daemon/templates/_helpers.tpl:67:14: executing "daemon.tag" at <.Values.image.eth.tag>: nil pointer evaluating interface {}.tag


Going to try again from clean:

`make destroy`

Also need to know how to set that mnemonic.

> Error from server (NotFound): secrets "mayanode-mnemonic" not found

So it's called `mayanode-mnemonic`

```
mnemonic="<REDACTED>"
kubectl -n "mayanode-stagenet" create secret generic mayanode-mnemonic --from-literal=mnemonic="$mnemonic"
kubectl -n "mayanode-stagenet" create secret generic thornode-mnemonic --from-literal=mnemonic="$mnemonic"
```

Test running with eth/btc enabled:
```
helm template mayanode-stagenet ./mayanode-stack \
  --namespace mayanode-stagenet \
  --set global.passwordSecret=mayanode-password \
  --set global.mnemonicSecret=mayanode-mnemonic \
  --set global.net=stagenet \
  --set mayanode.type=validator \
  --values ./mayanode-stack/stagenet.yaml \
  --validate
```

```
helm install mayanode-stagenet ./mayanode-stack \
  --namespace mayanode-stagenet \
  --set global.passwordSecret=mayanode-password \
  --set global.mnemonicSecret=mayanode-mnemonic \
  --set global.net=stagenet \
  --set mayanode.type=validator \
  --values ./mayanode-stack/stagenet.yaml \
  --dry-run \
  --debug
```

Trying a thornode stack:

```
helm template thornode-stagenet ./thornode-stack \
  --namespace thornode-stagenet \
  --set global.passwordSecret=thornode-password \
  --set global.mnemonicSecret=thornode-mnemonic \
  --set global.net=stagenet \
  --set thornode.type=validator \
  --values ./thornode-stack/stagenet.yaml \
  --validate
```

```
mayanode-stack/charts/midgard/templates/statefulset.yaml:6:8:

midgard.labels
mayanode-stack/charts/midgard/templates/_helpers.tpl:40
midgard.labels
daemon.tag
mayanode-stack/charts/ethereum-daemon/templates/_helpers.tpl:67
daemon.tag
.Values.image.eth.tag
nil pointer evaluating interface {}.tag

Error: plugin "diff" exited with error
helm.go:84: [debug] plugin "diff" exited with error
```

I deleted everything in statefulset.yaml and get the same error.

```
helm dependency build ./mayanode-stack && helm lint ./mayanode-stack
```

Got past the very strange helm modgard/ethereum template error.
Now I'm faced with another image pull issue:

```
docker image pull registry.gitlab.com/mayachain/devops/node-launcher:bitcoin-daemon-24.0.1@sha256:e67498d34f39fca2050ef2ecefcba324f4acaf7b9ab2491748565d9a7fcfc876
```

Just ran it without the sha and updated manually.

Testing ethereum:

```
kubectl -n mayanode-stagenet get pods
kubectl -n mayanode-stagenet exec --stdin --tty ethereum-daemon-59989f76cf-jhsgw -- sh
geth attach
eth.syncing
eth.blockNumber
```

Damn. That returned `false` and `0`.

```
kubectl -n mayanode-stagenet logs --follow ethereum-daemon-59989f76cf-jhsgw
```

It's "Looking for peers" a helluva lot.

This is the command it's running:

```
geth \
  --syncmode snap \
  --cache 4096 \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.api eth,net,engine,web3,miner,personal,txpool,debug \
  --http.corsdomain * \
  --http.vhosts=* \
  --authrpc.vhosts=localhost \
  --authrpc.jwtsecret=/
```

```
out version:     Geth/v1.10.26
latest release:  Geth/v1.11.2
```

Perhaps that's causing some issues?

```
docker image pull ethereum/client-go
```

> a8f5f00a46114b25f4300ce31819455c6008bcaf8b4ffe247e17a98737b99a84

```
docker run --rm -it ethereum/client-go version
```

> `1.11.3-unstable`

Side note: Getting this error for bitcoin:

> Startup probe failed: error: Authorization failed: Incorrect rpcuser or rpcpassword

Fixed with changes from `daemons` branch.

### 04.03.2023 Saturday 2h

Changing over to `master` on `node-launcher` as Itzamna pushed updates.

Little bit of manual k8s encouragement was needed.  
Kept getting `dialing failed ... i/o timeout` on thornode but a few restarts did the trick.  
Well, it was more of a:
- scale down to 0
- delete vpc
- scale up to 1
- 🤞

It's called `mayanode-stagenet` even though it will eventually not be stagenet,
but there isn't an easy way to change that I feel comfortable with right now.
Eth sync status being something I don't want to touch with the maya release
date so close.

### 06.03.2023 Monday 8h

Unfortunately, the ethereum beacon didn't finish syncing over the weekend.  
We're at `2821344/5941770`.

```
kubectl -n mayanode-stagenet exec --stdin --tty ethereum-daemon-59989f76cf-jhsgw -- sh
geth attach
eth.syncing
eth.blockNumber

kubectl -n mayanode-stagenet get pods
kubectl -n mayanode-stagenet exec --stdin --tty bitcoin-daemon-89965f8cf-vmq64 -- bitcoin-cli getblockcount
kubectl -n mayanode-stagenet exec --stdin --tty ethereum-daemon-59989f76cf-v55kx -- sh
```

- bitcoin: auth incorrect
- bitcoin: block sync 100%
- ethereum: beacon sync 47% (2826848/5941823)
- mayanode: at block 4628
- thornode: block sync 100%

Auth issues returned, sent patch.

```
-             - -rpcauth=mayachain:d8fb9f4b5ac5e99dce5f3b4d89d12bea$86d80993816d355e118fde58e439716dd1e2e0a9b6973287858c7cac85896eb3
+             - -rpcauth=thorchain:d7e53bb9757b6d4fabf87775c7824b5c$7097e9cde30ef4319ed708fc559267679ae6cc0bf7e18fd49b283650c0c26a10
```

Everything is green now, which is quite nice.

`make status` actually works for me now.

Other people are having auth issues.

Suggested this as a way to check what the `rpcauth` arg is:
```
kubectl -n mayanode-stagenet exec service/bitcoin-daemon -- cat /proc/1/cmdline
```

Have to use `cat /proc...` because `ps` hasn't been installed, or perhaps was
deleted for security reasons.

Went to the gym, came back, ETH, BTC and MAYA all down.

> 0/2 nodes are available: 2 Insufficient cpu. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod.

Same error for all.  
Shouldn't it auto-scale nodes??

> Could not launch Spot Instances. UnfulfillableCapacity - Unable to fulfill capacity due to your request configuration. Please adjust your request and try again. Launching EC2 instance failed

Now I have a volume node affinity conflict for eth and btc.  
DAMN.

What aws az is the eth pvc in?

```
kubectl get pvc -n mayanode-stagenet
kubectl get pv
kubectl describe pv pvc-3ab9f2e1-5c31-4bb1-8fde-ee8d3eaf3656
```

node affinity > required terms > term 0:  
`us-east-2c`

Okay what az is the new node in?

```
kubectl get nodes
```

```
ip-10-0-20-32.us-east-2.compute.internal    Ready    <none>   156m    v1.24.10-eks-48e63af
ip-10-0-43-114.us-east-2.compute.internal   Ready    <none>   3h50m   v1.24.10-eks-48e63af
```

`ProviderID:                   aws:///us-east-2a/i-0641a5b60580bc7bd`

```
kubectl describe node ip-10-0-20-32.us-east-2.compute.internal | grep us-east
kubectl describe node ip-10-0-43-114.us-east-2.compute.internal | grep us-east
```

I'm attempting to fix this by creating a new node group in us-east-2c with a single node.  
Hopefully the pods will be scheduled to it.

Node group: mayanode-fix-eth-btc

This happened because the cluster/node launcher stuff is configured to use aws "spot" ec2 instances which get disconnected when there's a lot of activity. They're cheaper but run the risk of being shut down at a moments notice. That's what happened to our setup.

Might need:
- This SG sg-031b502a1d5cb846e
- This launch template lt-059d9566fbb11ae7b / mayanode-2023030322130453650000000e

Bifrost is showing this:

> {"level":"error","service":"bifrost","module":"thorchain_client","error":"failed to get node status: failed to get node status: failed to get node account: Status code: 404 Not Found returned","time":"2023-03-07T00:02:28Z","caller":"/app/bifrost/thorclient/thorchain.go:301","message":"observer is n


### 08.03.2023 Monday 4h

```
NET=mainnet TYPE=validator make recycle
```

```
10-0-59-79
2.72 cpu
us-east-2c
```

```
10-0-59-81
4 cpu
```

Bifrost failed to pull image now.

```
docker image pull registry.gitlab.com/mayachain/mayanode:stagenet-1.97.0
docker image pull registry.gitlab.com/mayachain/mayanode:stagenet-1.98.0


docker image pull registry.gitlab.com/mayachain/mayanode:4ed4c4de5
docker run --rm -it registry.gitlab.com/mayachain/mayanode:4ed4c4de5 sh
1.98.0

docker image pull registry.gitlab.com/mayachain/thornode:stagenet-1.97.0
docker image pull registry.gitlab.com/mayachain/thornode:stagenet-1.98.0

docker image inspect registry.gitlab.com/mayachain/mayanode:4ed4c4de5

4ed4c4de5
d59b60e4a4b1ea6e7b391df75e6498a480a7b2bdf91afd803a3e58ef765321a5
```

Now it's:
`docker image pull registry.gitlab.com/mayachain/mayanode:4ed4c4de5`

`950286c04e4df7c96de493edec06262a9b2042c9ff5b8e7ef2532a95705498c1`

### 09.03.2023 Thursday 4h

WARNING: It might be a good idea to delete the lines which remove the secrets
         before running `make recycle` otherwise the wallet will be recreated
         with a different mnemonic.

```
NAME=mayanode-stagenet NET=stagenet TYPE=validator make recycle
```

```
mnemonic="..."
kubectl -n "mayanode-stagenet" create secret generic mayanode-mnemonic --from-literal=mnemonic="$mnemonic"
kubectl -n "mayanode-stagenet" create secret generic thornode-mnemonic --from-literal=mnemonic="$mnemonic"
```

correct  
`smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk`

incorrect  
`smaya16x0a20z73nn2gdrl2dryh038l2zk474h8aqewf`


### 13.03.2023 Monday 14h

```
make debug
find / -name '*genesis*'

rm -rf /root/.mayanode/data/{*.db,snapshots}
rm -rf /root/.mayanode/config/genesis.json
```

```
NAME=mayanode-stagenet NET=stagenet TYPE=validator make recycle
```

```
mayanode version
mayanode keys list --keyring-backend file
```

```
make shell
mayanode tx mayachain deposit 100000000 cacao bond:smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk \
  --from mayachain \
  --keyring-backend file \
  --chain-id mayachain-stagenet-v1
  --yes \
  --gas auto \
  --node tcp://localhost:27147
```

> Database compacting, degraded performance

Don't like the look of that!

> Unable to process past deposit contract logs, perhaps your execution client is not fully synced" error="no contract code at given address" prefix=powchain

Or that on beacon.

> Attestation is too old to broadcast, discarding it. Current Slot: 5991919 , Attestation Slot: 5991885

geth:

> Checkpoint challenge timed out, dropping id=bdfa6602f579b499 conn=dyndial addr=34.202.193.18:30303  type=Geth/v1.11.1-stable/...

```
--datadir=/data/beacon2
--checkpoint-sync-url=https://sync-mainnet.beaconcha.in
--genesis-beacon-api-url=https://sync-mainnet.beaconcha.in
```

```
make update
NET=stagenet TYPE=validator NAME=mayanode-stagenet make recycle
- When prompted to install choose n
make pre-install

kubectl -n mayanode-stagenet delete secrets mayanode-mnemonic
kubectl -n mayanode-stagenet create secret generic mayanode-mnemonic --from-literal=mnemonic=""
NET=stagenet TYPE=validator NAME=mayanode-stagenet make install
```

https://stagenet.mayanode.mayachain.info/mayachain/nodes

I think `smaya1cqdy2xguhsnsx3j285qgd5sd50cyq70z43gtuv` is their node.
It's the only one on the list right now.

```
kubectl config set-context –current –namespace=mayanode-stagenet
```

```
rm -rf /root/.mayanode/data/*.db
rm -rf /root/.mayanode/data/snapshots
rm -rf /root/.mayanode/data/cs.wal
rm -rf /root/.mayanode/config/genesis.json

vi /root/.mayanode/config/addrbook.json
cat /root/.mayanode/config/addrbook.json | jq '.addrs = []' > /root/.mayanode/config/addrbook.json

vi ~/.mayanode/config/config.toml
```

```
kubectl logs -f service/mayanode
```

`{"height":"0","txhash":"BBA6054D78501E25F7775684400E57B7C7A9FB029C56BEBE00E50C18BE4C8A14","codespace":"sdk","code":4,"data":"","raw_log":"signature verification failed; please verify account number (4) and chain-id (mayachain-stagenet-v1): unauthorized","logs":[],"info":"","gas_wanted":"176907","gas_used":"38500","tx":null,"timestamp":"","events":[]}`


```
make shell

mayanode tx mayachain deposit 100000000 cacao bond:smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk \
  --from mayachain \
  --keyring-backend file \
  --chain-id mayachain-mainnet-v1 \
  --yes \
  --gas auto \
  --node tcp://localhost:27147

mayanode tx mayachain deposit 100000000 cacao bond:smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk \
  --from mayachain \
  --keyring-backend file \
  --chain-id mayachain-stagenet-v1 \
  --yes \
  --gas auto \
  --node tcp://localhost:27147
```

```
{"height":"3367","txhash":"9587D4B592104FB7FDAE54DFE0F48FD1A07D92083D4CB89082D5DE16D24DAD30","codespace":"","code":0,"data":"0A130A112F74797065732E4D73674465706F736974","raw_log":"[{\"events\":[{\"type\":\"bond\",\"attributes\":[{\"key\":\"amount\",\"value\":\"0\"},{\"key\":\"bond_type\",\"value\":\"\\u0000\"},{\"key\":\"id\",\"value\":\"9587D4B592104FB7FDAE54DFE0F48FD1A07D92083D4CB89082D5DE16D24DAD30\"},{\"key\":\"chain\",\"value\":\"MAYA\"},{\"key\":\"from\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"to\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"},{\"key\":\"coin\",\"value\":\"100000000 MAYA.CACAO\"},{\"key\":\"memo\",\"value\":\"bond:smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"}]},{\"type\":\"coin_received\",\"attributes\":[{\"key\":\"receiver\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"receiver\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"},{\"key\":\"amount\",\"value\":\"100000000cacao\"}]},{\"type\":\"coin_spent\",\"attributes\":[{\"key\":\"spender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"spender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"100000000cacao\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"deposit\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"}]},{\"type\":\"new_node\",\"attributes\":[{\"key\":\"address\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"recipient\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"100000000cacao\"}]}]}]","logs":[{"msg_index":0,"log":"","events":[{"type":"bond","attributes":[{"key":"amount","value":"0"},{"key":"bond_type","value":"\u0000"},{"key":"id","value":"9587D4B592104FB7FDAE54DFE0F48FD1A07D92083D4CB89082D5DE16D24DAD30"},{"key":"chain","value":"MAYA"},{"key":"from","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"to","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"},{"key":"coin","value":"100000000 MAYA.CACAO"},{"key":"memo","value":"bond:smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"}]},{"type":"coin_received","attributes":[{"key":"receiver","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"amount","value":"2000000cacao"},{"key":"receiver","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"},{"key":"amount","value":"100000000cacao"}]},{"type":"coin_spent","attributes":[{"key":"spender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"},{"key":"spender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"100000000cacao"}]},{"type":"message","attributes":[{"key":"action","value":"deposit"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"}]},{"type":"new_node","attributes":[{"key":"address","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"}]},{"type":"transfer","attributes":[{"key":"recipient","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"},{"key":"recipient","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"100000000cacao"}]}]}],"info":"","gas_wanted":"176907","gas_used":"175172","tx":null,"timestamp":"","events":[{"type":"tx","attributes":[{"key":"ZmVl","value":"","index":true},{"key":"ZmVlX3BheWVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"tx","attributes":[{"key":"YWNjX3NlcQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGsvMA==","index":true}]},{"type":"tx","attributes":[{"key":"c2lnbmF0dXJl","value":"NEdwUmVMdVBPZEFlU28yQzJPSnBMRUlqcFI0ZzBvUktNbkFnbU5wcGkzTUZRTGhhcGtQc1ZiQjNQSXVxT0R6SGpNeGUxb3hOWmVzTDNBWkVmNVVGN2c9PQ==","index":true}]},{"type":"message","attributes":[{"key":"YWN0aW9u","value":"ZGVwb3NpdA==","index":true}]},{"type":"coin_spent","attributes":[{"key":"c3BlbmRlcg==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"coin_received","attributes":[{"key":"cmVjZWl2ZXI=","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"transfer","attributes":[{"key":"cmVjaXBpZW50","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"message","attributes":[{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"coin_spent","attributes":[{"key":"c3BlbmRlcg==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MTAwMDAwMDAwY2FjYW8=","index":true}]},{"type":"coin_received","attributes":[{"key":"cmVjZWl2ZXI=","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true},{"key":"YW1vdW50","value":"MTAwMDAwMDAwY2FjYW8=","index":true}]},{"type":"transfer","attributes":[{"key":"cmVjaXBpZW50","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true},{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MTAwMDAwMDAwY2FjYW8=","index":true}]},{"type":"message","attributes":[{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"new_node","attributes":[{"key":"YWRkcmVzcw==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"bond","attributes":[{"key":"YW1vdW50","value":"MA==","index":true},{"key":"Ym9uZF90eXBl","value":"AA==","index":true},{"key":"aWQ=","value":"OTU4N0Q0QjU5MjEwNEZCN0ZEQUU1NERGRTBGNDhGRDFBMDdEOTIwODNENENCODkwODJENURFMTZEMjREQUQzMA==","index":true},{"key":"Y2hhaW4=","value":"TUFZQQ==","index":true},{"key":"ZnJvbQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"dG8=","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true},{"key":"Y29pbg==","value":"MTAwMDAwMDAwIE1BWUEuQ0FDQU8=","index":true},{"key":"bWVtbw==","value":"Ym9uZDpzbWF5YTFndjg1djBqdmMwcnNqdW5rdTNxeGVtcGF4Nmttcmc1ajV3bTZkaw==","index":true}]}]}
```

18.191.59.213

```
make set-ip-address
```

`{"height":"3395","txhash":"F4A986FCA5E00DED3F4ACAFA56906555B2CAD99DFAEA38D9240180B81F190332","codespace":"","code":0,"data":"0A180A162F74797065732E4D7367536574495041646472657373","raw_log":"[{\"events\":[{\"type\":\"bond\",\"attributes\":[{\"key\":\"amount\",\"value\":\"2000000\"},{\"key\":\"bond_type\",\"value\":\"\\u0003\"},{\"key\":\"id\",\"value\":\"0000000000000000000000000000000000000000000000000000000000000000\"},{\"key\":\"chain\"},{\"key\":\"from\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"to\"},{\"key\":\"coin\"},{\"key\":\"memo\"}]},{\"type\":\"coin_received\",\"attributes\":[{\"key\":\"receiver\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]},{\"type\":\"coin_spent\",\"attributes\":[{\"key\":\"spender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"set_ip_address\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"}]},{\"type\":\"set_ip_address\",\"attributes\":[{\"key\":\"maya_address\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"address\",\"value\":\"3.136.127.159\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]}]}]","logs":[{"msg_index":0,"log":"","events":[{"type":"bond","attributes":[{"key":"amount","value":"2000000"},{"key":"bond_type","value":"\u0003"},{"key":"id","value":"0000000000000000000000000000000000000000000000000000000000000000"},{"key":"chain","value":""},{"key":"from","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"to","value":""},{"key":"coin","value":""},{"key":"memo","value":""}]},{"type":"coin_received","attributes":[{"key":"receiver","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"amount","value":"2000000cacao"}]},{"type":"coin_spent","attributes":[{"key":"spender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"}]},{"type":"message","attributes":[{"key":"action","value":"set_ip_address"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"}]},{"type":"set_ip_address","attributes":[{"key":"maya_address","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"address","value":"3.136.127.159"}]},{"type":"transfer","attributes":[{"key":"recipient","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"}]}]}],"info":"","gas_wanted":"141458","gas_used":"68772","tx":null,"timestamp":"","events":[{"type":"tx","attributes":[{"key":"ZmVl","value":"","index":true},{"key":"ZmVlX3BheWVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"tx","attributes":[{"key":"YWNjX3NlcQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGsvMQ==","index":true}]},{"type":"tx","attributes":[{"key":"c2lnbmF0dXJl","value":"amJBWlFTaDRjNEtpbDFNSldCdloxRmFIbElZdHAxcWlhdTBxZUJRWi8wOVgzb0lJcnZxa3diMXRYekdPUDIrSzJSTkRZOUlLeXBtc09odVVpUkp0dGc9PQ==","index":true}]},{"type":"message","attributes":[{"key":"YWN0aW9u","value":"c2V0X2lwX2FkZHJlc3M=","index":true}]},{"type":"coin_spent","attributes":[{"key":"c3BlbmRlcg==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"coin_received","attributes":[{"key":"cmVjZWl2ZXI=","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"transfer","attributes":[{"key":"cmVjaXBpZW50","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"message","attributes":[{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"bond","attributes":[{"key":"YW1vdW50","value":"MjAwMDAwMA==","index":true},{"key":"Ym9uZF90eXBl","value":"Aw==","index":true},{"key":"aWQ=","value":"MDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMA==","index":true},{"key":"Y2hhaW4=","value":"","index":true},{"key":"ZnJvbQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"dG8=","value":"","index":true},{"key":"Y29pbg==","value":"","index":true},{"key":"bWVtbw==","value":"","index":true}]},{"type":"set_ip_address","attributes":[{"key":"bWF5YV9hZGRyZXNz","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YWRkcmVzcw==","value":"My4xMzYuMTI3LjE1OQ==","index":true}]}]}`

```
make set-version
```

```
make set-node-keys
```

`{"height":"3424","txhash":"DFC1391B9E91ECF6049D77EACE534597B6A3C18CBCB6A4E78ECBC4F9BF5D7668","codespace":"","code":0,"data":"0A170A152F74797065732E4D73675365744E6F64654B657973","raw_log":"[{\"events\":[{\"type\":\"bond\",\"attributes\":[{\"key\":\"amount\",\"value\":\"2000000\"},{\"key\":\"bond_type\",\"value\":\"\\u0003\"},{\"key\":\"id\",\"value\":\"0000000000000000000000000000000000000000000000000000000000000000\"},{\"key\":\"chain\"},{\"key\":\"from\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"to\"},{\"key\":\"coin\"},{\"key\":\"memo\"}]},{\"type\":\"coin_received\",\"attributes\":[{\"key\":\"receiver\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"receiver\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]},{\"type\":\"coin_spent\",\"attributes\":[{\"key\":\"spender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"spender\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"set_node_keys\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"sender\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"}]},{\"type\":\"set_node_keys\",\"attributes\":[{\"key\":\"node_address\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"node_secp256k1_pubkey\",\"value\":\"smayapub1addwnpepqw4upnvtujukvayxvlg37tsgsmu9ya2ahpl2p7q8azn2mzmfsq3tvafhjsl\"},{\"key\":\"node_ed25519_pubkey\",\"value\":\"smayapub1zcjduepq4sw24y0rppsazs5kjzyu8wujyewgn96xpft7xfm4n8whwqgarj3s92lu4m\"},{\"key\":\"validator_consensus_pub_key\",\"value\":\"smayacpub1zcjduepqq48s6pue3v4yd9c3a0rdtpc5j2d052wzc3lcs9c5qksrdwqyw5qspuykcu\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"sender\",\"value\":\"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"},{\"key\":\"recipient\",\"value\":\"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9\"},{\"key\":\"sender\",\"value\":\"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz\"},{\"key\":\"amount\",\"value\":\"2000000cacao\"}]}]}]","logs":[{"msg_index":0,"log":"","events":[{"type":"bond","attributes":[{"key":"amount","value":"2000000"},{"key":"bond_type","value":"\u0003"},{"key":"id","value":"0000000000000000000000000000000000000000000000000000000000000000"},{"key":"chain","value":""},{"key":"from","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"to","value":""},{"key":"coin","value":""},{"key":"memo","value":""}]},{"type":"coin_received","attributes":[{"key":"receiver","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"amount","value":"2000000cacao"},{"key":"receiver","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"amount","value":"2000000cacao"}]},{"type":"coin_spent","attributes":[{"key":"spender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"},{"key":"spender","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"},{"key":"amount","value":"2000000cacao"}]},{"type":"message","attributes":[{"key":"action","value":"set_node_keys"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"sender","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"}]},{"type":"set_node_keys","attributes":[{"key":"node_address","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"node_secp256k1_pubkey","value":"smayapub1addwnpepqw4upnvtujukvayxvlg37tsgsmu9ya2ahpl2p7q8azn2mzmfsq3tvafhjsl"},{"key":"node_ed25519_pubkey","value":"smayapub1zcjduepq4sw24y0rppsazs5kjzyu8wujyewgn96xpft7xfm4n8whwqgarj3s92lu4m"},{"key":"validator_consensus_pub_key","value":"smayacpub1zcjduepqq48s6pue3v4yd9c3a0rdtpc5j2d052wzc3lcs9c5qksrdwqyw5qspuykcu"}]},{"type":"transfer","attributes":[{"key":"recipient","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"sender","value":"smaya1gv85v0jvc0rsjunku3qxempax6kmrg5j5wm6dk"},{"key":"amount","value":"2000000cacao"},{"key":"recipient","value":"smaya1dheycdevq39qlkxs2a6wuuzyn4aqxhvepwy3x9"},{"key":"sender","value":"smaya17gw75axcnr8747pkanye45pnrwk7p9c3vw3zsz"},{"key":"amount","value":"2000000cacao"}]}]}],"info":"","gas_wanted":"194980","gas_used":"95533","tx":null,"timestamp":"","events":[{"type":"tx","attributes":[{"key":"ZmVl","value":"","index":true},{"key":"ZmVlX3BheWVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"tx","attributes":[{"key":"YWNjX3NlcQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGsvMw==","index":true}]},{"type":"tx","attributes":[{"key":"c2lnbmF0dXJl","value":"Z3JqbnIxQ3g2dkErODBHaTdlYmdjc2dFWkVQeUNVYUxzeldUYkZFNS90RkIwSVV3aU44SEdLOXd1TU9vNjMvaFVsR1didnRmOXpRWGJtLzI2engvd1E9PQ==","index":true}]},{"type":"message","attributes":[{"key":"YWN0aW9u","value":"c2V0X25vZGVfa2V5cw==","index":true}]},{"type":"coin_spent","attributes":[{"key":"c3BlbmRlcg==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"coin_received","attributes":[{"key":"cmVjZWl2ZXI=","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"transfer","attributes":[{"key":"cmVjaXBpZW50","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"message","attributes":[{"key":"c2VuZGVy","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true}]},{"type":"coin_spent","attributes":[{"key":"c3BlbmRlcg==","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"coin_received","attributes":[{"key":"cmVjZWl2ZXI=","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"transfer","attributes":[{"key":"cmVjaXBpZW50","value":"c21heWExZGhleWNkZXZxMzlxbGt4czJhNnd1dXp5bjRhcXhodmVwd3kzeDk=","index":true},{"key":"c2VuZGVy","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true},{"key":"YW1vdW50","value":"MjAwMDAwMGNhY2Fv","index":true}]},{"type":"message","attributes":[{"key":"c2VuZGVy","value":"c21heWExN2d3NzVheGNucjg3NDdwa2FueWU0NXBucndrN3A5YzN2dzN6c3o=","index":true}]},{"type":"bond","attributes":[{"key":"YW1vdW50","value":"MjAwMDAwMA==","index":true},{"key":"Ym9uZF90eXBl","value":"Aw==","index":true},{"key":"aWQ=","value":"MDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMA==","index":true},{"key":"Y2hhaW4=","value":"","index":true},{"key":"ZnJvbQ==","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"dG8=","value":"","index":true},{"key":"Y29pbg==","value":"","index":true},{"key":"bWVtbw==","value":"","index":true}]},{"type":"set_node_keys","attributes":[{"key":"bm9kZV9hZGRyZXNz","value":"c21heWExZ3Y4NXYwanZjMHJzanVua3UzcXhlbXBheDZrbXJnNWo1d202ZGs=","index":true},{"key":"bm9kZV9zZWNwMjU2azFfcHVia2V5","value":"c21heWFwdWIxYWRkd25wZXBxdzR1cG52dHVqdWt2YXl4dmxnMzd0c2dzbXU5eWEyYWhwbDJwN3E4YXpuMm16bWZzcTN0dmFmaGpzbA==","index":true},{"key":"bm9kZV9lZDI1NTE5X3B1YmtleQ==","value":"c21heWFwdWIxemNqZHVlcHE0c3cyNHkwcnBwc2F6czVranp5dTh3dWp5ZXdnbjk2eHBmdDd4Zm00bjh3aHdxZ2FyajNzOTJsdTRt","index":true},{"key":"dmFsaWRhdG9yX2NvbnNlbnN1c19wdWJfa2V5","value":"c21heWFjcHViMXpjamR1ZXBxcTQ4czZwdWUzdjR5ZDljM2EwcmR0cGM1ajJkMDUyd3pjM2xjczljNXFrc3Jkd3F5dzVxc3B1eWtjdQ==","index":true}]}]}`

```
NET=stagenet NAME=mayanode-stagenet TYPE=validator make status
```

We can ignore peer `123bb6e4f77c16a50aa94a745ffc0c6b989a89ab`

Crazy long day, but we're looking good. Just midgard that isn't playing ball.

### 14.03.2023 Tuesday 4h

Today, we prep for recycling into mainnet.

```
make update
NET=mainnet TYPE=validator NAME=mayanode-stagenet make recycle

NET=mainnet TYPE=validator NAME=mayanode-stagenet make shell

mayanode tx mayachain deposit 100000000 cacao bond:maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg \
  --from mayachain \
  --keyring-backend file \
  --chain-id mayachain-mainnet-v1 \
  --yes \
  --gas auto \
  --node tcp://localhost:27147

NET=mainnet NAME=mayanode-stagenet TYPE=validator make status
NET=mainnet NAME=mayanode-stagenet TYPE=validator make logs

NET=mainnet NAME=mayanode-stagenet TYPE=validator make debug
rm -rf ~/.mayanode
NET=mainnet NAME=mayanode-stagenet TYPE=validator make reset
```

```
Error: RPC error -32603 - Internal error: timed out waiting for tx to be included in a block
```

```
Error: rpc error: code = Unknown desc = rpc error: code = Unknown desc = account sequence mismatch, expected 1, got 0: incorrect account sequence [cosmos/cosmos-sdk@v0.45.9/x/auth/ante/sigverify.go:265] With gas wanted: '0' and gas used: '33578' : unknown request
```

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-ip-address
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-version
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-node-keys
```

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make reset
```

We're running live!!


https://www.explorer.mayachain.info/nodes/


## Batch Three

```
18.03.2023 Saturday  20m
21.03.2023 Tuesday   20m
27.03.2023 Monday    10m
06.04.2023 Thursday       2h
07.04.2023 Friday         6h
12.04.2023 Wednesday      1h
14.04.2023 Friday         1h
15.04.2023 Saturday       1h
17.04.2023 Monday         1h
18.04.2023 Tuesday        1h
Total                50m 13h
```

### 18.03.2023 Saturday 20m

```
NET=mainnet TYPE=validator NAME=mayanode-stagenet make update
NET=mainnet TYPE=validator NAME=mayanode-stagenet make backup
NET=mainnet TYPE=validator NAME=mayanode-stagenet make status
NET=mainnet TYPE=validator NAME=mayanode-stagenet make set-ip
NET=mainnet TYPE=validator NAME=mayanode-stagenet make set-version
```

### 21.03.2023 Tuesday 20m

```
NET=mainnet TYPE=validator NAME=mayanode-stagenet make update
NET=mainnet TYPE=validator NAME=mayanode-stagenet make set-version

NET=mainnet TYPE=validator NAME=mayanode-stagenet make status
```

Spammed with:
> Skipping the migration of funds while transactions are still pending

Logs looking good otherwise.

### 27.03.2023 Monday 10m

`NET=mainnet TYPE=validator NAME=mayanode-stagenet make update`

### 06.04.2023 Thursday 2h

Itsamna asked me to open the same MR against Maya.

```
git clone git@gitlab.com:mayachain/mayanode.git
cd mayanode
git remote add alex-thornode git@gitlab.com:alexdcox/thornode.git
git checkout v1.96.1
git checkout -b add-dash-chain
git merge alex-thornode/982-add-dash-chain

git diff c279c48b9..bdbcd65a7
```

That resulted in way too many changes.

```
gco v1.96.1
git reset --hard HEAD
gb -D add-dash-chain
gco -b add-dash-chain
```

```
git merge alex-thornode/982-add-dash-chain --no-commit
```

Nope. Did a fast-forward.

```
git merge alex-thornode/982-add-dash-chain --no-ff --no-commit
```

Okay that's what I wanted, staged changes without a commit.
Now to whittle this down to just what we actually want.
Here's the entire file list:

Perhaps this isn't the easiest way to do this.

Options:
- cherry pick the dash related commits
- get the patch of thorchain -> dash and see if that can be cleanly applied to maya
- do what I'm already doing and just work through it

```
git merge -s recursive -X ours alex-thornode/982-add-dash-chain --no-ff --no-commit
```

Still a bit brutal.
May as well try the patch approach, that'll have to wait until tomorrow.


### 07.04.2023 Friday 6h

Right trying to get the cleanest patch I can from thornode, to then apply to maya
in a separate branch...

Most recent tag on my thornode/dash branch is `v1.101.0`

```
gd v1.101.0..982-add-dash-chain > add-dash.patch
```

`build/scripts/node-status.sh`
quite heavily modified

`thornode/cmd/genaccounts.go`
didn't copy accross this last line fix:
```
  cmd.Flags().String(flags.FlagHome, defaultNodeHome, "The application home directory")
  cmd.Flags().String(flagVestingAmt, "", "amount of coins for vesting accounts")
  cmd.Flags().Int64(flagVestingStart, 0, "schedule start time (unix epoch) for vesting accounts")
  cmd.Flags().Int64(flagVestingEnd, 0, "schedule end time (unix epoch) for vesting accounts")
>>>  cmd.Flags().String(flags.FlagKeyringBackend, flags.DefaultKeyringBackend, "Select keyring's backend (os|file|kwallet|pass|test)")
```

`common/chain.go`
Needs manual review

`config/config.go`
Didn't apply this fix
```
assert(viper.BindEnv(
  "bifrost.chains.dash.block_scanner.block_height_discover_back_off",
  "BLOCK_SCANNER_BACKOFF",
))
```

`pubkey2address/pubkey2address.go`
Didn't add the last line of each of these:
```
  case common.TestNet, common.MockNet:
    fmt.Println("THORChain testnet|mocknet:")
```
```
  default:
    fmt.Printf("No thorchain bech32 prefixes are configured for chain: '%v'\n", nw)
```

`mayanode/Makefile`
added the volume mount, how are people making this work without that!??

Right that's looking very nice and clean.
Apart from the eye watering amount of package errors.

Tidying module.
Oh, we have to specify the version.

```
go mod tidy -compat=1.17
```

Running unit tests.

Ah dash bifrost client is relying on some refactored methods that aren't
currently present in maya.

Let Itzamna know the MR is up. Still in draft atm though.

Now I need to update the smoke tests.

Rebuild image failed on my vm.

`builder-v2` is no longer available.
Use `registry.gitlab.com/thorchain/thornode:builder-v3`.

Had to install `protoc-gen-gocosmos` and add `/root/go/bin` to `PATH`.

```
go install protoc-gen-gocosmos
```

My `mayanode:add-dash-chain` has a `build/docker/docker-compose` with older node
versions than build by `./ci/images/build.sh` in `node-launcher:add-dash`.

Need to update these.

```
gaia-daemon-8.0.1
bitcoin-daemon-24.0.1
bitcoin-cash-daemon-26.0.0
dogecoin-daemon-1.14.6
dash-daemon-latest
avalanche-daemon-1.9.9
litecoin-daemon-0.21.2.2
```

Oh also, we're pulling from `thornode` not `mayachain`.
Noticed they've just put `latest` for dash, that does match my Dockerfile tbf.

`litecoin` didn't work.

I split all the `RUN` commands into their own lines to help debug, and it worked.
I'm going to pretend it worked the first time round 🙈

That failed like a spacex prototype. Explosively, impressively.
Failed to connect.
Looks like ethereum is having troubles:

```
WARN [04-08|04:29:52.066] Failed to load snapshot, regenerating    err="missing or corrupted snapshot"
WARN [04-08|04:29:52.067] Error reading unclean shutdown markers   error="leveldb: not found"
WARN [04-08|04:29:52.067] Engine API enabled                       protocol=eth
WARN [04-08|04:29:52.067] Engine API started but chain not configured for merge yet
WARN [04-08|04:29:52.067] P2P server will be useless, neither dialing nor listening
```

Maybe that's not an issue, they are just warnings.

Might have to try again with even more cpu/mem.
Yeah that was the issue. 14 cpu / 25 gb ram.

Got error: `CONSENSUS FAILURE`

Oh.

```
5:04AM ERR go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:726 > CONSENSUS FAILURE!!! err="runtime error: invalid memory address or nil pointer dereference" module=consensus stack="goroutine 139 [running]:\nruntime/debug.Stack()\n\t/usr/local/go/src/runtime/debug/stack.go:24 +0x65\ngithub.com/tendermint/tendermint/consensus.(*State).receiveRoutine.func2()\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:726 +0x4c\npanic({0x1d210a0, 0x34d6b10})\n\t/usr/local/go/src/runtime/panic.go:838 +0x207\ngitlab.com/thorchain/thornode/x/thorchain.AppModule.BeginBlock({{}, _, {{_, _}, {_, _}, {_, _}}, _}, {{0x2614350, ...}, ...}, ...)\n\t/app/x/thorchain/module.go:205 +0xc68\ngithub.com/cosmos/cosmos-sdk/types/module.(*Manager).BeginBlock(_, {{0x2614350, 0xc000052080}, {0x26219c0, 0xc00014b0c0}, {{0xb, 0x0}, {0xc000f08c70, 0x9}, 0x1, ...}, ...}, ...)\n\t/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/types/module/module.go:479 +0x3a2\ngitlab.com/thorchain/thornode/app.(*THORChainApp).BeginBlocker(_, {{0x2614350, 0xc000052080}, {0x26219c0, 0xc00014b0c0}, {{0xb, 0x0}, {0xc000f08c70, 0x9}, 0x1, ...}, ...}, ...)\n\t/app/app/app.go:306 +0x85\ngithub.com/cosmos/cosmos-sdk/baseapp.(*BaseApp).BeginBlock(_, {{0xc0012e4d40, 0x20, 0x20}, {{0xb, 0x0}, {0xc000f08c70, 0x9}, 0x1, {0x2f810e10, ...}, ...}, ...})\n\t/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/baseapp/abci.go:194 +0x97c\ngithub.com/tendermint/tendermint/abci/client.(*localClient).BeginBlockSync(_, {{0xc0012e4d40, 0x20, 0x20}, {{0xb, 0x0}, {0xc000f08c70, 0x9}, 0x1, {0x2f810e10, ...}, ...}, ...})\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/abci/client/local_client.go:280 +0x118\ngithub.com/tendermint/tendermint/proxy.(*appConnConsensus).BeginBlockSync(_, {{0xc0012e4d40, 0x20, 0x20}, {{0xb, 0x0}, {0xc000f08c70, 0x9}, 0x1, {0x2f810e10, ...}, ...}, ...})\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/proxy/app_conn.go:81 +0x55\ngithub.com/tendermint/tendermint/state.execBlockOnProxyApp({0x2615ab8?, 0xc000ed4180}, {0x261afb8, 0xc0017fb490}, 0xc0011e61e0, {0x261ff48, 0xc0017fa760}, 0x0?)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/state/execution.go:307 +0x3dd\ngithub.com/tendermint/tendermint/state.(*BlockExecutor).ApplyBlock(_, {{{0xb, 0x0}, {0xc00063eb30, 0x7}}, {0xc00063eb37, 0x9}, 0x1, 0x0, {{0x0, ...}, ...}, ...}, ...)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/state/execution.go:140 +0x171\ngithub.com/tendermint/tendermint/consensus.(*State).finalizeCommit(0xc00142ae00, 0x1)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:1635 +0x9fd\ngithub.com/tendermint/tendermint/consensus.(*State).tryFinalizeCommit(0xc00142ae00, 0x1)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:1546 +0x305\ngithub.com/tendermint/tendermint/consensus.(*State).enterCommit.func1()\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:1481 +0x87\ngithub.com/tendermint/tendermint/consensus.(*State).enterCommit(0xc00142ae00, 0x1, 0x0)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:1519 +0xbea\ngithub.com/tendermint/tendermint/consensus.(*State).addVote(0xc00142ae00, 0xc001402000, {0x0, 0x0})\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:2132 +0xb7c\ngithub.com/tendermint/tendermint/consensus.(*State).tryAddVote(0xc00142ae00, 0xc001402000, {0x0?, 0x475446?})\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:1930 +0x2c\ngithub.com/tendermint/tendermint/consensus.(*State).handleMsg(0xc00142ae00, {{0x25fb300?, 0xc000594140?}, {0x0?, 0x0?}})\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:838 +0x3ff\ngithub.com/tendermint/tendermint/consensus.(*State).receiveRoutine(0xc00142ae00, 0x0)\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:782 +0x512\ncreated by github.com/tendermint/tendermint/consensus.(*State).OnStart\n\t/go/pkg/mod/github.com/tendermint/tendermint@v0.34.14/consensus/state.go:378 +0x11c\n"
```

Doesn't match up to my codebase.
Also I fucked with the constants:
`gitlab.com/mayachain/mayanode/constants.Version=1.96.1`
The might need to back to `thorchain/thornode`...

Think the gasmanager is nil.
`5:11AM ERR app/x/thorchain/module.go:184 > fail to get managers error="fail to create gas manager: bad version"`
Confirmed.

Low disk space. Damn.

> ModuleNotFoundError: No module named 'dashtx'

Perhaps it's not using the right `smoke` image.
A tomorrow problem.


### 12.04.2023 Wednesday 1h

They asked me to push an update.

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make update
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-version
```

### 14.04.2023 Friday 1h

```
kubectl patch pvc thornode -p '{"metadata":{"finalizers":null}}'
```

He asked me to set recover: true for thornode and make reset.

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make reset
```

### 15.04.2023 Saturday 1h

Asked to toggle recover off and update...

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make status

NET=mainnet NAME=mayanode-stagenet TYPE=validator make update
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-version
```

### 17.04.2023 Monday 1h

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make status

NET=mainnet NAME=mayanode-stagenet TYPE=validator make update
NET=mainnet NAME=mayanode-stagenet TYPE=validator make set-version
```

```
kubectl -n mayanode-stagenet logs deployment/mayanode
```

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make shell
ping 13.52.55.197
ping 18.194.46.101
ping 18.221.183.211
ping 15.156.45.237
ping 18.217.85.10
ping 3.132.55.140
```


### 18.04.2023 Tuesday 1h

```
kubectl scale --replicas=0 deploy/mayanode --timeout=5m
```

```
{
  "address": "maya1gv85v0jvc0rsjunku3qxempax6kmrg5jqh8vmg",
  "addr": []
}
```

"key": "ae1713e45cb5c579fc07b7f0ff24adad1ea93aa1",

```
kubectl get configmap gateway-external-ip -o jsonpath="{.data.externalIP}"
```

18.221.183.211
ae1713e45cb5c579fc07b7f0ff24adad1ea93aa1

```
{
  "key": "",
  "addr": []
}
```

```
/root/.mayanode/config/addrbook.json
```

```
kubectl -n mayanode-stagenet logs deployment/mayanode > mayanode-1804231318.log
```

```
NET=mainnet NAME=mayanode-stagenet TYPE=validator make shell
```

> arn:aws:eks:us-east-2:506926791942:cluster/dash-maya

```
kubectl config current-context

aws eks describe-cluster --name dash-maya
aws sts get-caller-identity

aws eks update-kubeconfig --region us-east-2 --name dash-maya
```

`NET=mainnet NAME=mayanode-stagenet TYPE=validator make status`

## Batch Four

```
09.06.2023 Friday         6h
12.06.2023 Monday         4h
13.06.2023 Tuesday        5h
14.06.2023 Wednesday     10h
23.06.2023 Friday         3h
27.06.2023 Tuesday        6h
28.06.2023 Wednesday      5h
Total                    39h
```

### 09.06.2023 Friday 6h

Back from holiday. They're adding the dash client in now live.

> Just to update you on the progress we've made for the Dash client in Maya, I
  created a new PR with a new branch that takes your changes and we applied
  some fixes and updates. We're still missing some fixes for smokes to work,
  but I think we're close


Okay what's the branch? `origin/add-dash-chain`

They said they're using delve to debug things.
https://github.com/go-delve/delve

My work vm no longer has mayanode or docker or anything we need to run the smoke
tests. Think I wiped and reinstalled after an emergency issue before I went on
holiday so Ash had a clean healthy box to work with. This of course, destroyed
my dev environment.

```
go install protoc-gen-gocosmos

apt install plocate
locate protoc-gen-gocosmos
```

```
/root/go/bin/protoc-gen-gocosmos
/root/go/pkg/mod/github.com/regen-network/cosmos-proto@v0.3.1/protoc-gen-gocosmos
/root/go/pkg/mod/github.com/regen-network/cosmos-proto@v0.3.1/protoc-gen-gocosmos/main.go
```

```
go install github.com/regen-network/cosmos-proto/protoc-gen-gocosmos
export PATH="/root/go/bin:$PATH"
```

Running the smoke tests:
```
DOCKER_BUILDKIT=0 make smoke
```

Gives me this error:
> unknown flag: --cache-from

Okay managed to get to the same place they are at with this error
Happens around smoke test `73`

```
11:07PM INF app/bifrost/signer/sign.go:229 > Signing transaction height=109 module=signer num=0 service=bifrost status=1 tx={"aggregator":"","chain":"DASH","coins":[{"amount":"2469349588","asset":"DASH.DASH"}],"gas_rate":510,"in_hash":"2ACAB022982A1EBB9DF1777F7FC0B4B6D67A65AFB85E1B045CCE2FB057B00A61","max_gas":[{"amount":"155040","asset":"DASH.DASH","decimals":8}],"memo":"OUT:2ACAB022982A1EBB9DF1777F7FC0B4B6D67A65AFB85E1B045CCE2FB057B00A61","out_hash":"","to":"yLLFQTxaW3wybbahkhyZcrdfqoRCnfzAV5","vault_pubkey":"tmayapub1addwnpepqtt62k7egujtc8dnxgxfwg3jvfr6380rdvqdu5atk2vry3h6t4u0kn307ka"}
11:07PM INF app/bifrost/signer/sign.go:489 > retrying broadcast of already signed tx memo=OUT:2ACAB022982A1EBB9DF1777F7FC0B4B6D67A65AFB85E1B045CCE2FB057B00A61 module=signer service=bifrost
11:07PM ERR app/bifrost/signer/sign.go:508 > fail to broadcast tx to chain error="fail to broadcast transaction to chain: -3: Amount is not a number or string" memo=OUT:2ACAB022982A1EBB9DF1777F7FC0B4B6D67A65AFB85E1B045CCE2FB057B00A61 module=signer service=bifrost
11:07PM ERR app/bifrost/signer/sign.go:249 > fail to sign and broadcast tx out store item error="fail to broadcast transaction to chain: -3: Amount is not a number or string" module=signer service=bifrost
```

Those are coming from `bifrost/signer/sign.go` in functions:
`processTransactions`
and 
`signAndBroadcast`

Delving deeper
`fail to broadcast transaction to chain: %w` 
is coming from `dash/signer.go:524`

The raw transaction looks like it's in the wrong format.
Need to actually see it...
I added in a line to dump the rawtx as a hex string.

How do I check it will actually build?
`go build ./cmd/mayanode`

Okay now to try that in the stack...

Is it `73     USER-1 => VAULT      [SWAP:DASH.DASH:USER-1] 1.00000000 BNB.BNB`
or is it `74`?

The rawtx is:
`01000000010167f17cf0398cf4fa5cecccb24f8c2f08ca5e15bbd3f3de13fef7f21477ccd0000000006a473044022001b86ef00430bd9fd9d5ed69f9c31daa6353d104bebc32adb31ff8c863beae7f02201580fdb8b1b8ae03777c06c7f19f42f0371f0b80ff57408917ff6b745cb06f2c01210268f36ec62b3d1bb55d21ce1f002a8860aa6a8e982d1733aec352cad971c4e597ffffffff03d4482f93000000001976a9140026dcfac0cd2092ea5a124f2089c3ad5b7aa2cd88ac8c2fe0ea020000001976a91443634a1c7b1ebfa28a1f2fa5ec50d25f204f80f888ac0000000000000000466a444f55543a3030353330424132333633313531354634433537353834383043434437323233414243424446374530303031434134364139313836314243443745323439333400000000`

```
{
  "txid": "c99e6194cbcf72a7cee3ec35b3c7d5feedba114551375cc71b79581995dee135",
  "version": 1,
  "type": 0,
  "size": 304,
  "locktime": 0,
  "vin": [
    {
      "txid": "d0cc7714f2f7fe13def3d3bb155eca082f8c4fb2ccec5cfaf48c39f07cf16701",
      "vout": 0,
      "scriptSig": {
        "asm": "3044022001b86ef00430bd9fd9d5ed69f9c31daa6353d104bebc32adb31ff8c863beae7f02201580fdb8b1b8ae03777c06c7f19f42f0371f0b80ff57408917ff6b745cb06f2c[ALL] 0268f36ec62b3d1bb55d21ce1f002a8860aa6a8e982d1733aec352cad971c4e597",
        "hex": "473044022001b86ef00430bd9fd9d5ed69f9c31daa6353d104bebc32adb31ff8c863beae7f02201580fdb8b1b8ae03777c06c7f19f42f0371f0b80ff57408917ff6b745cb06f2c01210268f36ec62b3d1bb55d21ce1f002a8860aa6a8e982d1733aec352cad971c4e597"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 24.69349588,
      "valueSat": 2469349588,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 0026dcfac0cd2092ea5a124f2089c3ad5b7aa2cd OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9140026dcfac0cd2092ea5a124f2089c3ad5b7aa2cd88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "yLLFQTxaW3wybbahkhyZcrdfqoRCnfzAV5"
        ]
      }
    },
    {
      "value": 125.30495372,
      "valueSat": 12530495372,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 43634a1c7b1ebfa28a1f2fa5ec50d25f204f80f8 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91443634a1c7b1ebfa28a1f2fa5ec50d25f204f80f888ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "ySTm4oz74GJ8mPoTWMxNpyoM5GwSJyaA4z"
        ]
      }
    },
    {
      "value": 0.00000000,
      "valueSat": 0,
      "n": 2,
      "scriptPubKey": {
        "asm": "OP_RETURN 4f55543a30303533304241323336333135313546344335373538343830434344373232334142434244463745303030314341343641393138363142434437453234393334",
        "hex": "6a444f55543a30303533304241323336333135313546344335373538343830434344373232334142434244463745303030314341343641393138363142434437453234393334",
        "type": "nulldata"
      }
    }
  ]
}
```

So the rawtx looks fine to me. Time to bring out the big guns.
Thankfully Itzamna and crew have got debugging working.
I'll see if I can use their patch file and do the same...

It adds:

```
BUILD_FLAGS ... -gcflags 'all=-N -l

CGO_ENABLED=0 go build...

RUN go install github.com/go-delve/delve/cmd/dlv@latest

command: ["dlv", "--listen=:2345", "--headless=true", "--api-version=2", "exec", "/docker/scripts/bifrost", "--", "-p"]
```

Exposes port `2345` (might have to do a ssh reverse tunnel for this)

```
ssh -L 2345:127.0.0.1:2345 adc-vm-zt
```

Where is the `bifrost` binary within the container?
`/usr/bin/bifrost` if I understand the Dockerfile.
The debug script they've sent in the patch is actually calling the entrypoint script, not the go binary.
Does that still work??
In the interest of efficiency I'm just going to "fix" it to follow the delve docs.  
I removed the `command:` config from the docker compose, recreated it at the end of the `bifrost.sh` entrypoint script,
and set the `dlv exec` target to the bifrost binary instead of the startup script.

```
docker run -it --rm registry.gitlab.com/mayachain/mayanode:mocknet bash
```

```
docker run \
  --network=host \
  --rm \
  -e RUNE=MAYA.CACAO \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF=5s \
  -w /app \
  -v $(pwd)/test/smoke:/app \
    registry.gitlab.com/mayachain/mayanode:smoke \
    python scripts/smoke.py --fast-fail=True

# VVVVVVVVVVvv THIS COULD BE THE SECRET SAUCE vvVVVVVVVVVVVVVVVVvv
docker run \
  --network=host \
  --rm \
  -e RUNE=MAYA.CACAO \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -w /app \
  -v $(pwd)/test/smoke:/app \
  -e BLOCK_SCANNER_BACKOFF="" \
    registry.gitlab.com/mayachain/mayanode:smoke \
    python scripts/smoke.py --fast-fail=True

docker run \
  --network=host \
  --rm \
  -e RUNE=THOR.RUNE \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -e BLOCK_SCANNER_BACKOFF="" \
  -w /app \
  -v $(pwd)/test/smoke:/app \
    registry.gitlab.com/mayachain/mayanode:smoke \
    python scripts/smoke.py --fast-fail=True
```

Now stuck on test `40 PROVIDER-1 => VAULT      [SWAP:BTC.BTC:PROVIDER-1] 0.10000000 BTC/BTC`
For some reason.

```
make smoke-build-image
```

- setup env
- build containers
- make run-mocknet
- attach debugger to bifrost
- wait for curl p2pid
- run smoke container

Will it get past `40` if I set `RUNE=THOR.RUNE`?
I got past 40 before, just wondering what has changed.
I put the bifrost back into standard non debug mode.

Okay, `make smoke` got past 40
What's different with that??
I'm setting a block scanner backoff, they're not??

`make run-mocknet` does
`docker compose -f build/docker/docker-compose.yml --profile mocknet --profile midgard up -d`

`make smoke` does
```
docker compose -f build/docker/docker-compose.yml --profile mocknet --profile midgard down -v
docker compose -f build/docker/docker-compose.yml --profile mocknet --profile midgard build
docker compose -f build/docker/docker-compose.yml --profile mocknet --profile midgard up -d
docker pull registry.gitlab.com/mayachain/mayanode:smoke
docker buildx build \
  --cache-from registry.gitlab.com/mayachain/mayanode:smoke \
  -f test/smoke/Dockerfile \
  -t registry.gitlab.com/mayachain/mayanode:smoke \
  ./test/smoke
docker run \
  --network=host \
  --rm \
  -e RUNE=MAYA.CACAO \
  -e LOGLEVEL=INFO \
  -e PYTHONPATH=/app \
  -w /app \
  -v $(pwd)/test/smoke:/app \
  -e BLOCK_SCANNER_BACKOFF=${BLOCK_SCANNER_BACKOFF} \
  registry.gitlab.com/mayachain/mayanode:smoke \
  python scripts/smoke.py --fast-fail=True
```

```
docker inspect $(docker ps -a --format '{{.ID}} {{.Image}}' | grep smoke | awk '{print $1}') | jq ".[0].Image"
docker inspect docker-bifrost-1 --format '{{ .Image}}'
docker inspect docker-mayanode-1 --format '{{ .Image}}'
```

smoke image: 8d67669c1fc79a708f5589f962c2600d24a56ebeeec6ccba242bf146bb5e125d  
bifrost image: 3d0063c49810ed328bbb3894f1e4170a14a0fa0a36404570b9818ecb26687abe  
mayanode image: 3d0063c49810ed328bbb3894f1e4170a14a0fa0a36404570b9818ecb26687abe  

Trying:
- stop-mocknet
- run-mocknet
- pasting the docker run above

Could it be setting the BLOCK_SCANNER_BACKOFF to an empty string?

Think I found the issue.
The `createrawtransaction` command has changed

Arguments:
1. hexstring       (string, required) The hex string of the raw transaction
2. maxfeerate      (numeric or string, optional, default=0.10) Reject transactions whose fee rate is higher than the specified value, expressed in DASH/kB.
                   Set to 0 to accept any fee rate.

3. instantsend     (boolean) Deprecated and ignored
4. bypasslimits    (boolean, optional, default=false) Bypass transaction policy limits


{"jsonrpc":"1.0","method":"sendrawtransaction","params":["0100000001d1967b094eeefdc7cc8270b8435237eec7de4e630fc43cb4e9c655d3b1cdea98000000006b483045022100cf0bb29d2186b8dbaae096e0afdb5569ed549853ccf764397707bf383edeb50602205724c64e879f12f2fae4314ddbded4d05a604cf881b4af6ac667b366b4412dc6012103a65365f24006764255b2f5699f6b6172a3ccdd7fbde8c7449a20c442919b825fffffffff03d4482f93000000001976a9140026dcfac0cd2092ea5a124f2089c3ad5b7aa2cd88ac8c2fe0ea020000001976a914e5edf781b6a1efe79dda0667d0bae7f94106882f88ac0000000000000000466a444f55543a4542344441413943353931354634343938313533414545414230433538433431383939363845344638444538463137303243393738373335334243384641374300000000",true],"id":6695}

yep that's what's happening here
the second argument is set to true, but the node is expecting arg 2 to be either a number or a string.

Alright that seems very likely to be the issue here.

```
ssh -L 19898:127.0.0.1:19898 adc-vm-zt
```

```
version, err := c.BackendVersion()
```

Ah it could actually be an issue with `parseBitcoindVersion` in `rpcclient`.

I'm a little confused now.
I added my changes to fix the version parsing to branch `thorchain-integration2`
When I pulled, it fetched branches `thorchain-integration3` and `thorchain-integration5`
Need to work out what's going on here tomorrow...


### 12.06.2023 Monday 4h

Starting just the dash node on my vm.

```
docker compose up -d dash
docker logs -f docker-dash-1

docker exec -it docker-dash-1 bash

ssh -L 19898:127.0.0.1:19898 adc-vm-zt
```

Oh yeah `/Dash Core:19.1.0/` needs to be parsed correctly from `getnetworkinfo`.

```
github.com/alexdcox/dashd-go/rpcclient
```

```
go get github.com/alexdcox/dashd-go@hotfix/fix-version-parse
```

Doesn't like my branch formatting

```
go get github.com/alexdcox/dashd-go@fix-version-parse
```

```
go: downloading github.com/alexdcox/dashd-go v0.22.0-beta.0.20230612215533-bd68382c3d77
go: github.com/alexdcox/dashd-go@v0.22.0-beta.0.20230612215533-bd68382c3d77: parsing go.mod:
  module declares its path as: github.com/dashevo/dashd-go
          but was required as: github.com/alexdcox/dashd-go
```

Oh this looks familiar.

Go `replace` directive?

```
github.com/dashevo/dashd-go => github.com/alexdcox/dashd-go v0.22.0-beta.0.20230612215533-bd68382c3d77
```

What's the branch?
Oh I'm behind dashevo/master still, thought I checked that.
Pulled, rebased.

`bd68382c...784c860a`

Oh yeah I remember go doesn't like force push. Damn.
How to force go to fetch the latest commit again?
Just use the entire commit hash.

`go get -u github.com/alexdcox/dashd-go@784c860a0c0a701cf7f55e473144992fee0b9d10`

Then find the replace directive from the error:

`downloading github.com/alexdcox/dashd-go v0.22.0-beta.0.20230612220201-784c860a0c0a`

Then add to `go.mod`.

`github.com/dashpay/dashd-go => github.com/alexdcox/dashd-go v0.22.0-beta.0.20230612220201-784c860a0c0a`

Then go get the alias:

`go get github.com/dashpay/dashd-go`

Time to test...

```
github.com/dashpay/dashd-go/btcec: module github.com/dashpay/dashd-go@latest found (v0.24.0, replaced by github.com/alexdcox/dashd-go@v0.22.0-beta.0.20230612220201-784c860a0c0a), but does not contain package github.com/dashpay/dashd-go/btcec
```

github.com/dashpay/dashd-go/btcec
github.com/alexdcox/dashd-go/btcec

```
github.com/dashpay/dashd-go => github.com/alexdcox/dashd-go v0.22.0-beta.0.20230612220201-784c860a0c0a
github.com/dashpay/dashd-go/rpcclient => github.com/alexdcox/dashd-go/rpcclient v0.22.0-beta.0.20230612220201-784c860a0c0a
github.com/dashpay/dashd-go/btcec => github.com/alexdcox/dashd-go/btcec v0.22.0-beta.0.20230612220201-784c860a0c0a
```

> btcec/go.mod has post-v0 module path "github.com/dashpay/dashd-go/btcec/v2" at revision 784c860a0c0a

```
go get github.com/dashpay/dashd-go/btcec/v2
go get -u github.com/dashpay/dashd-go/chaincfg@784c860a0c0a701cf7f55e473144992fee0b9d10
```

There are broken tests within `dashd-go`.
addrmanager_internal_test.go:165

go get github.com/alexdcox/dashd-go@f8549e242eddb12a272b1d565d6e07d634b268e5

If I delete all traces of dashpay/dashd-go and run a go mod tidy, it is still
requiring it. This is making it hard to test.
Why is it required????

There are 4 go modules in the `dashd-go` repo:  
./go.mod  
./btcutil/go.mod (no dashpay in go.sum)  
./btcutil/psbt/go.mod (deleted dashpay directives, not in sum)  
./btcec/go.mod (not in sum)  

I've removed require dashpay directives from each one, and ensured they use the
correct local path replacement directive.

Also rebuilt go.sum for each.

It still says:

`go: found github.com/dashpay/dashd-go/btcec/v2/ecdsa in github.com/dashpay/dashd-go/btcec/v2 v2.0.0-00010101000000-000000000000`

Fixed broken test.
d893c2878507780d70b56510b7ab927c763cec9b

1. go get the hash (it will error)
2. use formatting from error to update replace directive
3. fetch the update for the package you've replaced by using the original name

github.com/alexdcox/dashd-go@v0.22.0-beta.0.20230612231255-d893c2878507

```
go get github.com/dashpay/dashd-go                                                                                                                                                                                                                                                                                                                                  ~/go/src/github.com/alexdcox/crypto-tools 1 ↵
go: downloading github.com/dashpay/dashd-go/btcutil v0.0.0-00010101000000-000000000000
github.com/dashpay/dashd-go imports
  github.com/dashpay/dashd-go/btcutil: github.com/dashpay/dashd-go/btcutil@v0.0.0-00010101000000-000000000000: invalid version: unknown revision 000000000000
github.com/dashpay/dashd-go imports
  github.com/dashpay/dashd-go/btcutil/bloom: github.com/dashpay/dashd-go/btcutil@v0.0.0-00010101000000-000000000000: invalid version: unknown revision 000000000000
github.com/dashpay/dashd-go imports
  github.com/dashpay/dashd-go/blockchain/indexers imports
  github.com/dashpay/dashd-go/btcutil/gcs: github.com/dashpay/dashd-go/btcutil@v0.0.0-00010101000000-000000000000: invalid version: unknown revision 000000000000
github.com/dashpay/dashd-go imports
  github.com/dashpay/dashd-go/blockchain/indexers imports
  github.com/dashpay/dashd-go/btcutil/gcs/builder: github.com/dashpay/dashd-go/btcutil@v0.0.0-00010101000000-000000000000: invalid version: unknown revision 000000000000
```

```
go get github.com/alexdcox/dashd-go@d76c4b53223db0e043c8ccfa6d8f3fc3ad88cf3b
go get github.com/alexdcox/dashutil@fe93ece8d0e7222ccd2544b3ff3bfd0c0d62e4bb
```

```
  pubKey := btcec.NewPublicKey(
    hexToFieldVal("d2e670a19c6d753d1a6d8b20bd045df8a08fb162cf508956c31268c6d81ffdab"),
    hexToFieldVal("ab65528eefbb8057aa85d597258a3fbd481a24633bc9b47a9aa045c91371de52"),
  )

  var f btcec.FieldVal
  if overflow := f.SetByteSlice(b); 
```

`bbb86436c6e0`

```
childXFieldVal := new(btcec.FieldVal)
childXFieldVal.SetByteSlice(childX.Bytes())
childYFieldVal := new(btcec.FieldVal)
childYFieldVal.SetByteSlice(childY.Bytes())
pk := btcec.NewPublicKey(childXFieldVal, childYFieldVal)
```

```
go get github.com/alexdcox/dashutil@fc4feafd54e5c0b63b791148645d09c39c7fa13c
```

> t.Fatal("ProcessTransaction: reported %d accepted

Right, finally managed to test it by creating a new branch and just following
the go conventions. 


### 13.06.2023 Tuesday 5h

Maya have opened up a new repo so I don't need to wait for DCG.

What's the best way to apply changes from one repo to another:
- apply patch of diff
- add remote

I used the remote approach:
- added local directory as remote
- created duplicate branch name fix-version-parse
- merged from alexdash/fix-version-parse (which was a fast forward merge as intended)
- pushed to remote

That's a nice workflow. So my dashd-go stays up-to-date, and the maya version
inherits those changes. No copying files, easy commands, can focus on essentially
one project/repo. Nice.

So now maya has `784c860a0c0a701cf7f55e473144992fee0b9d10`
Will try that in my crypto tools to send a raw transaction
Latest testnet block is `905217`

```
go get -u gitlab.com/mayachain/dashd-go@04869cb5
```

Updated import path.
Now getting the cyclic dependency warnings:

```
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/btcec/v2/ecdsa: reading gitlab.com/mayachain/dashd-go/btcec/go.mod at revision btcec/v2.1.0: unknown revision btcec/v2.1.0
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/btcutil: reading gitlab.com/mayachain/dashd-go/btcutil/go.mod at revision btcutil/v1.2.0: unknown revision btcutil/v1.2.0
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/btcutil/bloom: reading gitlab.com/mayachain/dashd-go/btcutil/go.mod at revision btcutil/v1.2.0: unknown revision btcutil/v1.2.0
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/blockchain imports
  gitlab.com/mayachain/dashd-go/btcec/v2: reading gitlab.com/mayachain/dashd-go/btcec/go.mod at revision btcec/v2.1.0: unknown revision btcec/v2.1.0
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/blockchain/indexers imports
  gitlab.com/mayachain/dashd-go/btcutil/gcs: reading gitlab.com/mayachain/dashd-go/btcutil/go.mod at revision btcutil/v1.2.0: unknown revision btcutil/v1.2.0
gitlab.com/mayachain/dashd-go imports
  gitlab.com/mayachain/dashd-go/blockchain/indexers imports
  gitlab.com/mayachain/dashd-go/btcutil/gcs/builder: reading gitlab.com/mayachain/dashd-go/btcutil/go.mod at revision btcutil/v1.2.0: unknown revision btcutil/v1.2.0
go: downloading golang.org/x/text v0.10.0
go: downloading golang.org/x/term v0.9.0
```

Time to remove those pesky submodules...

Temporarily removed golang actions on save: optimise imports and reformat code.
To keep these commits a little cleaner.

Replace `gitlab.com/mayachain/dashd-go/btcec/v2` with `gitlab.com/mayachain/dashd-go/btcec`

Fixed that broken test, again.
Now what?! Are we good?!!!

```
go get -u gitlab.com/mayachain/dashd-go@c884719a
```

Well that worked. Let's try some node communication...

I'm at 761K on my test node so another 150K or so before it will recognise any
payments from a faucet.

Can I get the testnet wallet on my phone again?
Yes.

```
export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"
```

Build > Build Tools > Gradle

error:
> Attribute meta-data#com.google.android.geo.API_KEY@value at
  AndroidManifest.xml:70:13-51 requires a placeholder substitution but no value
  for <GOOGLE_PLAY_API_KEY> is provided.


`build.gradle`

```
manifestPlaceholders = [
    GOOGLE_PLAY_API_KEY: "YOUR_API_KEY_HERE"
]
```

It wants me to use java 8. Fine.
openjdk@8: The x86_64 architecture is required for this software.

Maybe it's not fine.
Can't build via my m1 laptop.
ffs.
Onto my vm then...

```
dpkg --add-architecture i386
apt update
apt install git gradle openjdk-8-jdk libstdc++6:i386 zlib1g:i386
```

Now he wants me to install the android sdk and add to ANDROID_HOME. Why not add
that to the previous apt install list??

AM I GOING CRAZY HERE????!??? 🤯

```
apt install android-sdk
```

This was just thrown in next. Is that... a command from the project dir?
Am I suppose to recognise that?

`tools/android update sdk --no-ui --force --all --filter tool,platform-tool,build-tools-28,android-15,android-28`

Apparently the command line tools are not bundled with `android-sdk`.
That's why you can't get it with apt. Ok.
Why is that essential binary not included? Who knows?

```
wget https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip
unzip commandlinetools*
rm -rf commandlinetools*.zip
cmdline-tools
```

Error: Could not determine SDK root.
Error: Either specify it explicitly with --sdk_root= or move this package into its expected location: <sdk>/cmdline-tools/latest/

Okay so the format has changed with the newer sdkmanager than tools/android.
It's now:

```
./cmdline-tools/latest/bin/sdkmanager "tools" "platform-tools" "build-tools;28.0.3" "platforms;android-15" "platforms;android-28"
```

export ANDROID_HOME=/usr/lib/android-sdk
export ANDROID_NDK_HOME=/usr/lib/android-ndk/android-ndk-r25c

```
gradle clean assemble_testNet3Debug -x test
```

Whoa it's working.

Oh wait, spoke too soon:

```
FAILURE: Build failed with an exception.

* Where:
Build file '/code/dash-wallet/common/build.gradle' line: 1

* What went wrong:
A problem occurred evaluating project ':common'.
> Failed to apply plugin [id 'com.android.internal.version-check']
   > Minimum supported Gradle version is 7.3.3. Current version is 4.4.1. If using the gradle wrapper, try editing the distributionUrl in /code/dash-wallet/gradle/wrapper/gradle-wrapper.properties to gradle-7.3.3-all.zip

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 1m 33s
```

My gradle is 4.4.something
apt is out, sdkman was recommended

```
apt install zip
curl -s "https://get.sdkman.io" | bash
sdk install gradle 7.3.3
```

Take 2...

```
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':wallet'.
> Failed to notify project evaluation listener.
   > The file '/code/dash-wallet/local.properties' could not be found

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 22s
```

Ok I'll make it then I guess. Taking from presumably the autogenerated android studio version:

```
sdk.dir=/usr/lib/android-sdk
ndk.dir=/usr/lib/android-ndk/android-ndk-r25c
```

Take 3...

```
> Task :wallet:process_testNet3DebugMainManifest FAILED
/code/dash-wallet/wallet/AndroidManifest.xml:70:13-51 Error:
        Attribute meta-data#com.google.android.geo.API_KEY@value at AndroidManifest.xml:70:13-51 requires a placeholder substitution but no value for <GOOGLE_PLAY_API_KEY> is provided.
/code/dash-wallet/wallet/AndroidManifest.xml Error:
        Validation failed, exiting

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':wallet:process_testNet3DebugMainManifest'.
> Manifest merger failed with multiple errors, see logs

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 1m 50s
```

Okay back to my previous attempt at fixing this in `wallet/build.gradle`

Take 4...

Success! In only 4 takes too, was expecting at least double digits.

Okay so how do I actually build the apk now?

`gradle tasks`

```
gradle assemble_testNet3
gradle compile_testNet3DebugSources
```

FOUND IT

```
scp adc-vm-zt:/code/dash-wallet/wallet/build/outputs/apk/_testNet3/release/wallet-_testNet3-release-unsigned.apk .
adb install ./wallet-_testNet3-release-unsigned.apk
``` 

 Not sure when it was created, but there it is!
 
> adb: failed to install ./wallet-_testNet3-release-unsigned.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Failed to collect certificates from /data/app/vmdl416520446.tmp/base.apk: Attempt to get length of null array]
 

 ```
 scp adc-vm-zt:/code/dash-wallet/wallet/build/outputs/apk/_testNet3/debug/wallet-_testNet3-debug.apk .
 adb install ./wallet-_testNet3-debug.apk
 ```
 
 Well, it installed, and immediately crashed. Damn
 
Not going to waste more time on this and just wait for the testnet. 

 
### 14.06.2023 Wednesday 10h

Dash testnet down.
Trying with my llmq testnet from `thornode-cluster`.

Is there a way to expose a docker port to the host after it has been launched?

Yes, but it involves setting up ssh on the host machine and opening up a reverse
tunnel to `host.docker.internal`.

I'll just change the compose...

docker pull dashpay/dashd:19.1.0

https://insight.testnet.networks.dash.org:3002/insight/

For some reason the quorum is not syncing.

socket send error Broken pipe (32) (peer=435)

Right smokes are a smokin

Updating go.

```
/usr/lib/go-1.18
/usr/lib/go-1.20.5
cd /usr/lib
ln -s go-1.20.5 go
rm /usr/bin/go
ln -s /usr/lib/go/bin/go /usr/bin/go
```

```
export GOROOT=/usr/local/go
export GOPATH=~/.go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

```
# github.com/cosmos/cosmos-sdk/store/iavl
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/tree.go:10:11: cannot use (*immutableTree)(nil) (value of type *immutableTree) as Tree value in variable declaration: *immutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/tree.go:11:11: cannot use (*iavl.MutableTree)(nil) (value of type *"github.com/cosmos/iavl".MutableTree) as Tree value in variable declaration: *"github.com/cosmos/iavl".MutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:69:9: cannot use tree (variable of type *"github.com/cosmos/iavl".MutableTree) as Tree value in struct literal: *"github.com/cosmos/iavl".MutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:81:9: cannot use tree (variable of type *"github.com/cosmos/iavl".MutableTree) as Tree value in struct literal: *"github.com/cosmos/iavl".MutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:92:23: cannot use &immutableTree{…} (value of type *immutableTree) as Tree value in struct literal: *immutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:101:9: cannot use &immutableTree{…} (value of type *immutableTree) as Tree value in struct literal: *immutableTree does not implement Tree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:204:7: impossible type switch case: *immutableTree
  st.tree (variable of type Tree) cannot have dynamic type *immutableTree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:206:7: impossible type switch case: *iavl.MutableTree
  st.tree (variable of type Tree) cannot have dynamic type *"github.com/cosmos/iavl".MutableTree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:218:7: impossible type switch case: *immutableTree
  st.tree (variable of type Tree) cannot have dynamic type *immutableTree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
/go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.1/store/iavl/store.go:220:7: impossible type switch case: *iavl.MutableTree
  st.tree (variable of type Tree) cannot have dynamic type *"github.com/cosmos/iavl".MutableTree (wrong type for method Get)
    have Get([]byte) ([]byte, error)
    want Get([]byte) (int64, []byte)
```

My cosmos sdk slipped back a few minor versions somehow.
Redoing go mod tidy with that.
Rerunning mocknet tests.
Pushing.
Rebuilding.
Smoking.

`go test -tags mocknet ./bifrost/pkg/chainclients/dash`

Maya team added the createwallet steps but didn't create a mock response, so the
tests failed.

Not sure if they're handling the case where the wallet has already been created -
 dashd throws an error if you recreate.

Not able to build the smoke image atm:

```
#0 2.409 E: Unable to locate package libsecp256k1-0
------
ERROR: failed to solve: executor failed running [/bin/sh -c apt-get update && apt-get install -y libsecp256k1-0 build-essential git]: exit code: 100
make: *** [Makefile:163: smoke-build-image] Error 1
```

`libsecp256k1-0` is now `libsecp256k1-1`.

```
Traceback (most recent call last):
  File "/usr/local/lib/python3.10/site-packages/web3/contract.py", line 1498, in call_contract_function
    output_data = web3.codec.decode_abi(output_types, return_data)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/codec.py", line 196, in decode_abi
    return self.decode(types, data)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/codec.py", line 210, in decode
    return decoder(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 127, in __call__
    return self.decode(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_utils/functional.py", line 45, in inner
    return callback(fn(*args, **kwargs))
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 173, in decode
    yield decoder(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 127, in __call__
    return self.decode(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 142, in decode
    start_pos = decode_uint_256(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 127, in __call__
    return self.decode(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 198, in decode
    raw_data = self.read_data_from_stream(stream)
  File "/usr/local/lib/python3.10/site-packages/eth_abi/decoding.py", line 305, in read_data_from_stream
    raise InsufficientDataBytes(
eth_abi.exceptions.InsufficientDataBytes: Tried to read 32 bytes.  Only got 0 bytes

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/app/scripts/smoke.py", line 690, in <module>
    main()
  File "/app/scripts/smoke.py", line 123, in main
    smoker = Smoker(
  File "/app/scripts/smoke.py", line 231, in __init__
    self.mock_ethereum = MockEthereum(eth)
  File "/app/chains/ethereum.py", line 62, in __init__
    symbol = token.functions.symbol().call()
  File "/usr/local/lib/python3.10/site-packages/web3/contract.py", line 949, in call
    return call_contract_function(
  File "/usr/local/lib/python3.10/site-packages/web3/contract.py", line 1520, in call_contract_function
    raise BadFunctionCallOutput(msg) from e
web3.exceptions.BadFunctionCallOutput: Could not transact with/call contract function, is contract deployed correctly and chain synced?
```

What in the name of our sweet sweet lord is that?
Let's pretend that didn't happen and try again.

```
I[2023-06-14 22:22:48,903] 91 PROVIDER-1 => VAULT      [WITHDRAW:DASH.DASH:100] 0.00000001 MAYA.CACAO
E[2023-06-14 22:23:02,185] out coins not matching 134997762 DASH.DASH != 134997610 DASH.DASH
E[2023-06-14 22:23:53,335] Events mismatch
```

Oh that's painfully close.
Recreating smoke values.

```
make smoke-unit-test
```

```
I[2023-06-14 23:00:28,672] jsonpickle is not installed. The to_json_pickle and from_json_pickle functions will not work.If you dont need those functions, there is nothing to do.
```
It's complaining about jsonpickle not being installed.
Might have to add this to requirements?

```
jsonpickle==3.0.1
```

Ok maya know the smoke unit tests are broken, but didn't realise they generate
the events/balances, and were creating those by hand.

```
I[2023-06-15 00:15:32,533] 91 PROVIDER-1 => VAULT      [WITHDRAW:DASH.DASH:100] 0.00000001 MAYA.CACAO
E[2023-06-15 00:15:46,326] out coins not matching 135249474 DASH.DASH != 135249322 DASH.DASH
E[2023-06-15 00:16:37,477] Events mismatch

E[2023-06-15 00:16:37,477] Smoke tests failed
Traceback (most recent call last):
  File "/app/scripts/smoke.py", line 143, in main
    smoker.run()
  File "/app/scripts/smoke.py", line 656, in run
    outbounds, block_height, events, sim_events = self.sim_catch_up(
  File "/app/scripts/smoke.py", line 575, in sim_catch_up
    self.check_events(events[-count_events:],
  File "/app/scripts/smoke.py", line 385, in check_events
    self.error("Events mismatch\n")
  File "/app/scripts/smoke.py", line 256, in error
    raise Exception(err)
Exception: Events mismatch

>>>>>>>>>>>>>>> MISSING SIM EVENT
Event outbound | {'in_tx_id': 'EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D'} {'id': 'F3950E859533D53CE4AA38D6DAD9E9E4CF02C90AD2AD9843BF007A63BDA31E7B'} {'chain': 'DASH'} {'from': 'yQqcxBiZ9b5V1Z8XpS4vd1k6kwCaK7hcCY'} {'to': 'yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L'} {'coin': '135249322 DASH.DASH'} {'memo': 'OUT:EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D'}
<<<<<<<<<<<<<<< EXTRA SIM EVENT
Event gas | {'asset': 'DOGE.DOGE'} {'asset_amt': '750000000'} {'cacao_amt': '2779042719'} {'transaction_count': '1'}
make: *** [Makefile:171: smoke] Error 1
```

```
12:15AM INF app/x/mayachain/handler_deposit.go:59 > receive MsgDeposit coins=[{"amount":"1","asset":"MAYA.CACAO"}] from=tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzw memo=WITHDRAW:DASH.DASH:100
12:15AM INF app/x/mayachain/handler_withdraw.go:33 > receive MsgWithdrawLiquidity withdraw address=tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzw withdraw basis points=100
12:15AM INF app/x/mayachain/withdraw_current.go:74 > pool before withdraw balance RUNE=387454601955 balance asset=13530495372 pool units=100000000000
12:15AM INF app/x/mayachain/withdraw_current.go:75 > liquidity provider before withdraw liquidity provider unit=100000000000
12:15AM INF app/x/mayachain/withdraw_current.go:105 > imp loss calculation deposit value=529534829992 protection=0 redeem value=774909203910
12:15AM INF app/x/mayachain/withdraw_current.go:169 > client withdraw RUNE=3874546020 asset=135304954 units left=99000000000
12:15AM INF app/x/mayachain/withdraw_current.go:175 > pool after withdraw balance RUNE=383580055935 balance asset=13395190418 pool unit=99000000000
12:15AM INF app/x/mayachain/handler_outbound_tx.go:50 > receive MsgOutboundTx request outbound tx hash=0000000000000000000000000000000000000000000000000000000000000000
12:15AM INF go/pkg/mod/github.com/tendermint/tendermint@v0.34.21/state/execution.go:333 > executed block height=168 module=state num_invalid_txs=0 num_valid_txs=1
12:15AM INF go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.9/baseapp/abci.go:315 > commit synced commit=436F6D6D697449447B5B383520363320393820343020313132203231362034342031343620323436203830203731203336203234312031353020353120313520323431203130302032323620313635203230372031313120323438203130392031343920363020313333203230342037322031343520323534203131365D3A41387D
12:15AM INF go/pkg/mod/github.com/tendermint/tendermint@v0.34.21/state/execution.go:235 > committed state app_hash=553F622870D82C92F6504724F196330FF164E2A5CF6FF86D953C85CC4891FE74 height=168 module=state num_txs=1
12:15AM INF go/pkg/mod/github.com/tendermint/tendermint@v0.34.21/state/execution.go:333 > executed block height=169 module=state num_invalid_txs=0 num_valid_txs=0
12:15AM INF go/pkg/mod/github.com/cosmos/cosmos-sdk@v0.45.9/baseapp/abci.go:315 > commit synced commit=436F6D6D697449447B5B322032333020323334203138382031343920323230203435203133203938203136312039342031373620313432203134203130322031303420313130203232372036302032333520323437203132322032343120383620313335203332203530203137382037332032343820323532203231355D3A41397D
12:15AM INF go/pkg/mod/github.com/tendermint/tendermint@v0.34.21/state/execution.go:235 > committed state app_hash=02E6EABC95DC2D0D62A15EB08E0E66686EE33CEBF77AF156872032B249F8FCD7 height=169 module=state num_txs=0
12:15AM INF app/x/mayachain/handler_network_fee.go:112 > update network fee chain=DASH fee-rate=15 transaction-size=304
12:15AM INF app/x/mayachain/handler_observed_txout.go:171 > handleMsgObservedTxOut request Tx:="F3950E859533D53CE4AA38D6DAD9E9E4CF02C90AD2AD9843BF007A63BDA31E7B: yQqcxBiZ9b5V1Z8XpS4vd1k6kwCaK7hcCY ==> yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L (Memo: OUT:EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D) 135249322 DASH.DASH (gas: [27816 DASH.DASH])"
12:15AM INF app/x/mayachain/handler_outbound_tx.go:50 > receive MsgOutboundTx request outbound tx hash=F3950E859533D53CE4AA38D6DAD9E9E4CF02C90AD2AD9843BF007A63BDA31E7B
```

```
12:15AM INF app/bifrost/mayaclient/broadcast.go:100 > Received a TxHash of 2DD7B26B2A660BC2CFD96451FD3DE4BEE29339346973925D116B888EF802DC9C from the mayachain module=mayachain_client service=bifrost
12:15AM INF app/bifrost/observer/observe.go:516 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=2DD7B26B2A660BC2CFD96451FD3DE4BEE29339346973925D116B888EF802DC9C
12:15AM INF app/bifrost/signer/sign.go:285 > Received a TxOut Array of {168 [{DASH yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L tmayapub1addwnpepqfsqkm03rnl0425y77uvfmt2wp6lmn22mgq6n55d3f0p3wl3hx5rgnt59kv 13395218234 DASH.DASH OUT:EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D [27816 DASH.DASH] 91 EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D    <nil>}]} from the Thorchain module=signer service=bifrost
12:15AM INF app/bifrost/signer/sign.go:229 > Signing transaction height=168 module=signer num=0 service=bifrost status=1 tx={"aggregator":"","chain":"DASH","coins":[{"amount":"135249322","asset":"DASH.DASH"}],"gas_rate":91,"in_hash":"EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D","max_gas":[{"amount":"27816","asset":"DASH.DASH","decimals":8}],"memo":"OUT:EEFA12A4189E20C99D5B51B03A47A099B35C448E847E06A945072FD48FF9A79D","out_hash":"","to":"yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L","vault_pubkey":"tmayapub1addwnpepqfsqkm03rnl0425y77uvfmt2wp6lmn22mgq6n55d3f0p3wl3hx5rgnt59kv"}
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:296 > max gas: [27816 DASH.DASH], however estimated gas need 41041 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:327 > total: 13530495372, to customer: 135249322, gas: 27816 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:332 > send 13395218234 back to self module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:427 > UTXOs to sign: 2 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:451 > final size: 451, final vbyte: 451 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/signer.go:527 > broadcast to DASH chain successfully hash=f3950e859533d53ce4aa38d6dad9e9e4cf02c90ad2ad9843bf007a63bda31e7b module=dash service=bifrost
12:15AM INF app/bifrost/mayaclient/broadcast.go:46 > account info account_number=0 module=mayachain_client sequence_number=98 service=bifrost
12:15AM INF app/bifrost/mayaclient/broadcast.go:100 > Received a TxHash of 7F828ABEC772F4BC986FF888C0DEDAD9A7248721B61E1CFE2D62E760C13C1095 from the mayachain module=mayachain_client service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/client.go:1017 > totalTxValue:0,total fee and Subsidy:34518104363,confirmation:0 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/client.go:1042 > confirmation required: 0 module=dash service=bifrost
12:15AM INF app/bifrost/pkg/chainclients/dash/client.go:1060 > confirmation required: 0 module=dash service=bifrost
12:15AM INF app/bifrost/mayaclient/broadcast.go:46 > account info account_number=0 module=mayachain_client sequence_number=99 service=bifrost
12:15AM INF app/bifrost/mayaclient/broadcast.go:100 > Received a TxHash of E184771039A3B953D6FA8C68DE755D94CBEECA8F4AA1A4677D87DE66A06E7DD8 from the mayachain module=mayachain_client service=bifrost
12:15AM INF app/bifrost/observer/observe.go:516 > sign and send to thorchain successfully module=observer service=bifrost thorchain hash=E184771039A3B953D6FA8C68DE755D94CBEECA8F4AA1A4677D87DE66A06E7DD8
```

```
dash-cli gettransaction f3950e859533d53ce4aa38d6dad9e9e4cf02c90ad2ad9843bf007a63bda31e7b
```

```
{
  "amount": 1.35249322,
  "confirmations": 31,
  "instantlock": false,
  "instantlock_internal": false,
  "chainlock": false,
  "blockhash": "72c6392c70bec7bb53debeb886a5ff3f17f4dcc2e1b5cca3a48b78f0e6c38bdd",
  "blockindex": 5,
  "blocktime": 1686788143,
  "txid": "f3950e859533d53ce4aa38d6dad9e9e4cf02c90ad2ad9843bf007a63bda31e7b",
  "walletconflicts": [
  ],
  "time": 1686788142,
  "timereceived": 1686788142,
  "details": [
    {
      "involvesWatchonly": true,
      "address": "yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L",
      "category": "receive",
      "amount": 1.35249322,
      "label": "",
      "vout": 0
    }
  ],
  "hex": "010000000260cd1f5035172a25dbe2b8a5e3c347c68b579d77a62ba2aad62b2d25d626167e010000006a47304402201387c7491d596d60a92b62361aa0bc6cec55ce23695554701e329f99ac32fd530220249a144f552500935b5fd9f62e735c7952e5772167b70adbd16ed5a8444c7629012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834ffffffff8e767f3a8680d1225c4b1ace04b03cdeab1d2c9d3857c55234854a7d39ae8ad2000000006a473044022065390049499eeca9f7cdefc0de5f15bf77a325a3978e66e8e513632d9576682d0220410c192a6378d30087b29561f429daec1be6fac61a88649d205e5ca835311600012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834ffffffff03aabd0f08000000001976a9147c2bb42a8be69791ec763e51f5a49bcd41e8223788ac3acf6a1e030000001976a91431956a696d7415b47ed02919814422e9aa15c25288ac0000000000000000466a444f55543a4545464131324134313839453230433939443542353142303341343741303939423335433434384538343745303641393435303732464434384646394137394400000000"
}
```

```
{
  "txid": "f3950e859533d53ce4aa38d6dad9e9e4cf02c90ad2ad9843bf007a63bda31e7b",
  "version": 1,
  "type": 0,
  "size": 451,
  "locktime": 0,
  "vin": [
    {
      "txid": "7e1626d6252d2bd6aaa22ba6779d578bc647c3e3a5b8e2db252a1735501fcd60",
      "vout": 1,
      "scriptSig": {
        "asm": "304402201387c7491d596d60a92b62361aa0bc6cec55ce23695554701e329f99ac32fd530220249a144f552500935b5fd9f62e735c7952e5772167b70adbd16ed5a8444c7629[ALL] 02600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834",
        "hex": "47304402201387c7491d596d60a92b62361aa0bc6cec55ce23695554701e329f99ac32fd530220249a144f552500935b5fd9f62e735c7952e5772167b70adbd16ed5a8444c7629012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834"
      },
      "value": 125.30495372,
      "valueSat": 12530495372,
      "address": "yQqcxBiZ9b5V1Z8XpS4vd1k6kwCaK7hcCY",
      "sequence": 4294967295
    },
    {
      "txid": "d28aae397d4a853452c557389d2c1dabde3cb004ce1a4b5c22d180863a7f768e",
      "vout": 0,
      "scriptSig": {
        "asm": "3044022065390049499eeca9f7cdefc0de5f15bf77a325a3978e66e8e513632d9576682d0220410c192a6378d30087b29561f429daec1be6fac61a88649d205e5ca835311600[ALL] 02600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834",
        "hex": "473044022065390049499eeca9f7cdefc0de5f15bf77a325a3978e66e8e513632d9576682d0220410c192a6378d30087b29561f429daec1be6fac61a88649d205e5ca835311600012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834"
      },
      "value": 10.00000000,
      "valueSat": 1000000000,
      "address": "yQqcxBiZ9b5V1Z8XpS4vd1k6kwCaK7hcCY",
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 1.35249322,
      "valueSat": 135249322,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 7c2bb42a8be69791ec763e51f5a49bcd41e82237 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9147c2bb42a8be69791ec763e51f5a49bcd41e8223788ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "yXdzzagQPwWXdZzFD2vRmsYMxgeD6o9D2L"
        ]
      }
    },
    {
      "value": 133.95218234,
      "valueSat": 13395218234,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 31956a696d7415b47ed02919814422e9aa15c252 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91431956a696d7415b47ed02919814422e9aa15c25288ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "yQqcxBiZ9b5V1Z8XpS4vd1k6kwCaK7hcCY"
        ]
      }
    },
    {
      "value": 0.00000000,
      "valueSat": 0,
      "n": 2,
      "scriptPubKey": {
        "asm": "OP_RETURN 4f55543a45454641313241343138394532304339394435423531423033413437413039394233354334343845383437453036413934353037324644343846463941373944",
        "hex": "6a444f55543a45454641313241343138394532304339394435423531423033413437413039394233354334343845383437453036413934353037324644343846463941373944",
        "type": "nulldata"
      }
    }
  ],
  "hex": "010000000260cd1f5035172a25dbe2b8a5e3c347c68b579d77a62ba2aad62b2d25d626167e010000006a47304402201387c7491d596d60a92b62361aa0bc6cec55ce23695554701e329f99ac32fd530220249a144f552500935b5fd9f62e735c7952e5772167b70adbd16ed5a8444c7629012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834ffffffff8e767f3a8680d1225c4b1ace04b03cdeab1d2c9d3857c55234854a7d39ae8ad2000000006a473044022065390049499eeca9f7cdefc0de5f15bf77a325a3978e66e8e513632d9576682d0220410c192a6378d30087b29561f429daec1be6fac61a88649d205e5ca835311600012102600b6df11cfefaaa84f7b8c4ed6a7075fdcd4ada01a9d28d8a5e18bbf1b9a834ffffffff03aabd0f08000000001976a9147c2bb42a8be69791ec763e51f5a49bcd41e8223788ac3acf6a1e030000001976a91431956a696d7415b47ed02919814422e9aa15c25288ac0000000000000000466a444f55543a4545464131324134313839453230433939443542353142303341343741303939423335433434384538343745303641393435303732464434384646394137394400000000",
  "blockhash": "72c6392c70bec7bb53debeb886a5ff3f17f4dcc2e1b5cca3a48b78f0e6c38bdd",
  "height": 879,
  "confirmations": 391,
  "time": 1686788143,
  "blocktime": 1686788143,
  "instantlock": false,
  "instantlock_internal": false,
  "chainlock": false
}
```

```
var a = [{"chain":"BNB","from_address":"MASTER","to_address":"CONTRIB","memo":"SEED","coins":[{"asset":"BNB.BNB","amount":1000000000}],"gas":null,"max_gas":null},{"chain":"BNB","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"BNB.BNB","amount":2600000000},{"asset":"BNB.LOK-3C0","amount":150000000000}],"gas":null,"max_gas":null},{"chain":"BNB","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"BNB.BNB","amount":1000000000},{"asset":"BNB.LOK-3C0","amount":80000000000}],"gas":null,"max_gas":null},{"chain":"BNB","from_address":"MASTER","to_address":"PROVIDER-2","memo":"SEED","coins":[{"asset":"BNB.BNB","amount":200000000},{"asset":"BNB.LOK-3C0","amount":10000000000}],"gas":null},{"chain":"BTC","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"BTC.BTC","amount":200000000}],"gas":null},{"chain":"BTC","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"BTC.BTC","amount":500000000}],"gas":null},{"chain":"DOGE","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"DOGE.DOGE","amount":20000000000}],"gas":null},{"chain":"DOGE","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"DOGE.DOGE","amount":200000000000}],"gas":null},{"chain":"GAIA","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"GAIA.ATOM","amount":50000000000000}],"gas":null},{"chain":"GAIA","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"GAIA.ATOM","amount":50000000000000}],"gas":null},{"chain":"DASH","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"DASH.DASH","amount":2000000000}],"gas":null},{"chain":"DASH","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"DASH.DASH","amount":20000000000}],"gas":null},{"chain":"BTC","from_address":"MASTER","to_address":"PROVIDER-2","memo":"SEED","coins":[{"asset":"BTC.BTC","amount":500000000}],"gas":null},{"chain":"BCH","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"BCH.BCH","amount":200000000}],"gas":null},{"chain":"BCH","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"BCH.BCH","amount":200000000}],"gas":null},{"chain":"BCH","from_address":"MASTER","to_address":"PROVIDER-2","memo":"SEED","coins":[{"asset":"BCH.BCH","amount":200000000}],"gas":null},{"chain":"LTC","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"LTC.LTC","amount":200000000}],"gas":null},{"chain":"LTC","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"LTC.LTC","amount":200000000}],"gas":null},{"chain":"LTC","from_address":"MASTER","to_address":"PROVIDER-2","memo":"SEED","coins":[{"asset":"LTC.LTC","amount":200000000}],"gas":null},{"chain":"ETH","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"ETH.ETH","amount":40000000000000000000}],"gas":null},{"chain":"ETH","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"ETH.ETH","amount":40000000000000000000}],"gas":null},{"chain":"ETH","from_address":"MASTER","to_address":"USER-1","memo":"SEED","coins":[{"asset":"ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A","amount":400000000000000000000}],"gas":null},{"chain":"ETH","from_address":"MASTER","to_address":"PROVIDER-1","memo":"SEED","coins":[{"asset":"ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A","amount":400000000000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1-SYNTH","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BNB.BNB:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":10000000000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BNB.BNB:PROVIDER-1","coins":[{"asset":"BNB.BNB","amount":250000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:DOGE.DOGE:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":100000000000}],"gas":null},{"chain":"DOGE","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:DOGE.DOGE:PROVIDER-1","coins":[{"asset":"DOGE.DOGE","amount":150000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:GAIA.ATOM:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":100000000000}],"gas":null},{"chain":"GAIA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:GAIA.ATOM:PROVIDER-1","coins":[{"asset":"GAIA.ATOM","amount":150000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:DASH.DASH:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":100000000000}],"gas":null},{"chain":"DASH","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:DASH.DASH:PROVIDER-1","coins":[{"asset":"DASH.DASH","amount":15000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1-SYNTH","coins":[{"asset":"MAYA.CACAO","amount":10000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB/BNB:PROVIDER-1-SYNTH","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:PROVIDER-1","coins":[{"asset":"BNB/BNB","amount":10}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1-SYNTH","coins":[{"asset":"BNB.BNB","amount":100000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB/BNB:PROVIDER-1-SYNTH","coins":[{"asset":"BNB.BNB","amount":100000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BTC.BTC:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":10000000000000}],"gas":null},{"chain":"BTC","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BTC.BTC:PROVIDER-1","coins":[{"asset":"BTC.BTC","amount":250000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BTC/BTC:PROVIDER-1-SYNTH","coins":[{"asset":"BNB/BNB","amount":100000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BTC.BTC:PROVIDER-1","coins":[{"asset":"BTC/BTC","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1","coins":[{"asset":"BTC/BTC","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1","coins":[{"asset":"BTC/BTC","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BTC.BTC:PROVIDER-1-SYNTH","coins":[{"asset":"BTC/BTC","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:BTC/BTC:PROVIDER-1-SYNTH","coins":[{"asset":"BTC/BTC","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BCH.BCH:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"BCH","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BCH.BCH:PROVIDER-1","coins":[{"asset":"BCH.BCH","amount":150000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:LTC.LTC:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"LTC","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:LTC.LTC:PROVIDER-1","coins":[{"asset":"LTC.LTC","amount":150000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:ETH.ETH:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"ETH","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:ETH.ETH:PROVIDER-1","coins":[{"asset":"ETH.ETH","amount":400000000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"ETH","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:PROVIDER-1","coins":[{"asset":"ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A","amount":4000000000000000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BNB.LOK-3C0:PROVIDER-1","coins":[{"asset":"BNB.LOK-3C0","amount":40000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:BNB.LOK-3C0:PROVIDER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-2","to_address":"VAULT","memo":"","coins":[{"asset":"BNB.BNB","amount":150000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-2","to_address":"VAULT","memo":"ABDG?","coins":[{"asset":"BNB.BNB","amount":150000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-1","to_address":"VAULT","memo":"ADD:","coins":[{"asset":"BNB.BNB","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-2","to_address":"VAULT","memo":"ADD:MAYA.CACAO","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-2","to_address":"VAULT","memo":"ADD:BNB.BNB:PROVIDER-2","coins":[{"asset":"MAYA.CACAO","amount":3000000000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-2","to_address":"VAULT","memo":"ADD:BNB.BNB:PROVIDER-2","coins":[{"asset":"BNB.BNB","amount":30000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-2","to_address":"VAULT","memo":"ADD:BNB.BNB:PROVIDER-2","coins":[{"asset":"BNB.BNB","amount":90000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-2","to_address":"VAULT","memo":"DONATE:BNB.BNB","coins":[{"asset":"MAYA.CACAO","amount":500000000000}],"gas":null},{"chain":"BNB","from_address":"PROVIDER-2","to_address":"VAULT","memo":"DONATE:BNB.BNB","coins":[{"asset":"BNB.BNB","amount":30000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":" ","coins":[{"asset":"MAYA.CACAO","amount":20000000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"ABDG?","coins":[{"asset":"MAYA.CACAO","amount":20000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB","coins":[{"asset":"BNB.BNB","amount":30000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BTC.BTC:USER-1","coins":[{"asset":"BNB.BNB","amount":500000000}],"gas":null},{"chain":"BTC","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"BTC.BTC","amount":1500000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:DOGE.DOGE:USER-1","coins":[{"asset":"BNB.BNB","amount":100000000}],"gas":null},{"chain":"DOGE","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"DOGE.DOGE","amount":10000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:GAIA.ATOM:USER-1","coins":[{"asset":"BNB.BNB","amount":100000000}],"gas":null},{"chain":"GAIA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"GAIA.ATOM","amount":10000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:DASH.DASH:USER-1","coins":[{"asset":"BNB.BNB","amount":100000000}],"gas":null},{"chain":"DASH","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"DASH.DASH","amount":1000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BCH.BCH:USER-1","coins":[{"asset":"BNB.BNB","amount":500000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:LTC.LTC:USER-1","coins":[{"asset":"BNB.BNB","amount":500000000}],"gas":null},{"chain":"LTC","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"LTC.LTC","amount":1500000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:ETH.ETH:USER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"ETH","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"ETH.ETH","amount":150000000000000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:USER-1","coins":[{"asset":"MAYA.CACAO","amount":5000000000000}],"gas":null},{"chain":"BCH","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"BCH.BCH","amount":1500000}],"gas":null},{"chain":"ETH","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A","amount":150000000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:ETH.ETH:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB","coins":[{"asset":"BNB.BNB","amount":30000000},{"asset":"BNB.TCAN-014","amount":1000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:USER-1","coins":[{"asset":"MAYA.CACAO","amount":10000100000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:USER-1:26572599","coins":[{"asset":"MAYA.CACAO","amount":1000000000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:USER-1","coins":[{"asset":"MAYA.CACAO","amount":1000000000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BTC.BTC:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:DOGE.DOGE:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:DASH.DASH:100","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:GAIA.ATOM:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BCH.BCH:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:LTC.LTC:1000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"BNB.BNB","amount":10000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1:23853375","coins":[{"asset":"MAYA.CACAO","amount":1000000000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB:PROVIDER-1:22460886","coins":[{"asset":"MAYA.CACAO","amount":1000000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:bnbPROVIDER-1","coins":[{"asset":"BNB.BNB","amount":10000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:USER-1","coins":[{"asset":"BNB.LOK-3C0","amount":5000000000}],"gas":null},{"chain":"MAYA","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.LOK-3C0:USER-1","coins":[{"asset":"MAYA.CACAO","amount":500000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.BNB","coins":[{"asset":"BNB.LOK-3C0","amount":5000000000}],"gas":null},{"chain":"BNB","from_address":"USER-1","to_address":"VAULT","memo":"SWAP:BNB.LOK-3C0","coins":[{"asset":"BNB.BNB","amount":5000000}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BNB.BNB:5000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BNB.LOK-3C0","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BNB.BNB:10000","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:ETH.TKN-0X40BCD4DB8889A8BF0B1391D0C819DCD9627F9D0A","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:ETH.ETH","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:PROVIDER-1","coins":[{"asset":"BNB/BNB","amount":40493464}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"SWAP:MAYA.CACAO:PROVIDER-1","coins":[{"asset":"BTC/BTC","amount":2206199}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BTC.BTC","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:DOGE.DOGE","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:DASH.DASH","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:GAIA.ATOM","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:BCH.BCH","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null},{"chain":"MAYA","from_address":"PROVIDER-1","to_address":"VAULT","memo":"WITHDRAW:LTC.LTC","coins":[{"asset":"MAYA.CACAO","amount":1}],"gas":null}]
var b = a.filter(a => ["DASH", "MAYA"].includes(a.chain) && ["BNB", "DOGE", "GAIA", "BTC", "BCH", "LTC", "ETH"].every(x => a.memo.match(x) == null))
console.log(JSON.stringify(b, null, '  '))
```

That gets me close, but still needed to trip a few more out.

Looked back at my thorchain notes. The smoke tests run maya + nodes against the
python emulator, which does the same thing in python. There's also the event
checking, and the balance checking, but if you get out coins mismatch, it's
nothing to do with the cached/generated/exported json data in events/balances.
That's what threw me again this time.

So. 451. I think that's the issue here.
The dash node has changed and if I remember correctly the smoke tests fail if
the python emulator doesn't "estimate" the size of each transaction 100%
accurately.
Makes total sense, right.
Maybe there's more data per tx now than there used to be, so we're getting 451,
a size larger than the python generator is expecting, which is causing the fees
to be calculated wrong, which is the cause of this mismatch.
That's my guess.

Now... where to begin.

This is where having working smoke tests is critical.
I want to strip all transaction away apart from dash to test this fast.

Found this in `smoke_test_balances.json`. Oops.

```
<<<<<< HEAD
            "BCH.BCH": 118168948,
            "MAYA.CACAO": 72021727732
        },
        "POOL.LTC.LTC": {
            "LTC.LTC": 125296107,
            "MAYA.CACAO": 63807888857
        },
        "POOL.ETH.ETH": {
            "ETH.ETH": 30075528,
            "MAYA.CACAO": 99747310638
=======
            "BCH.BCH": 150000000,
            "THOR.RUNE": 50000000000
        },
        "POOL.LTC.LTC": {
            "LTC.LTC": 150000000,
            "THOR.RUNE": 50000000000
        },
        "POOL.ETH.ETH": {
            "ETH.ETH": 40000000,
            "THOR.RUNE": 50000000000
>>>>>>> 1dd40947d (Update smoke test balances/events)

```

You can set this in the python unit tests to see the full events:
```
self.maxDiff = None
```

```
Exception: Cannot transfer. No DASH UTXO available for yZnyJAdouDu3gmAuhG3dTc66hroS4AXxnL
```

That address is the MASTER address.
By the time I checked it, it did have utxos available.
Perhaps the smoke tests just aren't waiting long enough.

Yeah waiting manually and then just running the tests by hand made it work.
Block 13 is about right.
Now getting this.
I'm probably off course here.

```
2:20AM INF app/x/mayachain/handler_deposit.go:59 > receive MsgDeposit coins=[{"amount":"1","asset":"MAYA.CACAO"}] from=tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzw memo=WITHDRAW:DASH.DASH:10
2:20AM INF app/x/mayachain/handler_withdraw.go:33 > receive MsgWithdrawLiquidity withdraw address=tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzw withdraw basis points=10
2:20AM INF app/x/mayachain/withdraw_current.go:74 > pool before withdraw balance RUNE=93750000000 balance asset=16000000000 pool units=100000000000
2:20AM INF app/x/mayachain/withdraw_current.go:75 > liquidity provider before withdraw liquidity provider unit=100000000000
2:20AM INF app/x/mayachain/withdraw_current.go:105 > imp loss calculation deposit value=187890625000 protection=0 redeem value=187500000000
2:20AM INF app/x/mayachain/withdraw_current.go:169 > client withdraw RUNE=93750000 asset=16000000 units left=99900000000
2:20AM INF app/x/mayachain/withdraw_current.go:175 > pool after withdraw balance RUNE=93656250000 balance asset=15984000000 pool unit=99900000000
2:20AM INF app/x/mayachain/manager_txout_current.go:490 > tx out item has zero coin tx_out="To Address:tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzwAsset:MAYA.CACAOAmount:0Memo:OUT:6A4EBFD2881BE3405599E5F9DC80E45C4BF380C95560E596FB6F5415A1860EB6GasRate:2000000000"
2:20AM ERR app/x/mayachain/handler_withdraw.go:41 > fail to process msg withdraw error="2 errors occurred:\n\t* prepare outbound tx not successful\n\t* fail to prepare outbound tx: tx out item has zero coin\n\n"
2:20AM INF app/x/mayachain/manager_txout_current.go:490 > tx out item has zero coin tx_out="To Address:tmaya1wz78qmrkplrdhy37tw0tnvn0tkm5pqd6z6lxzwAsset:MAYA.CACAOAmount:0Memo:REFUND:6A4EBFD2881BE3405599E5F9DC80E45C4BF380C95560E596FB6F5415A1860EB6GasRate:2000000000"
2:20AM ERR app/x/mayachain/helpers.go:100 > fail to prepare outbund tx error="fail to prepare outbound tx: tx out item has zero coin"
```

That's enough for today.

### 23.06.2023 Friday 3h

Right so working on other projects I noticed some of the commands now require
a wallet url as part of the jsonrpc request.

How much this will effect us, I'm not sure at the moment.

Last time I left this I had my own branch `add-dash-chain2` which fixed the unit
tests which are responsible for generating the smoke tests.

This was the main fix: `self.dash_estimate_size = 451`

Have they tested the smoke tests with `self.dash_estimate_size = 451` in
`test/smoke/mayachain/mayachain.py`? I assume so, that's merged into my branch
now.

So they want help rebasing my branch onto develop. Why not merge develop back
into my branch? That's quite often easier.

Okay so the smoke test `balances/events` json have conflicts.
Along with `mayachain.py`.

I think the fastest way would be to fix the smoke unit tests, regenerate the
balances/events, see how that diffs against the current develop branch, and if
it just adds dash, we're good. If it also changes values accross other coins,
I'll go back to the develop branch and run the smokes again. If those fail,
then we're stuck, if they pass, then I can come back to this branch and try
either the merge again with painsteaking attention to what dash is doing, or
even worse, manually go through every god damn transaction.

Whoa, look at this:

```
The following paths are ignored by one of your .gitignore files:
test/smoke/data/smoke_test_balances.json
```

I did a WIP commit which removed balances/events then re-generated them, and git
has been configured to completely ignore balances. So we've had files in the
repo which have been committed and subsequently added to the gitignore file.

Which one is it? Ignore completely? Commit and preserve for all eternity? I'm
guessing the former is more likely, and it's been commited accidentily, seeing
as they're supposed to be throw-away files that just get regenerated (so long
as the unit tests work)

I'm going to remove the balances from `.gitignore` though. It doesn't hurt to
start from a place of security and safety.

Okay this attempt isn't exactly going to plan. I have the files in place, but
a lot seemed to change from develop. I'll give the smokes a go anyway, but...
another way of tackling this might be to:  
run the develop smoke tests (they should be fine)  
run the develop smoke units  
  if they change the balances/events, checkout, update the unit tests, try again  
  repeat until you can generate the same expected balances/events as the successful develop branch
then switch over to dash and go from there

Okay want the last 2 commits on my vm.
```
gd head~2..head > /tmp/t.patch
scp /tmp/t.patch adc-vm:/tmp/t.patch

git apply /tmp/t.patch
```

First run:

```
E[2023-06-23 19:49:03,750] Status 500 - timed out waiting for tx to be included in a block
E[2023-06-23 19:49:03,750] Smoke tests failed
```

Second run. All good. Amazing.

Pushed to maya add dash branch, let them know.

### 27.06.2023 Tuesday 6h

Nothing ever seems to be quite done.  

In the case of Dash / Maya, we've realised that too many non-chainlocked blocks
are being skipped. Itzamna reported from block `1894590` to `1894659`.

That's 69 blocks! Usually fun to see, but here, unfortunately not. With an
average block time of 2.5m that would be nearly 3h of waiting.

DAMN.

Although Dash community member xkcd has reported no chainlock delays in the past
24h, so perhaps it's a localised issue. Either way, the plan to accomodate this
is:

- Ignore non-chainlocked blocks
- Wait for a CL block
- If that block is more than 1 block since the last CL'd block, we follow the
  ancestor chain backwards until the first non-CL'd block since the last CL'd
  block and replay all those blocks.

Right waiting for a new mainnet node to catch up before I can test the rpc flow
that makes the most sense, have a little PoC ready.

In the mean time may as well plan out how I'm going to tackle the TC/Maya
codebase.

**Key Components**

`BlockScannerFetcher`
- `FetchMemPool`
- `FetchTxs`
- `GetHeight`

`BlockScanner`
- `FetchLastHeight`
- `scanBlocks`
- `scanMempool`

`ChainClient`
- `OnObservedTxIn`
- `GetHeight`
- `GetConfirmationCount`

`Observer`
- `globalTxsQueue`
- `chunkifyAndSendToThorchain`
- `processTxIns`
- `sendDeck`

**Notes / Recap**

• `TxIn` is an array of transactions from a specific chain.

• A "deck" is `[]types.TxIn` which is - besides confusing thanks to the lack of
  plurals - an array of an array of transactions, because TxIn is itself a
  multitude of transactions.

• Each chain has a `BlockScanner`, which polls each client at the maya block
  rate and forwards txs to the `Observer` via the `globalTxsQueue`.

• `FetchTxs` is **only** called within a `BlockScanner`, with a specific block
  height. `scanBlocks` within `BlockScanner`s are the only place where this
  happens.

• `GetHeight` is **always** called before `FetchTxs`.

• `FetchMempool` within each `BlockScanner` also results in transactions being
  passed to the `globalTxsQueue`.

• The `Observer` collects these transactions onto a "deck" - they also call a
  multitude of decks a "deck" so be careful there - within `processTxIns`.
  These are finally broadcast to maya in the `sendDeck` method, which is called
  repeatedly every maya block duration.

• The `Observer` sends the entire deck back to each `ChainClient` implementation
  via `GetConfirmationCount` during a `sendDeck`. This is the only place where
  `GetConfirmationCount` is actually used.

• Maya doesn't need to scan the mempool for ETH or BNB.

• Transactions in the ETH mempool are confirmed.

• `getBlock` within the dash ChainClient is called by `FetchTxs`,
  `GetConfirmationCount` **but not (bypassed by?)** `ConfirmationCountReady`.

**Resolve These**

- What if the block height of a chain client has moved say, 4 blocks since the
  last maya block?

  The `BlockScanner` always moves along the chain one block at
  a time at the same rate as maya chain. So you can't have more than 1 chain
  client block to 1 maya block. Is it safe then to assume the maya block time
  HAS to be faster than the chain client block time or else it would slowly
  fall behind - I think so, yes.

- Where does the confirmation count checking happen?

  The `ChainClient`s own `BlockScanners` query them periodically and forward txs
  to the `Observer`. The `Observer` builds a deck of txs and then passes them
  all back to the `ChainClient` via the `GetConfirmationCount` func, which does
  this little number: `totalTxValue / totalFeeAndSubsidy = confirmationsNeeded`
  and returns that to the `Observer`, we then send the entire deck back to the
  `ChainClient` again via the `ConfirmationCountReady` func which should return
  a boolean.

- Can we remove the mempool logic for dash?

  We still have to conform to the interface, but we could return an empty array.
  TBC on this one...

- What happens if the blockscanner asks for the mempool and it contains txs that
  we don't want to include yet because they haven't been CL'd?

  TBC

- `ReportSolvency` on `ChainClient` accepts a block height integer and is part of
  the SolvencyCheckProvider interface. Do we need to worry that it can run ahead
  of the normal block scanning?

  Nope. `SolvencyCheckRunner` always starts with `GetHeight`.

- How do we broadcast transactions from a previous block?

- In the blockscanner it says we scan the mempool as well as blocks so maya is
  still aware of outbound transactions while a chain is halted. Can the mempool
  contain transactions that have not been CL'd? I think so. How does that effect
  us?

- There's this disturbing piece to `ConfirmationCountReady`:
  ```
    if txIn.MemPool {
      return true
    }
  ```

  So the `Observer` WILL be sent mempool txs via the `BlockScanner`. It will
  then check those txs are good to go via the method above, which always
  returns true. AFAIK mempool txs are unconfirmed txs - so what the fuck?? This
  is happening accross all chains.

**Proposition**

After reading the codebase, it seems like the best way to implement the plan is
to stall the block number returned by `GetHeight` until a CL'd block, and remove
the code in `getBlock` which actually prevents getting those blocks entirely.

Everything seems to rely on `GetHeight`, apart from the mempool stuff which I
haven't quite figured out. It looks like mempool txs CAN get to the `Observer` 
deck, which then calls `ConfirmationCountReady` which seems to bypass the
`GetConfirmationCount` / `GetHeight` approach.

This line, in the `BlockScanner`, makes me think - so long as the mempool stuff
is ok - nothing will happen if we stall maya in the above way:

```go
if chainHeight < currentBlock {
  time.Sleep(b.cfg.BlockHeightDiscoverBackoff)
  continue
}
```

Oh wait, it looks like it sends unconfirmed txs to maya WITH a block height at
which they should count as confirmed. That's not going to work for dash seeing
as that is variable depending on how long the next CL'd block takes.

Yeah I'm going to get some sleep now. Interesting though, maybe we can afford to
rip out the mempool logic for dash...

TBC

### 28.06.2023 Wednesday 5h

Okay, mempool stuff...

They're suggesting wait 14400 maya blocks (1day)

576 dash blocks in a day, 2.5m  
14400 maya blocks in a day, 6s

```
dash-cli getblock $(dash-cli getblockhash 1894590) true
```

Yeah that WAS the problem block mentioned by Itzamna.  
Looks fine now. As in - it's chainlocked.

Alright we've adjusted our plan:
- follow forwards NOT BACKWARDS from the latest cl'd block, just incase we end up following an orphan chain
- assume blocks don't return retroactively cl'd (even though they have for both of our nodes) because QE said it could happen, let's be safe
- exponentially backoff node requests while polling for the cl chain, as it may get to 200+ blocks or something crazy
- remove the mempool logic for dash, it requires sending txs with a X block conf. height which we won't know

How do I find out WHEN chainlocks were implemented for a chain.  
The block that I can skip to essentially?  
https://github.com/dashpay/dips/blob/master/dip-0008.md

```
dash-cli getblockchaininfo | jq -r '.bip9_softforks.dip0008.since'
```

I need to test this against regtest, testnet and mainnet ideally because they
will all have different chainlock starting points (if at all) and the code
should work for all situations. Also while a node is catching up the block
height might be before the chainlock dip0008, but that dip will still show as
active on the chain.


Not sure how descriptive I need to be with the `GetHeight` method.  
We deliberated a fair bit to get to our current plan.  
```
// There are a couple of things to consider though:
// • The chainlock feature hasn't been around since the release of dash. As a more recent feature, the original block
//   ledger consists of purely non-chainlocked blocks. We can ignore these since we're deploying after chainlocks.
// • Chainlocking is carried out by the layer 2 masternode quorum, and occurs after a block propagates the network.
// • It is possible for the network to split into sister chains of non-chainlocked blocks, before it settles on a
//   single chain resolution and retroactively chainlocks blocks.
//
// Situation:
//  We are at a block number that's before the chainlock feature was deployed on this network.
// Action:
//  Return the actual node height - it hasn't reached the area with edge-cases yet.
//
// Situation:
//  We don't have a cached value for the latest chainlocked block.
// Action:
//  Iterate backwards through the block history from the current node height until we find a chainlocked block.
//
// Situation:
//  We have a cached value for the latest chainlock height.
//  The nodeHeight is the same value as the lastChainlockHeight.
// Action:
//  Return either value - the block is chainlocked.
//
// Situation:
//  We have a cached value for the latest chainlock height.
//  The nodeHeight is greater than the lastChainlockHeight.
// Action:
//  Scan forwards from our cached value to find the next chainlocked block, cache and return that height
//
```

```
1dd40947d (HEAD -> add-dash-chain, adc/add-dash-chain) Update smoke test balances/events
b0ab68dd0 (HEAD -> origin/add-dash-chain, origin/origin/add-dash-chain) Fix smoke balances and events
```

Have a weird branch `origin/origin/...` because I thought checking out would auto
track the origin branch - like it used to?? - but it didn't.  
Some gitfu required.  

Well I thought I was done, have the mempool stuff ripped out, the getheight
rework in, the exponential backoff ready, then I noticed a note on
`OnObservedTxIn`: "do we need to worry about this?" Well I wasn't worried but
maybe now I should be...

It's called within `chunkifyAndSendToThorchain` (shhh they mean Maya) ... yeah
that's it. That happens after we've sent txs to the observer deck, which only
happens now after the blockscanner processes the next block height from the node
which we don't tell it to, until we see a chainlocked block.

I think we might be done here. Now how the hell are we going to test this!?

Will sleep on it...

