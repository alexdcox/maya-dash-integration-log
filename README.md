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

