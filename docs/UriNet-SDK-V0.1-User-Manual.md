# UriNet SDK V0.1 User Manual

## Contents

- [About this manual](#ch-about)
  - [Who it is for and what it covers](#about-scope)
  - [Entries and notation](#about-entries)
- [What UriNet is](#ch-what)
  - [What is inside](#what-inside)
  - [The coin](#what-coin)
  - [Account types](#what-account-types)
- [How UriNet works](#ch-how)
  - [Accounts](#how-accounts)
  - [Recovery](#how-recovery)
  - [Delegation](#how-delegation)
  - [Tokens](#how-tokens)
  - [Reputation](#how-reputation)
  - [Transactions](#how-transactions)
- [Connecting to UriNet](#ch-connect)
  - [Interfaces](#connect-interfaces)
  - [Addresses, amounts and denoms](#connect-addresses)
  - [Reading lists and other conventions](#connect-conventions)
  - [Network details](#connect-network)
- [Using urid](#ch-urid)
  - [Getting started](#urid-get)
  - [Keys](#urid-keys)
  - [Queries](#urid-query)
  - [Transactions](#urid-tx)
  - [Governance proposals](#urid-gov)
  - [Reading results](#urid-results)
- [Accounts: x/smartaccount](#ch-smartaccount)
  - [What it is](#smartaccount-overview)
  - [Transactions](#smartaccount-transactions)
  - [Queries](#smartaccount-queries)
  - [Parameters](#smartaccount-parameters)
- [Delegation: x/delegation](#ch-delegation)
  - [What it is](#delegation-overview)
  - [Transactions](#delegation-transactions)
  - [Queries](#delegation-queries)
  - [Parameters](#delegation-parameters)
- [Tokens: x/token](#ch-token)
  - [What it is](#token-overview)
  - [Transactions](#token-transactions)
  - [Queries](#token-queries)
  - [Parameters](#token-parameters)
- [Vesting: x/vesting](#ch-vesting)
  - [What it is](#vesting-overview)
  - [Transactions](#vesting-transactions)
  - [Queries](#vesting-queries)
  - [Parameters](#vesting-parameters)
- [Reputation: x/reputation](#ch-reputation)
  - [What it is](#reputation-overview)
  - [Transactions](#reputation-transactions)
  - [Queries](#reputation-queries)
  - [Parameters](#reputation-parameters)
- [Governance](#ch-gov)
  - [How proposals work](#gov-overview)
  - [Transactions](#gov-transactions)
  - [Queries](#gov-queries)
  - [Parameters](#gov-parameters)
- [Standard modules](#ch-standard)
  - [What UriNet keeps](#std-overview)
  - [Bank](#std-bank)
  - [Fee grants](#std-feegrant)
  - [Accounts and auth](#std-auth)
  - [Staking](#std-staking)
  - [Distribution](#std-distribution)
  - [Slashing](#std-slashing)
  - [Mint](#std-mint)
  - [Upgrade](#std-upgrade)
  - [Consensus](#std-consensus)
  - [Evidence](#std-evidence)
  - [Transaction service](#std-tx-service)
  - [Node and chain information](#std-node-service)
- [Tools around the chain](#ch-tools)
  - [The Admin Wallet](#tools-admin-wallet)
  - [The sponsor service](#tools-sponsor)
  - [The indexer](#tools-indexer)
  - [The TypeScript SDK and wallet core](#tools-sdk)
- [Appendices](#ch-appendix)
  - [Glossary](#appendix-glossary)
  - [Events](#appendix-events)
  - [Error codes](#appendix-errors)

---

<a id="ch-about"></a>

# About this manual

<a id="about-scope"></a>

## Who it is for and what it covers

This manual is for people and teams who use UriNet. You might hold URI, run an agent for someone, issue a token, vote on proposals, or build a wallet. You should know what a blockchain and a transaction are. You do not need to know UriNet's own words. Each one is explained the first time a chapter uses it, and the [glossary](#appendix-glossary) collects them.

The manual covers how to use the chain. It tells you how to reach a node, which transactions you can send, what you can look up, and what the tools around the chain do. It does not explain how the chain is built inside.

The first chapters explain the ideas and the ways in. The middle chapters describe each module, one entry per transaction and per query. The last chapters cover the tools and the reference appendices.

<a id="about-entries"></a>

## Entries and notation

Every transaction entry has the same parts, in the same order.

1. A short description of what the transaction does and who sends it.
2. A fact box: a small two-column table. It names the signer, what must be true before you send, what comes back, and the `urid` command that builds it.
3. A proto block: the message definition as the chain reads it, with comments removed. Field names in your JSON match the names shown there.
4. A field table that explains each field, its type, and its meaning. Fields of the response sit in the same table.
5. A "Refused when" list with the main reasons the chain turns the transaction down.

A query entry is shorter. It has a description and a fact box with the gRPC method, the REST path and the `urid` command. A proto block shows the request and the response, and a field table explains them.

A transaction that only governance can run carries a "Governance only" note. You put such a message inside a proposal, and it runs if the proposal passes.

The manual writes values this way.

- Account addresses start with `uri1`. Validator operator addresses start with `urivaloper1`.
- Amounts are whole numbers of base units, written as strings. For URI the base unit is kwak, so `"1000000000"` in a URI amount means 1 URI.
- `<node>` stands for the node you connect to. Replace it with the host name your node operator gives you, as in `https://<node>:1317/urinet/token/v1/params`.
- A word in square brackets in a command, such as `[denom]`, is a value you supply.

---

<a id="ch-what"></a>

# What UriNet is

AI agents are starting to do real work for people. Most of them still work through systems built for humans. They sign in with borrowed passwords, click through screens meant for human eyes, and act as if they were the person they serve. That gets tasks done, but it blurs what the agent may do and who answers for its actions.

UriNet is a blockchain built for agents. An agent gets an account of its own. The person it serves gives it a precise grant of authority, with limits the network enforces. The network checks every action against those limits before it runs. The agent's reputation is recorded on the chain, where anyone can read it, so it follows the agent wherever it works. People stay in control, and agents get room to work.

<a id="what-inside"></a>

## What is inside

UriNet adds five modules of its own. A module is the part of the chain that handles one feature, with its own transactions and queries.

**Smart accounts.** Every account on UriNet is a SmartAccount: an address controlled by one or more registered keys, with a rule for how many must sign. An account can record an Owner, such as the shop that runs an agent. The record gives the Owner no power to sign or spend. An account can also name recovery helpers who restore access if its keys are lost. See [Accounts](#ch-smartaccount).

**Delegation.** One account can give another account limited authority to spend its funds. Alice can let her agent send up to a fixed total, to chosen recipients, until a set date. The chain checks every use against those limits. See [Delegation](#ch-delegation).

**Tokens.** Anyone can issue a token. The issuer chooses its rules: whether more can be minted, whether transfers need approval, who may hold it, and which of these rules can change later. The chain applies those rules to every movement. See [Tokens](#ch-token).

**Vesting.** Coins can arrive locked and unlock on a schedule. Locked coins count in your balance but cannot be spent until they unlock. See [Vesting](#ch-vesting).

**Reputation.** The chain stores one current score per account, with an evidence hash, a fingerprint of the evidence behind it. A committee decides who may write scores. The scores themselves are computed outside the chain. See [Reputation](#ch-reputation).

Underneath, UriNet runs the standard modules that most chains of its kind share. They cover accounts, balances and sends, staking, slashing, rewards and minting. They also cover governance, upgrades, consensus settings, fee grants and evidence. They work as usual, with a few values fixed for UriNet. See [Standard modules](#ch-standard).

Some things are deliberately absent.

- No cross-chain transfers. Assets stay on UriNet.
- No general smart contracts. The chain runs only the modules described here.
- No generic authorization module. Delegation replaces it, with limits the chain enforces.
- No separate recovery module. Recovery lives inside each account.

<a id="what-coin"></a>

## The coin

UriNet's own coin is URI. Its base unit is kwak, and 1 URI is 1,000,000,000 kwak. On the wire every amount is a whole number of kwak, written as a string. Network fees are paid in kwak only. URI is also the coin you stake and the coin you deposit on governance proposals.

<a id="what-account-types"></a>

## Account types

Every account carries a type. The type is a label for display and search. All four types work the same way on the chain.

- **HUMAN** marks a person, such as Alice.
- **AGENT** marks a software agent that acts for someone, such as Alice's agent.
- **ORGANIZATION** marks a company or other group, such as the shop.
- **SERVICE** marks a service run by an operator, such as a fee sponsor.

---

<a id="ch-how"></a>

# How UriNet works

<a id="how-accounts"></a>

## Accounts

Every account on UriNet is a SmartAccount. A SmartAccount is an address plus the rules for who may sign for it. The address never changes, even when every key behind it is replaced.

The keys an account signs with are its credentials. A credential is a public key the chain uses to check signatures. There are two kinds. A KEYPAIR credential is an ordinary key pair on the secp256k1 or P-256 curve. A WEBAUTHN credential is a passkey held by a phone, a laptop or a security key. The chain stores only public keys. It never sees a private key or knows which device holds a credential.

An account can have several credentials. Its minimum approvals, `min_approvals`, is how many distinct credentials must sign each transaction. Bob might register three credentials with minimum approvals of 2. Any two of his three devices can then sign together. The number is at least 1 and at most the number of credentials.

The auth version, `auth_version`, counts changes to this setup. It goes up by one whenever a credential is added or removed or the minimum approvals change. A transaction signed before such a change no longer passes.

An account is created with a bootstrap key. The bootstrap key is a secp256k1 or P-256 key whose public key gives the account its address. It signs the creation once and then has no special power. To keep using that key, register it as a credential. An address can receive funds before its account is created, but it cannot sign, send or delegate until then. You can create the account yourself, paying the fee from the address. Or an existing account can submit the creation for you and pay the fee.

An account may have one Owner. The Owner is another SmartAccount recorded as the owner, such as the shop that owns its agent's account. Being the Owner gives no right to sign, spend or vote in a recovery. Setting an Owner needs both the account and the new Owner to approve. A transfer needs the current and the new Owner. Removing the Owner needs only the Owner. Ownership can never form a loop.

Each account also carries a type: HUMAN, AGENT, ORGANIZATION or SERVICE. The type is a label for display and search. Every type works the same way. See [Accounts](#ch-smartaccount) for the transactions and queries.

<a id="how-recovery"></a>

## Recovery

Recovery lets an account get back in control when its keys are lost or stolen. It does not restore old keys. It installs new credentials on the same address, approved by helpers the account chose in advance.

Those helpers are recovery factors. A recovery factor is another SmartAccount or a public key that holds one vote in a recovery. Alice might name Bob's account and a key she keeps in a safe. An account can never be its own factor. A factor does not need to agree to being named.

The recovery policy is the list of factors plus the recovery threshold, `recovery_min_approvals`. The threshold is the number of votes a decision needs. With any factors at all, it must be more than half of them and no more than their count. An account with no factors has a threshold of 0 and cannot be recovered.

Changes to the policy wait. Adding a factor, removing one, or changing the threshold takes effect 7 days after you ask. Only one change can wait at a time, and you can cancel it before it applies. Factors named when the account is created count at once.

A recovery runs as a recovery session. Any one factor can start a session. The session fixes the factor list, the threshold and the voting period as they stand at that moment. Only one session runs on an account at a time.

Factors vote in the session. A factor proposes an exact change, and proposing counts as its vote for that change. Another factor can vote for the same change or vote to cancel. Each factor holds one current choice and can switch it. A change may add or remove credentials, change minimum approvals, remove factors, or change the threshold. It cannot add factors, change the Owner or move funds.

The session ends the moment one change reaches the threshold, and that change applies. It also ends the moment cancel votes reach the threshold, and nothing changes. The voting period is 7 days. If neither side reaches the threshold by then, the session ends with no change. If a recovery replaces the credentials, every active delegation the account gave is suspended.

While a session runs, the account is locked, with the status RECOVERY_LOCKED. A locked account cannot do the following.

- Sign ordinary transactions with its credentials.
- Send funds, or receive them from other accounts.
- Create, use, suspend, resume or revoke delegations.
- Change its credentials, profile, Owner or recovery policy.
- Pay a fee for itself or for anyone else.

Payments the chain itself makes, such as staking rewards and refunds, still arrive. They stay unusable until the session ends. Anyone may submit a factor's vote, because the vote's authority comes from the factor's own proof. See [Accounts](#ch-smartaccount).

<a id="how-delegation"></a>

## Delegation

A delegation lets one account spend another account's funds within fixed limits. The delegator gives the authority, and the funds come from the delegator's balance. The delegate uses it, signing with its own credentials. Alice is the delegator and her agent is the delegate. A delegation always names exactly two different SmartAccounts.

A delegation holds a list of permissions. A permission has a description, rules written in plain language, and chain rules. People and agents read the plain-language rules. The chain does not interpret them. The chain enforces only the chain rules, and the only chain rule is the SEND rule.

A SEND rule covers one coin or token. It sets who may receive, either anyone or a fixed list of addresses. It sets a required total, `max_total`, and can also cap each single send and the number of uses. Alice could let her agent send at most 500 URI in total, at most 50 URI at a time, only to the shop.

Meters track each SEND rule: the amount sent so far and the number of uses. A refund does not lower a meter. The meter measures authority used, not money lost.

Fees can come from the delegator too. The delegation's fee cap, `max_fee_total`, limits how much in fees the delegator pays for the delegate's transactions. The chain counts every fee charged this way against the cap.

A live delegation is ACTIVE or SUSPENDED. The delegator can suspend it and resume it. Either side can revoke it, which ends it for good. A delegation also ends at the exact moment of its expiry time, `expires_at`. It cannot be used before its start time, `valid_from`. The chain deletes a revoked or expired delegation, so "revoked" is not a stored status. A delegation can last at most 90 days. It cannot be edited. To change one, revoke it and create a new one.

A delegation can never change the delegator's credentials or recovery policy, change its Owner, vote in governance, stake, or pass the authority on. See [Delegation](#ch-delegation).

<a id="how-tokens"></a>

## Tokens

Anyone can issue a token on UriNet. The account that creates it is the issuer. The issuer holds the top authority over its token forever, and no one can take that position away. Even the issuer cannot break the token's own settings and rules. A token's denom, its machine name, is `token/<issuer address>/<subdenom>`. The subdenom is a short name the issuer picks.

A token's character comes from its nine settings. They say whether it can be minted, paused, frozen, force-moved or burned. They also set its maximum supply, its transfer restriction, a holding limit per account, and a maximum number of holders. Each setting has a change rule fixed at creation. The rule says how the setting may change later, for example "increase only" or "fixed". The issuer can also lock a setting for good, so its value never changes again.

The issuer can hand work to other accounts with permissions. There are nine, including admin, burn, pause, freeze and blocklist, forced transfer, and approve transfer. A permission says who may act. A setting says whether the action is possible at all. Minting works differently. A minter is an account with a minter record and an allowance it may still mint. Even the issuer needs its own minter record to mint.

The issuer can pause its token, which stops ordinary movements. Governance can place a harder pause that stops everything for that token. The issuer can freeze part of an account's balance. It can keep a blocklist of accounts that may not send or receive the token, and an allowlist of accounts that may receive it.

The transfer restriction decides how the token moves. Under FREE, anyone may send it with an ordinary send. Under ALLOWLIST_ONLY, only listed accounts may receive. Under APPROVAL_REQUIRED, every transfer also needs approval from the issuer's side.

Some coins in a balance can be locked by vesting, and some can be frozen. Your usable balance is what remains after both. It is what you can actually send. The token query for usable balance shows all the parts.

Vesting schedules on tokens arise in two ways. The issuer can attach a schedule to coins it hands out when it creates the token. A forced transfer can also deliver coins with a schedule. A holder who does not want a token schedule can reject it, and the locked coins are burned. Schedules on URI itself exist only from the chain's starting state, and no transaction attaches one. See [Tokens](#ch-token) and [Vesting](#ch-vesting).

<a id="how-reputation"></a>

## Reputation

The chain keeps one reputation record per account. A record holds a score, a whole number, and an evidence hash, a fingerprint of the evidence behind the score. A new record replaces the old one. An account with no record has no score, which differs from a score of 0.

Updaters write the records. An updater is an account allowed to write scores. Several records can go in one batch. The committee, a single SmartAccount, adds and removes updaters. Governance chooses the committee.

The scores are computed outside the chain by a reputation system. That system keeps the evidence, the method and the history. The chain stores only the current result. See [Reputation](#ch-reputation).

<a id="how-transactions"></a>

## Transactions

A transaction carries one or more messages. A message is one request, such as a send or a vote. You build the transaction, sign it, and broadcast it to a node. If any message fails, none of them takes effect.

UriNet accounts sign with a signature bundle. A signature bundle holds one proof from each credential that signs. It must contain at least the account's minimum approvals, with no credential twice. A bundle with any bad proof is refused as a whole. One transaction may hold at most 20 proofs across all its bundles.

Signers can sit on different devices. Bob's laptop signs first and hands over a partial bundle. His phone adds its proof, and the bundle is complete. Some messages also carry proofs inside them. Examples are the proof that you hold a new credential's key, or a recovery factor's vote. Those proofs go into the message before anyone signs the transaction.

Only two sign modes exist. In DIRECT the signer signs the whole transaction, fee included. The account that pays the fee always signs this way. In DIRECT_AUX a signer signs everything except the fee. It is for signers who do not pay the fee. In a sponsored transaction every signer of a message signs this way. The older JSON sign mode is refused. So is a signature in the multisig form, and so is a transaction that carries extension options, optional add-ons to the transaction body. Each account has one sequence, a counter that goes up by one with every transaction it sends. Unordered transactions, which skip the sequence, are refused too.

There are four ways to pay a fee.

- **Yourself.** The first signer pays from its own balance.
- **A fee grant.** Another account gave you an allowance, and you name it as the granter. See [Fee grants](#std-feegrant).
- **The delegator.** A transaction that uses a delegation can name the delegator as the granter. It must use exactly one delegation, and every message must spend under it. The delegation's `max_fee_total` caps what the delegator pays.
- **A sponsor.** Another account signs as the fee payer. You sign in DIRECT_AUX, and the sponsor signs last. See [Tools around the chain](#ch-tools).

The chain accepts exactly 77 message types. Of these, 32 come from the standard modules and 45 from UriNet's own. Anything else is refused before any fee is taken. Some standard messages are refused by name, such as funding the community pool and the old governance messages.

Eighteen of the 77 are governance-only. They name the governance account as their signer, so only a passed proposal can run them. They are the parameter updates of thirteen modules and five others. The five are the send switch per denom, community pool spending, a software upgrade and its cancellation, and the governance pause of a token. See [Governance](#ch-gov).

Every transaction needs enough gas, the unit that measures the work it costs the chain. `urid` and the TypeScript SDK estimate it by simulating the transaction, then add 30 percent. The fee must cover the gas at the node's minimum gas price, which is 1 kwak per unit of gas by default. If the chain refuses a transaction before running it, no fee is charged. If a message fails after that, the fee is still charged and the sequence moves on, but the messages change nothing. See [Connecting to UriNet](#ch-connect) and [Using urid](#ch-urid).

---

<a id="ch-connect"></a>

# Connecting to UriNet

<a id="connect-interfaces"></a>

## Interfaces

A UriNet node offers four ways in. Your node operator tells you the host name. The ports below are the ones a node uses unless its operator changes them.

| Interface | Port | What it is for |
|---|---|---|
| `urid` | none, it runs on your machine | The command line program. It manages keys, runs queries, and builds, signs and broadcasts transactions. |
| REST | 1317 | HTTP with JSON. Queries are `GET` requests. Transactions are posted to the transaction service. Browser pages and the TypeScript SDK use this. |
| gRPC | 9090 | The same queries and services as REST, over gRPC. Programs and `grpcurl` use this. |
| The node's RPC | 26657 | Node status, blocks and transaction results. `urid` talks to the node here by default. |

Queries and messages reach the chain in different ways. A query is a method you call directly. You send a request and get an answer at once, and nothing on the chain changes. A message is not callable on its own. It rides inside a transaction that its signers sign, and you broadcast that transaction through the transaction service. See [The transaction service](#std-tx-service).

Each message type has a name such as `urinet.token.v1.MsgMint`. Inside a transaction it is written with its type URL, the same name with a leading slash, as in `/urinet.token.v1.MsgMint`. A JSON message carries the type URL in its `@type` field.

<a id="connect-addresses"></a>

## Addresses, amounts and denoms

Addresses are written with a prefix that says what they are.

| Prefix | What it names |
|---|---|
| `uri1` | An account |
| `urivaloper1` | A validator's operator. It is the operator's account address written with another prefix. |
| `urivalcons1` | A validator's consensus key |

The coin is URI, and its base unit is kwak. 1 URI is 1,000,000,000 kwak. Every amount on the wire is a whole number of base units, written as a string. In JSON a coin looks like this, and it means 2.5 URI.

```json
{"denom": "kwak", "amount": "2500000000"}
```

On the command line a coin is the amount followed by the denom, with no space, as in `2500000000kwak`.

A token issued on UriNet has a denom of the form `token/<issuer address>/<subdenom>`. The subdenom is a short name the issuer picks, using lowercase letters, digits, `.`, `_` and `-`. The issuer's address makes the denom unique, so two issuers can both use `gold`. A token is always named by its full denom, never by its display name or symbol. Its amounts are base units too, like kwak for URI.

When a denom goes into a REST path, write it as it is, slashes included. For example, `GET /urinet/token/v1/tokens/token/uri1.../gold` reads one token.

<a id="connect-conventions"></a>

## Reading lists and other conventions

REST paths follow one pattern. UriNet's own modules answer under `/urinet/<module>/v1/`, as in `/urinet/reputation/v1/params`. The standard modules answer under `/cosmos/<module>/<version>/`, as in `/cosmos/bank/v1beta1/balances/{address}`. A part in braces is a value you fill in.

gRPC methods follow the same pattern. A UriNet query is `urinet.<module>.v1.Query/<Method>`. A standard query is `cosmos.<module>.<version>.Query/<Method>`. The node lists its services itself, so `grpcurl` can find and describe every method.

```sh
grpcurl <node>:9090 list
grpcurl <node>:9090 describe urinet.token.v1.Query
grpcurl <node>:9090 describe urinet.token.v1.QueryUsableBalanceRequest
```

To call a method, pass the request as JSON with `-d`.

```sh
grpcurl -d '{"account": "uri1...", "denom": "token/uri1.../gold"}' \
  <node>:9090 urinet.token.v1.Query/UsableBalance
```

Add `-plaintext` when the node serves gRPC without TLS. Your node operator tells you which.

**Paging.** Most queries that return a list return it one page at a time. The request takes `pagination`, and two of its fields matter most. `limit` is the page size, and `key` is where the next page starts. Each answer carries `pagination.next_key`. Pass it as `key` to get the next page. An empty `next_key` means you have the last page.

The paged lists of the token, vesting and reputation modules follow stricter rules.

- Without a limit, a page holds 100 entries.
- A limit above 200 is reduced to 200.
- `reverse` reads the list from the end.
- `offset` and `count_total` are refused. Use `key` to move through the list.

The list of a delegator's delegations is not paged. It returns every delegation in one answer. The standard modules' lists take the usual `pagination` fields, `offset` and `count_total` included.

On REST the fields go in the query string, as in `?pagination.limit=50&pagination.key=...`. The key is base64, so URL-encode it.

**Reading at a fixed height.** By default a query answers from the latest block. To read state as it was at a given height, send the request header `x-cosmos-block-height` with that height. The answer carries the height it was read at in the response header `Grpc-Metadata-X-Cosmos-Block-Height`. Pin several reads to one height when they must agree with each other.

```sh
curl -H 'x-cosmos-block-height: 120000' https://<node>:1317/urinet/token/v1/params
```

With `grpcurl`, send the same header with `-H`. With `urid`, use `--height`. A node that prunes old state cannot answer for heights it no longer keeps.

**Browser pages.** The node answers browser permission requests itself. Any origin may read from the REST port. A page may send the `Content-Type` and `x-cosmos-block-height` headers and may read the height response header. A web app can therefore call a node directly.

**JSON.** A few rules hold in every JSON body.

- Bytes, such as keys, hashes and page keys, are base64.
- Amounts and other large whole numbers are strings.
- Enum values are written by name, as in `"ACCOUNT_TYPE_AGENT"`.
- REST uses the field names as the proto blocks show them, such as `next_key`.
- REST answers include empty fields, with their empty value or `null`. `grpcurl` leaves empty fields out of its output.
- In a request, a field you leave out takes its empty value.

A query that fails answers with an HTTP error status and a JSON body that carries a `code` and a `message`.

<a id="connect-network"></a>

## Network details

Every signature is bound to one chain, so you need the chain ID to sign. Read it from the node, together with the latest block height.

```sh
urid status --node tcp://<node>:26657
curl https://<node>:26657/status
curl https://<node>:1317/cosmos/base/tendermint/v1beta1/node_info
```

In the `urid status` answer, `node_info.network` is the chain ID and `sync_info.latest_block_height` is the latest height. The node's RPC answer has the same fields under `result`. Over REST, the chain information service gives the chain ID in `default_node_info.network`, and `/cosmos/base/node/v1beta1/status` gives the latest height. See [Node and chain information](#std-node-service).

Every transaction pays a fee in kwak. The fee must cover the gas the transaction uses at the node's minimum gas price. That price is 1 kwak per unit of gas by default, and a node operator may set a different one. Read the node's own price at `/cosmos/base/node/v1beta1/config`, in the field `minimum_gas_price`.

Two services often run beside a node: the Admin Wallet, a browser console, and a sponsor service that pays fees for others. An indexer may also serve account history. See [Tools around the chain](#ch-tools).

---

<a id="ch-urid"></a>

# Using urid

<a id="urid-get"></a>

## Getting started

`urid` is UriNet's program. The same binary runs a node and works as the client. This chapter covers the client side: keys, queries and transactions.

`urid` keeps its files in a home directory, `~/.urid` unless you pass `--home`. The client settings and, by default, the keyring live there.

Check that you can reach a node with `urid status`. It prints the node's chain ID, in `node_info.network`, and its latest block height.

```sh
urid status --node tcp://<node>:26657
```

Save the node address, the chain ID and the keyring backend in the client settings, so you do not have to pass them every time. Use the chain ID that `urid status` printed.

```sh
urid config set client node tcp://<node>:26657
urid config set client chain-id <chain-id>
urid config set client keyring-backend os
urid config view client
```

The keyring backend decides where keys are kept. `os` uses your operating system's credential store and is the default. `file` keeps an encrypted file in the home directory and asks for a password. `urid version` prints the version of the binary.

<a id="urid-keys"></a>

## Keys

All keys live in one keyring. It holds three kinds of entry.

**Plain keys.** `urid keys add` makes an ordinary key entry. `urid keys list`, `show`, `rename` and `delete` work on every entry, including the two kinds below. UriNet accounts sign only with credential keys, so create the keys you sign with as credential keys.

**Credential keys.** A credential key is a key pair you register on an account as a credential. `urid keys credential` creates and manages them. Each one's keyring entry is its name with the prefix `credential-`, so a credential called `laptop` is the entry `credential-laptop`. Commands that take a credential name accept it with or without the prefix.

| Command | What it does |
|---|---|
| `urid keys credential create [name]` | Makes a new key pair. `--algorithm` is `secp256k1`, the default, or `p256`. |
| `urid keys credential import [name] [private-key-hex]` | Imports a key from its 32-byte private key in hex. Takes `--algorithm` too. |
| `urid keys credential export [name] --unsafe` | Prints the private key in hex. `--unsafe` confirms you mean it. |
| `urid keys credential list` | Lists the credential keys. |
| `urid keys credential show [name]` | Shows one key. `--format` chooses the output. |

`show --format` has three choices. `text` is the default. `credential-json` prints the key as a KEYPAIR credential, the value that the `--credentials` and `--add-credentials` flags of the smart account commands take. `bootstrap-key-json` prints the public key in the form the `--bootstrap-public-key` flag takes.

A bootstrap key is a credential key too. The address it shows is the address of the account it creates.

**Recovery keys.** A recovery key is the key pair behind a public-key recovery factor. `urid keys recovery` has the same five commands: `create`, `import`, `export`, `list` and `show`. Each entry's name carries the prefix `recovery-`. `urid keys recovery show [name] --format factor-json` prints the key as a recovery factor. That is the value `urid tx smartaccount request-recovery-factor-add` and the `--recovery-factors` flag of `urid tx smartaccount create-smart-account` take. Commands that sign with a recovery key read only recovery entries, never credential entries.

A passkey cannot live in the keyring. A proof made by a passkey is made elsewhere and handed to `urid` with a `--proof` flag, as described below.

<a id="urid-query"></a>

## Queries

Queries follow one shape: `urid query <module> <command>`, or `urid q` for short. `urid query --help` lists the modules, and `urid query <module> --help` lists a module's queries. Each query entry in this manual gives its command.

```sh
urid query token usable-balance uri1... token/uri1.../gold
urid query bank balances uri1... --output json
```

These flags work on every query.

| Flag | Meaning |
|---|---|
| `--output` or `-o` | `text`, the default, or `json`. |
| `--no-indent` | Prints JSON on one line. |
| `--height` | Reads state as it was at that height. A node that prunes old state may not have it. |
| `--node` | The node's RPC address, if not saved in the client settings. |
| `--grpc-addr` | Asks the node over gRPC instead. Add `--grpc-insecure` when the port does not use TLS. |

List queries take paging flags. `--page-limit` sets the page size, and `--page-key` takes the `next_key` of the previous answer. `--page-reverse` reads from the end. The list queries of the token, vesting and reputation modules refuse `--page-offset` and `--page-count-total`, and reduce a limit above 200 to 200. See [Reading lists](#connect-conventions).

A few queries read the chain rather than a module: `urid query tx [hash]`, `urid query wait-tx [hash]`, `urid query block` and `urid query txs`.

<a id="urid-tx"></a>

## Transactions

Sending a transaction with `urid` takes up to four steps.

1. **Compose.** Run the module command with `--generate-only`. It writes the unsigned transaction as JSON, which you save to a file. `--from` names the account, by its address.
2. **Add message proofs.** Some messages carry proofs inside them. Add those with `urid tx smartaccount proof` before anyone signs.
3. **Sign.** `urid tx sign [tx-file] [account-address]` signs as the account, with a signature bundle.
4. **Broadcast.** `urid tx broadcast [tx-file]` sends the signed transaction to the node.

Alice sends 2.5 URI to Bob like this. Her account has one credential, `laptop`.

```sh
urid tx bank send <alice-address> <bob-address> 2500000000kwak --generate-only > send.json
urid tx sign send.json <alice-address> --credential laptop --output-document signed.json
urid tx broadcast signed.json
```

**Message proofs.** Four helpers fill the proofs that smart account messages carry. Each reads an unsigned file and writes a new one with `--output-document`.

| Helper | Fills |
|---|---|
| `urid tx smartaccount proof creation [tx-file]` | The bootstrap proof and the proofs of the first credentials, for an account creation. |
| `urid tx smartaccount proof add-credential [tx-file]` | The proof that you hold each new credential's key, for a credential change. |
| `urid tx smartaccount proof initial-owner [tx-file]` | The new Owner's approval, for an account created with an Owner. `--credential` names the Owner's keys. |
| `urid tx smartaccount proof recovery [tx-file]` | A recovery factor's proof, for starting a recovery and for both kinds of vote. For a vote on a proposal it also adds the proofs for the credentials the proposal adds. `--recovery-key` names a public-key factor's key, and `--credential` a smart account factor's keys. |

A proof made elsewhere, such as one from a passkey, goes in with `--proof`, or `--recovery-proof` for a factor's proof. A helper refuses a file that already carries a signature, so add proofs first.

**Signing.** `urid tx sign` builds the account's signature bundle from the keys you name.

| Flag | Meaning |
|---|---|
| `--credential` | A credential key that signs. Repeat it for each key. |
| `--partial` | Allows a bundle below the account's minimum approvals, so another holder can add to it. |
| `--combine` | A partly signed copy of the same transaction. Its proofs for this account are merged in. Repeatable. |
| `--signer-credential` | A credential that will sign later, as `<address>=<credential id in hex or key name>`. Gas estimation needs to know every credential that will sign. |
| `--bootstrap` | Signs your own account creation with its bootstrap key. |
| `--gas` | `auto`, the default, simulates the transaction and adds 30 percent. Or give a number. |
| `--gas-adjustment` | The factor applied to the simulated gas, 1.3 by default. |
| `--fees`, `--gas-prices` | Set the fee directly, or from a price per unit of gas. With neither, the fee comes from the node's minimum gas price. |
| `--sign-mode` | `direct` or `direct-aux` for this account. The fee payer always signs `direct`. |
| `--offline` | Signs without a node. Then give `--account-number`, `--sequence` and `--gas`. |

When several devices sign, each adds its proofs in turn. Bob's account needs two of his credentials. His laptop signs first and names the credential that will follow. His second machine then signs the laptop's output.

```sh
urid tx sign tx.json <bob-address> --credential laptop --partial \
  --signer-credential <bob-address>=<desk credential id in hex> --output-document part.json
urid tx sign part.json <bob-address> --credential desk --output-document signed.json
```

Each holder can also sign the same file separately, and one of them merges the copies with `--combine`. Once any proof is in, the gas and the fee are fixed.

**Signing from a module command.** A module command can sign and send in one step only in a narrow case. The account must have a single credential, and its address must be that key's address. You must give an explicit `--gas`. `--gas auto` and `--dry-run` are refused there. Use `urid tx sign` for everything else. `urid tx simulate` always fails on UriNet, so estimate gas with `urid tx sign --gas auto`.

```sh
urid tx bank send <alice-address> <bob-address> 2500000000kwak --from credential-laptop \
  --gas 200000 --fees 200000kwak
```

**Fees.** `--fees` sets the fee, such as `200000kwak`. `--gas-prices` sets it from the gas limit, such as `1kwak`. Fees are paid in kwak only. Two more flags, given when you compose, decide who pays.

| Flag | Meaning |
|---|---|
| `--fee-payer` | Another account pays and signs as the fee payer. This is how a sponsor pays. |
| `--fee-granter` | The fee comes from a fee grant by this account. Name the delegator here to have a delegator pay for a transaction that uses its delegation. |

With `--fee-payer`, you sign first in DIRECT_AUX and the fee payer signs the same file last. The sponsor service in [Tools around the chain](#ch-tools) does this for you.

<a id="urid-gov"></a>

## Governance proposals

Governance-only messages run inside a proposal. Several commands build the proposal for you. Each one wraps its message in a normal proposal and takes the deposit with `--deposit`.

| Command | The proposal it submits |
|---|---|
| `urid tx <module> update-params-proposal [params]` | Replaces a module's parameters. Exists for `auth`, `bank`, `consensus`, `delegation`, `distribution`, `mint`, `reputation`, `slashing`, `smartaccount`, `token` and `vesting`. |
| `urid tx bank set-send-enabled-proposal [send_enabled]` | Turns sending of a denom on or off. |
| `urid tx distribution community-pool-spend-proposal [recipient] [amount]` | Pays from the community pool. |
| `urid tx token set-governance-hard-pause-proposal [denom] [paused]` | Sets or clears the governance pause of a token. |
| `urid tx upgrade software-upgrade [name]` | Plans a software upgrade. |
| `urid tx upgrade cancel-upgrade-proposal` | Cancels a planned upgrade. |

A parameter proposal replaces the whole parameter set. A field you leave out becomes 0, not its current value. Read the current values first with `urid query <module> params --output json`, change what you need, and send the whole set.

Staking and governance have no `update-params-proposal` command. For their parameters, and for any other proposal, write the messages into a JSON file and use `urid tx gov submit-proposal [proposal.json]`. `urid tx gov draft-proposal` writes a starting file. A message in a proposal names the governance account as its `authority`. `urid query auth module-account gov` shows that address.

An expedited proposal has its own command. You choose its voting period, from 30 seconds up to the chain's expedited voting period of 1 day.

```sh
urid tx gov submit-expedited-proposal proposal.json 6h --from uri1... --generate-only > tx.json
```

The file has the same form as for `submit-proposal`, without the `expedited` field. The deposit must be paid in full in the same transaction. It is 500 URI for a 1-day period and rises on a straight line to 10,000 URI at 30 seconds. A `submit-proposal` file with `expedited` set to true is refused. See [Governance](#ch-gov) for deposits and voting.

<a id="urid-results"></a>

## Reading results

`urid tx broadcast` returns once the node has checked the transaction. It prints the transaction hash, `txhash`, and a `code`. A non-zero code there means the node refused the transaction, and it never reaches a block. Code 0 there means only that the node accepted it. The messages have not run yet. Wait for the block with `urid query wait-tx [hash]`, or look the result up later with `urid query tx`. The transaction service's GetTx gives the same result over REST and gRPC.

```sh
urid query tx <hash> --output json
```

In that final result, `code` 0 means every message succeeded. The result also lists the events the transaction emitted, such as coins moving or a token being created. See [Events](#appendix-events).

A refusal reads as three parts: a codespace, a code and a message. The codespace names the part of the chain that refused, such as `smartaccount`, `token` or `sdk`. The code is a number within that codespace. The message explains the case and often adds `reason=` and `context=` details. A message type outside the allowed list, for example, is refused with codespace `urinet`, code 2, "message not allowed". See [Errors](#appendix-errors) for every codespace and code.

---

<a id="ch-smartaccount"></a>

# Accounts: x/smartaccount

<a id="smartaccount-overview"></a>

## What it is

Every account on UriNet is a SmartAccount: an address plus the rules for who may sign for it. The x/smartaccount module stores those rules and the messages that change them. Alice uses it to register her phone's passkey. The shop uses it to require two of three staff keys for every transaction.

An account's address never changes. It is derived once from a bootstrap key, a secp256k1 or P-256 key pair whose only job is to create the account. An address can receive coins before it is a SmartAccount, but it cannot sign anything until [MsgCreateSmartAccount](#smartaccount-msg-create-smart-account) runs. After that the bootstrap key has no special power. To keep signing with it, register it as a credential.

A credential is a registered public key that may sign for the account, either a passkey or an ordinary key pair. The approval threshold, `min_approvals`, is how many different credentials must approve each transaction. An account with three credentials and a threshold of 2 is a 2-of-3 account. The Auth Policy version, `auth_version`, goes up by one each time the credentials or the threshold change.

Recovery is the emergency path for an account that has lost its keys. It does not bring an old key back. Recovery Factors, which are other accounts or public keys chosen in advance, vote to install new credentials. The recovery threshold, `recovery_min_approvals`, is how many Factor votes a decision needs. [Recovering an account](#how-recovery) walks through the whole flow.

While a recovery runs, the account's status is `RECOVERY_LOCKED`. A locked account cannot sign, send coins, receive coins from other users, change its settings, pay a fee for someone else, or grant a fee allowance. Payments the chain itself makes, such as staking rewards, refunds and finished unbonding, still arrive. They cannot be spent until the recovery ends. Queries keep working.

The Owner is the one other SmartAccount, if any, recorded as the owner of an account. Being the Owner gives no right to sign for the account, spend its coins, or vote in its recovery. For joint ownership, make a shared SmartAccount the Owner, for example a company account with three staff credentials and a threshold of 2.

<a id="smartaccount-proofs"></a>

### Proofs inside messages

Some messages carry proofs of their own, besides the signatures on the transaction. Each proof is a signature over a short document that the chain rebuilds from the message. It names the chain and the exact action, so it cannot be reused for anything else.

A proof of possession shows that whoever adds a new credential really holds its private key. The new key signs a document that names the chain, the account and the credential. When a credential is added to an existing account, the document also names the account's current sequence, so an old proof cannot be replayed. A passkey gives this proof by signing the document in a normal WebAuthn sign-in. The data a passkey returns when it is first created is not a proof of possession.

Four helpers add these proofs to an unsigned transaction file written with `--generate-only`. Run them before you sign, because a file that already carries a signature is refused.

| Helper | What it fills in |
|---|---|
| `urid tx smartaccount proof creation [tx-file]` | The bootstrap proof and the initial credentials' proofs of MsgCreateSmartAccount. |
| `urid tx smartaccount proof initial-owner [tx-file]` | The New Owner's approval in MsgCreateSmartAccount. |
| `urid tx smartaccount proof add-credential [tx-file]` | The proofs of the credentials MsgUpdateAuthPolicy adds. |
| `urid tx smartaccount proof recovery [tx-file]` | The Factor's proof in the three recovery session messages, and the proofs of credentials a recovery proposal adds. |

The helpers read keys from the keyring: `urid keys credential` for credentials and `urid keys recovery` for public-key Recovery Factors. A proof made elsewhere, such as a passkey's, is passed with `--proof`, or with `--recovery-proof` for a Factor's proof. Each helper writes to standard output unless you give `--output-document`. Then sign with `urid tx sign`, described in [Transactions with urid](#urid-tx).

<a id="smartaccount-account-type"></a>

### Account type

The account type is a label set at creation. It says what kind of party the account is and does not change how the account works. It cannot be changed later.

```proto
enum AccountType {
  ACCOUNT_TYPE_UNSPECIFIED = 0;
  ACCOUNT_TYPE_HUMAN = 1;
  ACCOUNT_TYPE_AGENT = 2;
  ACCOUNT_TYPE_ORGANIZATION = 3;
  ACCOUNT_TYPE_SERVICE = 4;
}
```
| Value | Meaning |
|---|---|
| `ACCOUNT_TYPE_HUMAN` | A person. |
| `ACCOUNT_TYPE_AGENT` | An agent that acts for someone. |
| `ACCOUNT_TYPE_ORGANIZATION` | A company or another group. |
| `ACCOUNT_TYPE_SERVICE` | A service. |
| `ACCOUNT_TYPE_UNSPECIFIED` | Not a valid type. The chain refuses it. |

<a id="smartaccount-credential"></a>

### Credential

A credential is one public key that may sign for the account. The account stores only its current active credentials. The chain never stores a private key, a device name, or a credential's history.

```proto
message Credential {
  bytes credential_id = 1;
  CredentialType type = 2;
  Algorithm algorithm = 3;
  bytes public_key = 4;
  bytes webauthn_credential_id = 5;
  bytes verifier_metadata = 6;
}

enum CredentialType {
  CREDENTIAL_TYPE_UNSPECIFIED = 0;
  CREDENTIAL_TYPE_WEBAUTHN = 1;
  CREDENTIAL_TYPE_KEYPAIR = 2;
}

enum Algorithm {
  ALGORITHM_UNSPECIFIED = 0;
  ALGORITHM_SECP256K1 = 1;
  ALGORITHM_P256 = 2;
}
```
Three pairs of type and algorithm are allowed. A passkey is `CREDENTIAL_TYPE_WEBAUTHN` with `ALGORITHM_P256`. An ordinary key pair is `CREDENTIAL_TYPE_KEYPAIR` with `ALGORITHM_P256` or `ALGORITHM_SECP256K1`. Any other pair is refused. `urid keys credential show [name] --format credential-json` prints a key pair as a ready `KEYPAIR` credential.

| Field | Type | Meaning |
|---|---|---|
| credential_id | bytes, base64 in JSON | The credential's 32-byte ID within the account. The chain derives it from the other identity fields. In a credential you add, whatever you put here is ignored and replaced. |
| type | enum | `CREDENTIAL_TYPE_WEBAUTHN` for a passkey, `CREDENTIAL_TYPE_KEYPAIR` for an ordinary key pair. |
| algorithm | enum | `ALGORITHM_P256` or `ALGORITHM_SECP256K1`. |
| public_key | bytes, base64 in JSON | The public key as a 33-byte compressed point. For a passkey it is the passkey's P-256 public key. |
| webauthn_credential_id | bytes, base64 in JSON | Passkeys only, and required for them. The credential ID the authenticator returned, 1 to 1023 bytes. Must be empty for a key pair. |
| verifier_metadata | bytes, base64 in JSON | Must be empty. |

<a id="smartaccount-signature-bundle"></a>

### Signature bundle and credential proof

A SmartAccount signs a transaction with one signature bundle: a list of proofs, one for each credential that approves. The same bundle shape carries the New Owner's approval in MsgCreateSmartAccount and the vote of a Recovery Factor that is another account. A credential proof on its own is also the form of a proof of possession.

```proto
message SmartAccountSignatureBundle {
  repeated SignatureBundleProof proofs = 1;
}

message SignatureBundleProof {
  bytes credential_id = 1;
  CredentialProof proof = 2;
}

message CredentialProof {
  oneof proof {
    KeypairProof keypair = 1;
    WebAuthnProof webauthn = 2;
  }
}

message KeypairProof {
  bytes signature = 1;
}

message WebAuthnProof {
  bytes authenticator_data = 1;
  bytes client_data_json = 2;
  bytes signature = 3;
}
```
The chain refuses the whole transaction if a credential appears twice in a bundle, if a credential ID is not one of the account's active credentials, or if a proof's kind does not match its credential. It also refuses it if even one proof is invalid, or if fewer than `min_approvals` different credentials approve. It never skips a bad proof and counts the rest. One transaction may carry at most 20 credential proofs across all its bundles.

| Field | Type | Meaning |
|---|---|---|
| proofs | list of `SignatureBundleProof` | One entry per approving credential. |
| credential_id | bytes, base64 in JSON | The active credential this proof is checked against. |
| proof | `CredentialProof` | Exactly one of `keypair` or `webauthn`, matching the credential's type. |
| keypair.signature | bytes, base64 in JSON | A key pair's signature over the signed bytes: 64 bytes, `r` then `s`, with the low-S form. |
| webauthn.authenticator_data | bytes, base64 in JSON | The authenticator data of the passkey assertion, as returned. At least 37 bytes. |
| webauthn.client_data_json | bytes, base64 in JSON | The exact client data JSON of the assertion. |
| webauthn.signature | bytes, base64 in JSON | The passkey's signature, converted to 64 bytes `r` then `s` with the low-S form. The DER form authenticators return is refused. |

For a passkey, the WebAuthn challenge is the SHA-256 hash of the bytes being signed, written as base64url without padding. The chain checks that the client data type is `webauthn.get`, that the challenge matches, that the User Present flag is set, and that the signature is valid. It does not check the origin, the relying party, user verification, or a signature counter.

When a transaction is simulated to estimate gas, the signature bytes may be dummies. The credential IDs and proof kinds must be the real ones, because they decide the gas. `urid tx sign` does this for you.

<a id="smartaccount-recovery-factor"></a>

### Recovery Factor

A Recovery Factor has one vote in the account's recovery. A `SMART_ACCOUNT` Factor is another SmartAccount, which votes with its own credentials and threshold. However many of its credentials sign, it is still one vote. A `PUBLIC_KEY` Factor is a public key, which votes with one signature from its private key.

Registering a Factor needs neither the other party's consent nor a proof of possession. A proof is needed only when the Factor actually votes. An account can never be its own Factor, and the same account or the same public key cannot be an active Factor twice. A `SMART_ACCOUNT` Factor need not be a SmartAccount yet when it is registered. To vote, it must be one, and it must not be locked itself.

```proto
message RecoveryFactor {
  uint64 factor_id = 1;
  RecoveryFactorType type = 2;
  string account_address = 3;
  Algorithm algorithm = 4;
  bytes public_key = 5;
}

enum RecoveryFactorType {
  RECOVERY_FACTOR_TYPE_UNSPECIFIED = 0;
  RECOVERY_FACTOR_TYPE_SMART_ACCOUNT = 1;
  RECOVERY_FACTOR_TYPE_PUBLIC_KEY = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| factor_id | number | The Factor's ID within the account. The chain assigns it, starting at 1, and never reuses a deleted ID. In a Factor you add, whatever you put here is ignored. |
| type | enum | `RECOVERY_FACTOR_TYPE_SMART_ACCOUNT` or `RECOVERY_FACTOR_TYPE_PUBLIC_KEY`. |
| account_address | address | `SMART_ACCOUNT` Factors only. The account that votes. Leave empty for a public key. |
| algorithm | `Algorithm` | `PUBLIC_KEY` Factors only. `ALGORITHM_P256` or `ALGORITHM_SECP256K1`. |
| public_key | bytes, base64 in JSON | `PUBLIC_KEY` Factors only. The 33-byte compressed public key. |

`urid keys recovery show [name] --format factor-json` prints a recovery key pair as a ready `PUBLIC_KEY` Factor.

<a id="smartaccount-recovery-policy"></a>

### Recovery policy

The recovery policy is the account's list of active Recovery Factors and its recovery threshold. Recovery Factors are optional.

```proto
message RecoveryPolicy {
  repeated RecoveryFactor factors = 1;
  uint32 recovery_min_approvals = 2;
  uint64 version = 3;
}
```
The threshold follows one rule. With at least one active Factor, it must be more than half of them and at most all of them. With three Factors it is 2 or 3, and with one Factor it is 1. With no Factors it is 0, and the account cannot be recovered.

| Field | Type | Meaning |
|---|---|---|
| factors | list of `RecoveryFactor` | The active Recovery Factors. See [Recovery Factor](#smartaccount-recovery-factor). |
| recovery_min_approvals | number | The recovery threshold: how many Factor votes a proposal or a cancel needs. |
| version | number | Starts at 1 and goes up each time the policy actually changes. |

<a id="smartaccount-record"></a>

### SmartAccount record

This is everything the chain keeps about one SmartAccount. The account number and the sequence are not part of it. Read them from the standard account query.

```proto
message SmartAccount {
  string address = 1;
  AccountType type = 2;
  string name = 3;
  string metadata_uri = 4;
  repeated Credential credentials = 5;
  uint32 min_approvals = 6;
  uint64 auth_version = 7;
  string owner_account = 8;
  RecoveryPolicy recovery_policy = 9;
  AccountStatus status = 10;
  uint64 active_recovery_session_id = 11;
  uint64 next_recovery_factor_id = 12;
  google.protobuf.Timestamp created_at = 13;
}

enum AccountStatus {
  ACCOUNT_STATUS_UNSPECIFIED = 0;
  ACCOUNT_STATUS_NORMAL = 1;
  ACCOUNT_STATUS_RECOVERY_LOCKED = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account's permanent address. |
| type | `AccountType` | The label set at creation. See [Account type](#smartaccount-account-type). |
| name | string | The display name stored on the chain. |
| metadata_uri | string | A link to the account's profile and other details kept off the chain. |
| credentials | list of `Credential` | The active credentials. See [Credential](#smartaccount-credential). |
| min_approvals | number | The approval threshold, from 1 to the number of credentials. |
| auth_version | number | Starts at 1 and goes up each time the credentials or the threshold change. |
| owner_account | address | The Owner. Empty when the account has none. |
| recovery_policy | `RecoveryPolicy` | See [Recovery policy](#smartaccount-recovery-policy). |
| status | enum | `ACCOUNT_STATUS_NORMAL`, or `ACCOUNT_STATUS_RECOVERY_LOCKED` while a recovery runs. |
| active_recovery_session_id | number | The running recovery session's ID while the account is locked, and 0 otherwise. |
| next_recovery_factor_id | number | The ID the next added Factor will get. |
| created_at | timestamp | When the account became a SmartAccount. |

<a id="smartaccount-recovery-proof"></a>

### Recovery proof

A recovery proof is a Factor's approval of one exact recovery action: a start, a vote for a proposal, or a vote to cancel. It signs a document the chain rebuilds from the message. That document names the chain, the account being recovered, the session (0 for a start), the Factor's ID, the account's sequence, the Factor's vote counter (0 for a start), the action, and for a proposal vote the proposal's hash.

```proto
message RecoveryProof {
  oneof proof {
    urinet.signing.v1.SmartAccountSignatureBundle smart_account = 1;
    bytes public_key_signature = 2;
  }
}
```
| Field | Type | Meaning |
|---|---|---|
| smart_account | `SmartAccountSignatureBundle` | For a `SMART_ACCOUNT` Factor. A bundle from the Factor account's credentials that meets that account's own threshold. See [Signature bundle](#smartaccount-signature-bundle). |
| public_key_signature | bytes, base64 in JSON | For a `PUBLIC_KEY` Factor. One signature by the Factor's key: 64 bytes, `r` then `s`, with the low-S form. |

Set exactly one of the two, the one that matches the Factor's type.

<a id="smartaccount-recovery-proposal"></a>

### Recovery proposal

A recovery proposal is the change a successful recovery applies. When a Factor sends a proposal, that act is its vote for it.

```proto
message RecoveryProposal {
  CredentialChange credential_change = 1;
  repeated uint64 remove_recovery_factor_ids = 2;
  uint32 new_recovery_min_approvals = 3;
}

message CredentialChange {
  repeated Credential add_credentials = 1;
  repeated bytes remove_credential_ids = 2;
  uint32 new_min_approvals = 3;
}
```
A proposal can replace credentials, remove Recovery Factors and set both thresholds. It cannot add a Recovery Factor or change the Owner. To leave the credentials as they are, send empty lists and the current `min_approvals`. The chain sorts the lists and hashes the result, so two Factors that list the same change in a different order vote for the same proposal. A repeated entry in any list is refused.

| Field | Type | Meaning |
|---|---|---|
| credential_change | `CredentialChange` | Required. The credential changes to apply. |
| credential_change.add_credentials | list of `Credential` | Credentials to add. Each needs a proof of possession in the vote message. |
| credential_change.remove_credential_ids | list of bytes, base64 in JSON | IDs of current credentials to remove. |
| credential_change.new_min_approvals | number | The approval threshold afterwards. Always the final value, and never 0. |
| remove_recovery_factor_ids | list of numbers | IDs of Factors from the session's snapshot to remove. |
| new_recovery_min_approvals | number | The recovery threshold afterwards, following the [threshold rule](#smartaccount-recovery-policy) for the Factors that remain. Always the final value. |

<a id="smartaccount-recovery-session"></a>

### Recovery session

A recovery session is the running record of one recovery. It opens when one Factor starts the recovery. It takes a snapshot of the active Factors, the recovery threshold and the voting period at that moment, and counts every vote against that snapshot. Session IDs are numbered across the whole chain, from 1.

Each Factor in the snapshot holds one current choice: none, cancel, or one proposal. It can change its choice by voting again, but it cannot withdraw to none. A session ends in one of three ways:

- **Success.** The moment one proposal's votes reach the threshold, its changes apply and the account is unlocked. A recovery that replaces the account's credentials suspends every active [delegation](#ch-delegation) the account gave. The delegator can resume them afterwards.
- **Cancel.** The moment the cancel votes reach the threshold, the account is unlocked and nothing changes.
- **Void.** When the voting period ends below the threshold, the account is unlocked and nothing changes. This happens at the end of the first block at or after the deadline. Votes inside that block still count.

```proto
message RecoverySession {
  uint64 session_id = 1;
  string account = 2;
  google.protobuf.Timestamp started_at = 3;
  google.protobuf.Timestamp deadline = 4;
  uint64 recovery_policy_version = 5;
  repeated RecoveryFactor recovery_factor_snapshot = 6;
  uint32 recovery_min_approvals = 7;
  google.protobuf.Duration recovery_voting_period = 8;
  repeated RecoveryFactorChoice factor_choices = 9;
  repeated RecoveryProposalRecord proposal_records = 10;
  uint32 cancel_count = 11;
}

message RecoveryFactorChoice {
  uint64 factor_id = 1;
  RecoveryChoice choice = 2;
  bytes proposal_hash = 3;
  uint64 vote_nonce = 4;
}

enum RecoveryChoice {
  RECOVERY_CHOICE_UNSPECIFIED = 0;
  RECOVERY_CHOICE_NONE = 1;
  RECOVERY_CHOICE_CANCEL = 2;
  RECOVERY_CHOICE_PROPOSAL = 3;
}

message RecoveryProposalRecord {
  bytes proposal_hash = 1;
  RecoveryProposal proposal = 2;
  uint32 vote_count = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| session_id | number | The session's ID. Never 0 and never reused. |
| account | address | The account being recovered. |
| started_at | timestamp | When the session started. |
| deadline | timestamp | `started_at` plus the voting period. |
| recovery_policy_version | number | The recovery policy version at the start. |
| recovery_factor_snapshot | list of `RecoveryFactor` | The Factors that may vote, as they were at the start. |
| recovery_min_approvals | number | The threshold at the start. |
| recovery_voting_period | duration | The voting period at the start. |
| factor_choices | list of `RecoveryFactorChoice` | The Factors that have chosen. A Factor not listed has chosen nothing. |
| factor_choices.factor_id | number | The Factor, by its ID in the snapshot. |
| factor_choices.choice | enum | `RECOVERY_CHOICE_NONE`, `RECOVERY_CHOICE_CANCEL` or `RECOVERY_CHOICE_PROPOSAL`. |
| factor_choices.proposal_hash | bytes, base64 in JSON | The 32-byte hash of the chosen proposal, when the choice is a proposal. |
| factor_choices.vote_nonce | number | The Factor's vote counter. It starts at 0 and goes up by one each time the Factor's choice actually changes. The Factor's next vote must name this value. |
| proposal_records | list of `RecoveryProposalRecord` | Each proposal that holds at least one vote. |
| proposal_records.proposal_hash | bytes, base64 in JSON | The proposal's 32-byte hash. |
| proposal_records.proposal | `RecoveryProposal` | The proposal's content, with its lists sorted. See [Recovery proposal](#smartaccount-recovery-proposal). |
| proposal_records.vote_count | number | How many Factors currently vote for it. |
| cancel_count | number | How many Factors currently choose cancel. |

<a id="smartaccount-transactions"></a>

## Transactions

These are the twelve messages that create a SmartAccount and change its credentials, profile, Owner and recovery settings.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgCreateSmartAccount](#smartaccount-msg-create-smart-account) | Turns an address into a SmartAccount. | The address itself, or an account that relays |
| [MsgUpdateAuthPolicy](#smartaccount-msg-update-auth-policy) | Adds or removes credentials and sets the approval threshold. | The account |
| [MsgUpdateProfile](#smartaccount-msg-update-profile) | Sets the name and the metadata link. | The account |
| [MsgSetOwner](#smartaccount-msg-set-owner) | Sets, transfers or clears the Owner. | The accounts the change requires |
| [MsgRequestRecoveryFactorAdd](#smartaccount-msg-request-recovery-factor-add) | Asks to add a Recovery Factor after the safety delay. | The account |
| [MsgRequestRecoveryFactorRemove](#smartaccount-msg-request-recovery-factor-remove) | Asks to remove a Recovery Factor after the safety delay. | The account |
| [MsgRequestRecoveryMinApprovalsChange](#smartaccount-msg-request-recovery-min-approvals-change) | Asks to change the recovery threshold after the safety delay. | The account |
| [MsgCancelPendingRecoveryPolicyChange](#smartaccount-msg-cancel-pending-recovery-policy-change) | Cancels the waiting recovery policy change. | The account |
| [MsgStartRecovery](#smartaccount-msg-start-recovery) | Starts a recovery with one Factor's proof. | Anyone, as submitter |
| [MsgVoteRecoveryProposal](#smartaccount-msg-vote-recovery-proposal) | A Factor votes for a proposal. | Anyone, as submitter |
| [MsgVoteCancelRecovery](#smartaccount-msg-vote-cancel-recovery) | A Factor votes to cancel. | Anyone, as submitter |
| [MsgUpdateParams](#smartaccount-msg-update-params) | Replaces the module's parameters. | Governance only |

The commands below take the signing account from `--from`. They can sign and send directly only for an account with one credential whose address is the key's address, and only with an explicit `--gas`. Otherwise, write the transaction with `--generate-only`, add any proofs, and sign it with `urid tx sign`. `set-owner` always writes an unsigned file. See [Transactions with urid](#urid-tx) and [How transactions work](#how-transactions). When the chain refuses a message, the log carries an error code and a reason word, listed in [Errors](#appendix-errors).

<a id="smartaccount-tx-create"></a>

### Creating an account

<a id="smartaccount-msg-create-smart-account"></a>

#### MsgCreateSmartAccount

This message turns an address into a SmartAccount with its first credentials, approval threshold, Recovery Factors, and optional Owner. A balance, vesting or staking the address already has stays as it is. The initial Factors are active at once, without the safety delay that later changes wait for.

There are two ways to send it. In a self-funded creation, the address submits it itself and signs with its bootstrap key. The chain must already know the address, for example because it has received coins. In a relayed creation, an existing SmartAccount submits it and pays. A relayed creation also works for an address the chain has never seen. For example, Alice's wallet service can relay the creation of her new account.

Three kinds of proof go inside the message. The bootstrap proof shows that the bootstrap key's holder approves this exact setup. One proof of possession per credential shows each key is really held. When an initial Owner is named, the Owner's approval bundle shows that the Owner accepts.

| | |
|---|---|
| Signer | `submitter`, the address itself or an initialized SmartAccount that relays |
| Before you send it | The address is not a SmartAccount yet. For a self-funded creation, the chain already knows the address. The fee comes from the address unless a fee grant or a sponsor covers it. An initial Owner is an unlocked SmartAccount whose credentials have signed the approval. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount create-smart-account [address]`, with `--type`, `--name`, `--metadata-uri`, `--bootstrap-public-key`, `--credentials`, `--min-approvals`, `--recovery-factors`, `--recovery-min-approvals`, `--initial-owner`, and optionally `--bootstrap-proof`, `--credential-proofs`, `--initial-owner-approval`. Then `proof creation`, `proof initial-owner` if needed, and `urid tx sign`, with `--bootstrap` for a self-funded creation. |

```proto
message MsgCreateSmartAccount {
  string submitter = 1;
  string address = 2;
  google.protobuf.Any bootstrap_public_key = 3;
  AccountType type = 4;
  string name = 5;
  string metadata_uri = 6;
  repeated Credential credentials = 7;
  uint32 min_approvals = 8;
  repeated RecoveryFactor recovery_factors = 9;
  uint32 recovery_min_approvals = 10;
  string initial_owner = 11;
  urinet.signing.v1.SmartAccountSignatureBundle initial_owner_approval = 12;
  bytes bootstrap_proof = 13;
  repeated urinet.signing.v1.CredentialProof credential_proofs = 14;
}

message MsgCreateSmartAccountResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| submitter | address | Who submits and signs. The address itself, or a relaying SmartAccount. |
| address | address | The address to turn into a SmartAccount. It must be the address of `bootstrap_public_key`. |
| bootstrap_public_key | typed public key | The bootstrap key, as a JSON object with `@type` `/cosmos.crypto.secp256k1.PubKey` or `/cosmos.crypto.secp256r1.PubKey`. `urid keys credential show [name] --format bootstrap-key-json` prints it. |
| type | `AccountType` | See [Account type](#smartaccount-account-type). Must be set. |
| name | string | The display name. |
| metadata_uri | string | The link to the off-chain profile. |
| credentials | list of `Credential` | The first credentials. At least one. See [Credential](#smartaccount-credential). |
| min_approvals | number | The approval threshold, from 1 to the number of credentials. |
| recovery_factors | list of `RecoveryFactor` | The first Recovery Factors, possibly none. They get IDs 1, 2, 3 and so on, in this order. |
| recovery_min_approvals | number | The recovery threshold. 0 when there are no Factors. See [Recovery policy](#smartaccount-recovery-policy). |
| initial_owner | address | The Owner. Empty for none. |
| initial_owner_approval | `SmartAccountSignatureBundle` | The Owner's approval, made with the Owner's current credentials and meeting its threshold. Present exactly when `initial_owner` is set. |
| bootstrap_proof | bytes, base64 in JSON | The bootstrap key's signature over the whole setup, the chain and the address. |
| credential_proofs | list of `CredentialProof` | One proof of possession per credential, in the same order. |

The new account starts with `auth_version` 1, recovery policy version 1, and status `NORMAL`.

**Refused when**

- The address is already a SmartAccount.
- The address does not match the bootstrap key, or the bootstrap proof is invalid.
- The number of credential proofs differs from the number of credentials, or a proof is invalid.
- The threshold is out of range, a credential is repeated, a type and algorithm pair is not allowed, or there are more credentials or Factors than the parameters allow.
- The name or the metadata link is longer than the parameters allow.
- The recovery policy breaks its rules: a wrong threshold, a repeated Factor, or the account as its own Factor.
- The Owner is not an unlocked SmartAccount, its approval is missing or too weak, or an approval comes without an Owner.
- The address is one of the chain's own internal accounts, or a kind of account that cannot become a SmartAccount.

<a id="smartaccount-tx-auth"></a>

### Credentials and profile

<a id="smartaccount-msg-update-auth-policy"></a>

#### MsgUpdateAuthPolicy

This message adds or removes credentials and sets the approval threshold, in one step. It must be approved under the rules in force before the change. When Alice gets a new phone, she adds its passkey with the old phone's approval, then removes the old one.

Each added credential needs a proof of possession bound to the account's current sequence. Sign and send soon after adding the proofs, before the account sends another transaction. On success, `auth_version` goes up by one.

| | |
|---|---|
| Signer | `account`, the account being changed |
| Before you send it | The account is not locked. The final number of credentials stays within `max_credentials` if you add any. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount update-auth-policy [new-min-approvals]` with `--add-credentials` and `--remove-credential-ids` (repeatable, IDs in hex or base64), then `urid tx smartaccount proof add-credential [tx-file]`. |

```proto
message MsgUpdateAuthPolicy {
  string account = 1;
  repeated Credential add_credentials = 2;
  repeated bytes remove_credential_ids = 3;
  repeated urinet.signing.v1.CredentialProof credential_proofs = 4;
  uint32 new_min_approvals = 5;
}

message MsgUpdateAuthPolicyResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to change. Must sign. |
| add_credentials | list of `Credential` | Credentials to add. Empty for none. See [Credential](#smartaccount-credential). |
| remove_credential_ids | list of bytes, base64 in JSON | IDs of current credentials to remove. Empty for none. |
| credential_proofs | list of `CredentialProof` | One proof of possession per added credential, in the same order. |
| new_min_approvals | number | The threshold after the change. Always the final value, even if it stays the same. Never 0. |

A removal or a threshold change without additions works even when the account has more credentials than a lowered `max_credentials` now allows.

**Refused when**

- The account is locked by a recovery.
- The final threshold is 0 or larger than the final number of credentials.
- Credentials are added and the final number exceeds `max_credentials`.
- A credential is added twice, already exists, or is both added and removed.
- A credential to remove does not exist.
- A type and algorithm pair is not allowed, or a proof of possession is missing or invalid.
- Nothing actually changes.

<a id="smartaccount-msg-update-profile"></a>

#### MsgUpdateProfile

This message sets the account's display name and metadata link. Send both values every time. A value equal to the stored one counts as unchanged. The address, the account number, the type and the creation time can never change.

| | |
|---|---|
| Signer | `account`, the account being changed |
| Before you send it | The account is not locked. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount update-profile [name] [metadata-uri]`. Pass `""` for an empty value. |

```proto
message MsgUpdateProfile {
  string account = 1;
  string name = 2;
  string metadata_uri = 3;
}

message MsgUpdateProfileResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to change. Must sign. |
| name | string | The display name after the change. |
| metadata_uri | string | The metadata link after the change. |

Only a value that actually changes is checked against the current size limit. A stored value that is longer than a lowered limit can stay.

**Refused when**

- The account is locked by a recovery.
- A changed name exceeds `max_account_name_bytes`, or a changed link exceeds `max_metadata_uri_bytes`.
- Neither value changes.

<a id="smartaccount-tx-owner"></a>

### The Owner

<a id="smartaccount-msg-set-owner"></a>

#### MsgSetOwner

This message sets, transfers or clears an account's Owner. The accounts that must approve depend on the change:

| Change | Who must sign |
|---|---|
| No Owner, then Owner B | The account itself and B |
| Owner A, then Owner B | A and B. The account itself does not sign. |
| Owner A, then no Owner | A |

So an account can never remove its Owner on its own, and a recovery cannot change the Owner. Every approver signs the same transaction. The chain stores no half-approved change. For example, the shop becomes the Owner of its agent when the agent and the shop both sign.

| | |
|---|---|
| Signer | `approvers`, exactly the accounts the change requires |
| Before you send it | The account and the new Owner are SmartAccounts. Neither the account nor any approver is locked. The change would not create an ownership loop. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount set-owner [account] [new-owner] [approver]...`, where `none` as the new Owner clears it. This command only builds the unsigned transaction. |

The first approver listed pays the fee. Each approver then signs in turn with `urid tx sign [tx-file] [approver]`, each on the previous output. The first signer sets gas and fee. When `urid tx sign` estimates the gas, that signer names the other approvers' credentials with `--signer-credential`. Send the fully signed file with `urid tx broadcast`.

In this example `uri1aaa...` is the agent and `uri1bbb...` is the shop. Each signs with a credential from its own keyring.

```sh
urid tx smartaccount set-owner uri1aaa... uri1bbb... uri1aaa... uri1bbb... \
  --output-document owner.json
urid tx sign owner.json uri1aaa... --credential key-a \
  --signer-credential uri1bbb...=key-b --output-document owner-1.json
urid tx sign owner-1.json uri1bbb... --credential key-b --output-document owner-2.json
urid tx broadcast owner-2.json
```

```proto
message MsgSetOwner {
  string account = 1;
  string new_owner = 2;
  repeated string approvers = 3;
}

message MsgSetOwnerResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account whose Owner changes. |
| new_owner | address | The Owner afterwards. Empty for none. |
| approvers | list of addresses | The accounts that sign, exactly the set in the table above. Order does not matter. No address may appear twice. |

**Refused when**

- The account or the new Owner is not a SmartAccount.
- The approvers are not exactly the set the change requires, or the current Owner is not the one the change assumes.
- An approver's signature does not meet its threshold, or the account or an approver is locked.
- The change would create an ownership loop, such as A owning B while B owns A.
- Nothing actually changes.

<a id="smartaccount-tx-recovery-policy"></a>

### Recovery policy changes

Changes to the recovery policy do not apply at once. Each waits for the safety delay, 7 days, and an account can have only one waiting change at a time. You can cancel it while it waits. If a recovery starts in the meantime, the waiting change is deleted. The change applies at the end of the first block at or after its activation time, and the policy's version goes up by one. A transaction in that same block still sees the old policy. Read a waiting change with [PendingRecoveryPolicyChange](#smartaccount-query-pending-recovery-policy-change).

<a id="smartaccount-msg-request-recovery-factor-add"></a>

#### MsgRequestRecoveryFactorAdd

This message asks to add one Recovery Factor after the safety delay. An account with no Factors can add one this way. The other party does not need to agree, and a public key needs no proof of possession. Until the change applies, the new Factor cannot start or vote in a recovery and does not count toward the threshold.

For example, Alice has two Factors and registers Bob's account as a third, with a recovery threshold of 2. Bob's Factor becomes active 7 days later.

| | |
|---|---|
| Signer | `account`, the account being changed |
| Before you send it | The account is not locked and has no waiting change. It has fewer active Factors than `max_recovery_factors`. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount request-recovery-factor-add [new-factor] [new-recovery-min-approvals]`. The Factor is JSON, such as `{"type":"RECOVERY_FACTOR_TYPE_SMART_ACCOUNT","account_address":"uri1..."}`, or the output of `urid keys recovery show [name] --format factor-json`. |

```proto
message MsgRequestRecoveryFactorAdd {
  string account = 1;
  RecoveryFactor new_factor = 2;
  uint32 new_recovery_min_approvals = 3;
}

message MsgRequestRecoveryFactorAddResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to change. Must sign. |
| new_factor | `RecoveryFactor` | The Factor to add. Its ID is assigned when the change applies. See [Recovery Factor](#smartaccount-recovery-factor). |
| new_recovery_min_approvals | number | The recovery threshold for the Factors after the addition, following the [threshold rule](#smartaccount-recovery-policy). |

**Refused when**

- The account is locked by a recovery, or a change is already waiting.
- The Factor is the account itself, or is already an active Factor.
- The Factor has fields its type does not use, or an algorithm or key that is not allowed.
- The account already has `max_recovery_factors` active Factors.
- The new threshold breaks the threshold rule.

<a id="smartaccount-msg-request-recovery-factor-remove"></a>

#### MsgRequestRecoveryFactorRemove

This message asks to remove one active Recovery Factor after the safety delay. You can remove the last Factor too, and then the new threshold must be 0. [RecoveryPolicy](#smartaccount-query-recovery-policy) lists the Factor IDs.

| | |
|---|---|
| Signer | `account`, the account being changed |
| Before you send it | The account is not locked and has no waiting change. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount request-recovery-factor-remove [recovery-factor-id] [new-recovery-min-approvals]` |

```proto
message MsgRequestRecoveryFactorRemove {
  string account = 1;
  uint64 recovery_factor_id = 2;
  uint32 new_recovery_min_approvals = 3;
}

message MsgRequestRecoveryFactorRemoveResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to change. Must sign. |
| recovery_factor_id | number | The ID of the active Factor to remove. |
| new_recovery_min_approvals | number | The recovery threshold for the Factors that remain. 0 when none remains. |

**Refused when**

- The account is locked by a recovery, or a change is already waiting.
- The ID is not one of the account's active Factors.
- The new threshold breaks the [threshold rule](#smartaccount-recovery-policy) for the Factors that remain.

<a id="smartaccount-msg-request-recovery-min-approvals-change"></a>

#### MsgRequestRecoveryMinApprovalsChange

This message asks to change only the recovery threshold, after the safety delay. The Factors stay as they are.

| | |
|---|---|
| Signer | `account`, the account being changed |
| Before you send it | The account is not locked and has no waiting change. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount request-recovery-min-approvals-change [new-recovery-min-approvals]` |

```proto
message MsgRequestRecoveryMinApprovalsChange {
  string account = 1;
  uint32 new_recovery_min_approvals = 2;
}

message MsgRequestRecoveryMinApprovalsChangeResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to change. Must sign. |
| new_recovery_min_approvals | number | The recovery threshold afterwards. |

**Refused when**

- The account is locked by a recovery, or a change is already waiting.
- The new threshold breaks the [threshold rule](#smartaccount-recovery-policy).
- The new threshold equals the current one.

<a id="smartaccount-msg-cancel-pending-recovery-policy-change"></a>

#### MsgCancelPendingRecoveryPolicyChange

This message deletes the account's waiting recovery policy change at once. The policy and its version stay as they are. Use it if a change was requested by mistake or without your consent.

| | |
|---|---|
| Signer | `account`, the account that requested the change |
| Before you send it | A change is waiting. The account is not locked. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount cancel-pending-recovery-policy-change` |

```proto
message MsgCancelPendingRecoveryPolicyChange {
  string account = 1;
}

message MsgCancelPendingRecoveryPolicyChangeResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account whose waiting change is deleted. Must sign. |

**Refused when**

- No change is waiting.
- The account is locked by a recovery.

<a id="smartaccount-tx-recovery-session"></a>

### Recovery sessions

In the three messages below, the account that signs the transaction is only the submitter. It can be anyone, such as a relay service, and it gets no vote. The Factor's authority is the separate recovery proof inside the message. See [Recovery proof](#smartaccount-recovery-proof). The submitter must itself be an unlocked SmartAccount.

<a id="smartaccount-msg-start-recovery"></a>

#### MsgStartRecovery

This message starts a recovery of an account with the proof of one of its active Recovery Factors. The account is locked at once, and a [recovery session](#smartaccount-recovery-session) opens with a snapshot of the Factors, the threshold and the 7-day voting period. A waiting recovery policy change is deleted. Starting casts no vote. The Factors, including the one that started, then vote with the two messages that follow.

The account's sequence goes up by exactly one. The proof names the sequence before the start. Put any votes for the new session in a later transaction.

| | |
|---|---|
| Signer | `submitter`, any unlocked SmartAccount |
| Before you send it | The account is a SmartAccount, not locked, and has at least one active Factor. A `SMART_ACCOUNT` Factor is an unlocked SmartAccount. |
| What comes back | Nothing in the response. The new session ID is in the `urinet.smartaccount.v1.EventRecoveryStarted` event, listed in [SmartAccount events](#events-smartaccount), and [ActiveRecoverySession](#smartaccount-query-active-recovery-session) returns it. |
| urid | `urid tx smartaccount start-recovery [account] [recovery-factor-id]` with `--recovery-proof`, or with `--generate-only` and then `urid tx smartaccount proof recovery [tx-file]` using `--recovery-key` for a public-key Factor or `--credential` for an account Factor. |

```proto
message MsgStartRecovery {
  string submitter = 1;
  string account = 2;
  uint64 recovery_factor_id = 3;
  RecoveryProof recovery_proof = 4;
}

message MsgStartRecoveryResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| submitter | address | Who submits and signs the transaction. Casts no vote. |
| account | address | The account to recover. |
| recovery_factor_id | number | The active Factor that approves the start. |
| recovery_proof | `RecoveryProof` | That Factor's proof over the start action. See [Recovery proof](#smartaccount-recovery-proof). |

**Refused when**

- The account is not a SmartAccount.
- A recovery of the account is already running, or the same transaction already starts one.
- The Factor ID is not one of the account's active Factors, including when it has none.
- The proof's kind does not match the Factor's type, or the proof is invalid or names other values, such as an old sequence.
- A `SMART_ACCOUNT` Factor is not a SmartAccount, is locked, or its bundle does not meet its own threshold.

<a id="smartaccount-msg-vote-recovery-proposal"></a>

#### MsgVoteRecoveryProposal

A Factor in the session's snapshot votes for a [recovery proposal](#smartaccount-recovery-proposal) with this message. Its earlier choice, if any, is replaced. The moment a proposal reaches the threshold, the recovery succeeds in the same transaction and the account is unlocked.

Proofs of possession for added credentials are checked only by the vote that first records a proposal. A later vote for a proposal that already holds votes does not need them. If every vote for it moves away, the record is deleted and the next vote must bring the proofs again. Voting for your current choice again changes nothing.

For example, Bob and the shop are two of Alice's Factors. Both vote for the proposal that adds her new passkey and removes the lost one. With a threshold of 2, the second vote completes the recovery.

| | |
|---|---|
| Signer | `submitter`, any unlocked SmartAccount |
| Before you send it | The session is running. The Factor is in its snapshot. Read the Factor's `vote_nonce` with [RecoveryFactorChoices](#smartaccount-query-recovery-factor-choices). |
| What comes back | Nothing. The transaction result and the session's events tell you what happened. |
| urid | `urid tx smartaccount vote-recovery-proposal [account] [session-id] [recovery-factor-id] [proposal]` with `--recovery-proof` and `--credential-proofs`, or with `--generate-only` and then `urid tx smartaccount proof recovery [tx-file]`. |

The proposal is JSON. Counters of 64 bits, such as Factor IDs, are written as strings:

```json
{"credential_change":{"add_credentials":[...],"remove_credential_ids":["..."],"new_min_approvals":1},
 "remove_recovery_factor_ids":["3"],"new_recovery_min_approvals":1}
```

```proto
message MsgVoteRecoveryProposal {
  string submitter = 1;
  string account = 2;
  uint64 session_id = 3;
  uint64 recovery_factor_id = 4;
  RecoveryProposal proposal = 5;
  RecoveryProof recovery_proof = 6;
  repeated urinet.signing.v1.CredentialProof credential_proofs = 7;
}

message MsgVoteRecoveryProposalResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| submitter | address | Who submits and signs the transaction. Casts no vote. |
| account | address | The account being recovered. |
| session_id | number | The running session. |
| recovery_factor_id | number | The voting Factor, from the snapshot. |
| proposal | `RecoveryProposal` | What the Factor votes for. See [Recovery proposal](#smartaccount-recovery-proposal). |
| recovery_proof | `RecoveryProof` | The Factor's proof over this vote, naming the proposal's hash and the Factor's current `vote_nonce`. |
| credential_proofs | list of `CredentialProof` | One proof of possession per added credential, in the order the message lists them. Each names the account being recovered and its current sequence. |

**Refused when**

- The session ID is not the account's running session.
- The Factor is not in the snapshot, or its proof is invalid or names an old `vote_nonce`.
- A `SMART_ACCOUNT` Factor is not a SmartAccount or is locked.
- The proposal has no credential change, or a credential or Factor to remove does not exist.
- A list has a repeated entry, or a threshold breaks its rule.
- Added credentials push the count above `max_credentials`, a pair is not allowed, or a proof of possession is invalid.
- The proposal changes nothing.

<a id="smartaccount-msg-vote-cancel-recovery"></a>

#### MsgVoteCancelRecovery

A Factor in the session's snapshot votes to cancel the recovery with this message. Its earlier choice, if any, is replaced. The moment the cancel votes reach the threshold, the session ends, the account is unlocked and nothing changes. The Factor that started the recovery cannot cancel it alone.

| | |
|---|---|
| Signer | `submitter`, any unlocked SmartAccount |
| Before you send it | The session is running. The Factor is in its snapshot. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx smartaccount vote-cancel-recovery [account] [session-id] [recovery-factor-id]` with `--recovery-proof`, or with `--generate-only` and then `urid tx smartaccount proof recovery [tx-file]`. |

```proto
message MsgVoteCancelRecovery {
  string submitter = 1;
  string account = 2;
  uint64 session_id = 3;
  uint64 recovery_factor_id = 4;
  RecoveryProof recovery_proof = 5;
}

message MsgVoteCancelRecoveryResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| submitter | address | Who submits and signs the transaction. Casts no vote. |
| account | address | The account being recovered. |
| session_id | number | The running session. |
| recovery_factor_id | number | The voting Factor, from the snapshot. |
| recovery_proof | `RecoveryProof` | The Factor's proof over the cancel vote, naming its current `vote_nonce`. |

**Refused when**

- The session ID is not the account's running session.
- The Factor is not in the snapshot.
- The proof's kind does not match the Factor's type, or the proof is invalid or names an old `vote_nonce`.
- A `SMART_ACCOUNT` Factor is not a SmartAccount or is locked.

<a id="smartaccount-tx-governance"></a>

### Governance

<a id="smartaccount-msg-update-params"></a>

#### MsgUpdateParams

This message replaces all of the module's [parameters](#smartaccount-parameters) at once. Send every field, not just the ones that change. A field you leave out becomes 0, not its current value.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | the governance account, through a passed proposal |
| Before you send it | Read the current values with [Params](#smartaccount-query-params) and change what you need. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx smartaccount update-params-proposal [params]`, where `[params]` is the whole parameter set as JSON, with `--title`, `--summary`, `--deposit` and `--metadata`. It submits the proposal. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | `Params` | The complete new parameters. |

Lowering a limit does not touch existing accounts. An account with more credentials than a new `max_credentials` keeps them and can still remove some. The new limit applies only to later additions.

**Refused when**

- The authority is not the governance account.
- The parameters are missing or invalid.

<a id="smartaccount-queries"></a>

## Queries

These queries read an account's current settings, its recovery state, and the module's parameters.

| Query | What it answers |
|---|---|
| [SmartAccount](#smartaccount-query-smart-account) | The whole record of an account. |
| [Credentials](#smartaccount-query-credentials) | The account's active credentials. |
| [MinApprovals](#smartaccount-query-min-approvals) | The approval threshold. |
| [AuthVersion](#smartaccount-query-auth-version) | The Auth Policy version. |
| [OwnerAccount](#smartaccount-query-owner-account) | The Owner. |
| [RecoveryPolicy](#smartaccount-query-recovery-policy) | The Recovery Factors and the recovery threshold. |
| [PendingRecoveryPolicyChange](#smartaccount-query-pending-recovery-policy-change) | The waiting recovery policy change, if any. |
| [ActiveRecoverySession](#smartaccount-query-active-recovery-session) | The running recovery session, if any. |
| [RecoveryFactorChoices](#smartaccount-query-recovery-factor-choices) | Every Factor's current choice and vote counter. |
| [RecoveryProposalVoteCounts](#smartaccount-query-recovery-proposal-vote-counts) | The proposals with their votes, and the cancel count. |
| [Params](#smartaccount-query-params) | The module's parameters. |

An address that is not a SmartAccount answers not found. None of the lists is paged, because each is bounded by the account's own credentials and Factors.

<a id="smartaccount-q-account"></a>

### Account settings

<a id="smartaccount-query-smart-account"></a>

#### SmartAccount

Returns the whole stored record of an account. Ask it before you build a transaction, to see the credentials, the threshold and the status.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/SmartAccount` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}` |
| urid | `urid query smartaccount smart-account [address]` |

```proto
message QuerySmartAccountRequest {
  string address = 1;
}

message QuerySmartAccountResponse {
  SmartAccount smart_account = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| smart_account | `SmartAccount` | The record. See [SmartAccount record](#smartaccount-record). |

<a id="smartaccount-query-credentials"></a>

#### Credentials

Returns the account's active credentials. A wallet asks it to find the `credential_id` values to sign with or to remove.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/Credentials` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/credentials` |
| urid | `urid query smartaccount credentials [address]` |

```proto
message QueryCredentialsRequest {
  string address = 1;
}

message QueryCredentialsResponse {
  repeated Credential credentials = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| credentials | list of `Credential` | The active credentials. See [Credential](#smartaccount-credential). |

<a id="smartaccount-query-min-approvals"></a>

#### MinApprovals

Returns how many credentials must approve each transaction of the account.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/MinApprovals` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/min_approvals` |
| urid | `urid query smartaccount min-approvals [address]` |

```proto
message QueryMinApprovalsRequest {
  string address = 1;
}

message QueryMinApprovalsResponse {
  uint32 min_approvals = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| min_approvals | number | The approval threshold. |

<a id="smartaccount-query-auth-version"></a>

#### AuthVersion

Returns the Auth Policy version. A change in this number tells a wallet that the credentials or the threshold changed.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/AuthVersion` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/auth_version` |
| urid | `urid query smartaccount auth-version [address]` |

```proto
message QueryAuthVersionRequest {
  string address = 1;
}

message QueryAuthVersionResponse {
  uint64 auth_version = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| auth_version | number | The Auth Policy version. |

<a id="smartaccount-query-owner-account"></a>

#### OwnerAccount

Returns the account's Owner. Ask it before a [MsgSetOwner](#smartaccount-msg-set-owner) to know who must sign.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/OwnerAccount` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/owner_account` |
| urid | `urid query smartaccount owner-account [address]` |

```proto
message QueryOwnerAccountRequest {
  string address = 1;
}

message QueryOwnerAccountResponse {
  string owner_account = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| owner_account | address | The Owner. Empty when the account has none. |

<a id="smartaccount-q-recovery"></a>

### Recovery state

<a id="smartaccount-query-recovery-policy"></a>

#### RecoveryPolicy

Returns the account's active Recovery Factors with their IDs, the recovery threshold and the policy version.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/RecoveryPolicy` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/recovery_policy` |
| urid | `urid query smartaccount recovery-policy [address]` |

```proto
message QueryRecoveryPolicyRequest {
  string address = 1;
}

message QueryRecoveryPolicyResponse {
  RecoveryPolicy recovery_policy = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| recovery_policy | `RecoveryPolicy` | See [Recovery policy](#smartaccount-recovery-policy). |

<a id="smartaccount-query-pending-recovery-policy-change"></a>

#### PendingRecoveryPolicyChange

Returns the recovery policy change that is waiting for its safety delay, and when it will apply. Check it regularly. A change you did not request is a sign that someone has your keys, and you can still cancel it while it waits.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/PendingRecoveryPolicyChange` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/pending_recovery_policy_change` |
| urid | `urid query smartaccount pending-recovery-policy-change [address]` |

```proto
message QueryPendingRecoveryPolicyChangeRequest {
  string address = 1;
}

message QueryPendingRecoveryPolicyChangeResponse {
  PendingRecoveryPolicyChange pending_change = 1;
}
```
```proto
message PendingRecoveryPolicyChange {
  string account = 1;
  RecoveryPolicyChangeType type = 2;
  RecoveryFactor new_factor = 3;
  uint64 recovery_factor_id = 4;
  uint32 new_recovery_min_approvals = 5;
  google.protobuf.Timestamp requested_at = 6;
  google.protobuf.Timestamp activate_at = 7;
  google.protobuf.Duration delay_snapshot = 8;
}

enum RecoveryPolicyChangeType {
  RECOVERY_POLICY_CHANGE_TYPE_UNSPECIFIED = 0;
  RECOVERY_POLICY_CHANGE_TYPE_ADD_FACTOR = 1;
  RECOVERY_POLICY_CHANGE_TYPE_REMOVE_FACTOR = 2;
  RECOVERY_POLICY_CHANGE_TYPE_CHANGE_MIN_APPROVALS = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| pending_change | `PendingRecoveryPolicyChange` | The waiting change. Absent when none is waiting. |
| pending_change.account | address | The account. |
| pending_change.type | enum | `RECOVERY_POLICY_CHANGE_TYPE_ADD_FACTOR`, `RECOVERY_POLICY_CHANGE_TYPE_REMOVE_FACTOR` or `RECOVERY_POLICY_CHANGE_TYPE_CHANGE_MIN_APPROVALS`. |
| pending_change.new_factor | `RecoveryFactor` | The Factor to add, for an addition. Its ID is 0 until the change applies. |
| pending_change.recovery_factor_id | number | The Factor to remove, for a removal. 0 otherwise. |
| pending_change.new_recovery_min_approvals | number | The recovery threshold the change sets. |
| pending_change.requested_at | timestamp | When the change was requested. |
| pending_change.activate_at | timestamp | When it applies, at the end of the first block at or after this time. |
| pending_change.delay_snapshot | duration | The safety delay in force at the request. A later parameter change does not affect it. |

<a id="smartaccount-query-active-recovery-session"></a>

#### ActiveRecoverySession

Returns the account's running recovery session in full. Its `session_id` is the `[session-id]` the two vote commands take.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/ActiveRecoverySession` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/active_recovery_session` |
| urid | `urid query smartaccount active-recovery-session [address]` |

```proto
message QueryActiveRecoverySessionRequest {
  string address = 1;
}

message QueryActiveRecoverySessionResponse {
  RecoverySession session = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account to look up. |
| session | `RecoverySession` | The running session, with every choice and proposal. Absent when none is running. See [Recovery session](#smartaccount-recovery-session). |

<a id="smartaccount-query-recovery-factor-choices"></a>

#### RecoveryFactorChoices

Returns the current choice and vote counter of every Factor in the running session's snapshot, including Factors that have not chosen. A Factor reads its `vote_nonce` here before it votes, because its next vote must name that value.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/RecoveryFactorChoices` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/recovery_factor_choices` |
| urid | `urid query smartaccount recovery-factor-choices [address]` |

```proto
message QueryRecoveryFactorChoicesRequest {
  string address = 1;
}

message QueryRecoveryFactorChoicesResponse {
  uint64 session_id = 1;
  repeated RecoveryFactorChoice choices = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account being recovered. |
| session_id | number | The running session, or 0 when none is running. |
| choices | list of `RecoveryFactorChoice` | One entry per Factor in snapshot order. A Factor that has not chosen shows `RECOVERY_CHOICE_NONE` and `vote_nonce` 0. Empty when no session is running. See [Recovery session](#smartaccount-recovery-session). |

<a id="smartaccount-query-recovery-proposal-vote-counts"></a>

#### RecoveryProposalVoteCounts

Returns every proposal in the running session that holds at least one vote, with its count, and the number of cancel votes. Compare each count with the session's threshold to see how close a decision is.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/RecoveryProposalVoteCounts` |
| REST | `GET /urinet/smartaccount/v1/accounts/{address}/recovery_proposal_vote_counts` |
| urid | `urid query smartaccount recovery-proposal-vote-counts [address]` |

```proto
message QueryRecoveryProposalVoteCountsRequest {
  string address = 1;
}

message QueryRecoveryProposalVoteCountsResponse {
  uint64 session_id = 1;
  repeated RecoveryProposalRecord proposals = 2;
  uint32 cancel_count = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account being recovered. |
| session_id | number | The running session, or 0 when none is running. |
| proposals | list of `RecoveryProposalRecord` | Each proposal's hash, its sorted content and its `vote_count`, ordered by hash. Empty when no session is running. |
| cancel_count | number | How many Factors currently choose cancel. |

<a id="smartaccount-q-params"></a>

### Module parameters

<a id="smartaccount-query-params"></a>

#### Params

Returns the module's current parameters.

| | |
|---|---|
| gRPC | `urinet.smartaccount.v1.Query/Params` |
| REST | `GET /urinet/smartaccount/v1/params` |
| urid | `urid query smartaccount params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | `Params` | See [Parameters](#smartaccount-parameters). |

<a id="smartaccount-parameters"></a>

## Parameters

Parameters are the module's network-wide settings. Governance changes them with [MsgUpdateParams](#smartaccount-msg-update-params) inside a proposal.

```proto
message Params {
  google.protobuf.Duration recovery_voting_period = 1;
  google.protobuf.Duration recovery_factor_add_delay = 2;
  uint32 max_recovery_factors = 3;
  uint32 max_credentials = 4;
  uint32 max_account_name_bytes = 5;
  uint32 max_metadata_uri_bytes = 6;
  uint32 max_auth_proofs_per_tx = 7;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| recovery_voting_period | How long a recovery session stays open for votes. | 7 days |
| recovery_factor_add_delay | The safety delay before any recovery policy change applies: adding or removing a Factor, or changing the recovery threshold. | 7 days |
| max_recovery_factors | The most active Recovery Factors an account may have when it adds one. | 7 |
| max_credentials | The most credentials an account may have when it adds one. | 10 |
| max_account_name_bytes | The longest display name, in bytes. | 128 |
| max_metadata_uri_bytes | The longest metadata link, in bytes. | 512 |
| max_auth_proofs_per_tx | The most credential proofs one transaction may carry across all its signature bundles. | 20 |

A running session and a waiting change keep the period and delay that were in force when they began.

---

<a id="ch-delegation"></a>

# Delegation: x/delegation

<a id="delegation-overview"></a>

## What it is

A delegation lets one account act for another within written limits. The account that gives the authority is the delegator. The account that receives it is the delegate. Alice can give her agent a delegation to pay the shop up to a fixed amount. The agent then sends Alice's coins without holding her keys.

The delegate signs with its own credentials, the keys registered to its own account. Assets always leave the delegator's balance. A delegation names exactly one delegator and one delegate, and an account cannot delegate to itself. This is not a staking delegation, which stakes coins with a validator.

Each delegation holds a list of permissions. A permission is one allowed task. It carries a description and plain-language rules for people and agents to read, and chain rules that the validators enforce. The only chain rule is the SEND rule, which caps how much of one denom the delegate may send and to whom. The chain does not read the plain-language rules. An agent is expected to follow them, and a counterparty can check them.

A delegation cannot be edited. To change one, revoke it and create a new one. It can be used from `valid_from` until just before `expires_at`, and never for longer than 90 days. The authority ends at the exact moment `expires_at` arrives, even if the record is cleaned up a little later. The delegator can suspend a delegation and resume it later. Either party can revoke it for good.

A delegation does not let the delegate change the delegator's credentials, recovery settings, or owner. It does not let the delegate create delegations, vote, or stake. The delegate spends under a delegation in two ways: [MsgDelegatedSend](#delegation-msg-delegated-send) for bank coins, and the token module's [approved transfer](#token-msg-approved-transfer) with a [DelegationRef](#delegation-type-delegation-ref).

While an account is in the middle of a [recovery](#how-recovery), the chain locks it. Every delegation message that involves a locked account is refused until the recovery ends. A recovery that replaces an account's credentials suspends every active delegation that account gave. The delegator can resume them afterwards with [MsgResumeDelegation](#delegation-msg-resume-delegation).

<a id="delegation-type-delegation"></a>

### The delegation record

```proto
message Delegation {
  uint64 delegation_id = 1;
  string delegator = 2;
  string delegate = 3;
  string description = 4;
  repeated Permission permissions = 5;
  repeated DataEntry data = 6;
  google.protobuf.Timestamp valid_from = 7;
  google.protobuf.Timestamp expires_at = 8;
  string max_fee_total = 9;
  DelegationStatus status = 10;
  google.protobuf.Timestamp created_at = 11;
}

enum DelegationStatus {
  DELEGATION_STATUS_UNSPECIFIED = 0;
  DELEGATION_STATUS_ACTIVE = 1;
  DELEGATION_STATUS_SUSPENDED = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation's number. The chain gives it, from 1 upward across the whole chain. A number is never reused. |
| delegator | address | The account that gives the authority and whose assets are used. |
| delegate | address | The account that uses the authority. |
| description | string | The overall purpose, for people to read. |
| permissions | list of [`Permission`](#delegation-type-permission) | The allowed tasks. A permission's position in the list, counted from 0, is its `permission_index`. |
| data | list of [`DataEntry`](#delegation-type-data-entry) | Key and value entries the chain keeps but does not interpret. |
| valid_from | timestamp | The first moment the delegation can be used. |
| expires_at | timestamp | The end. The delegation can be used only while the block time is before this moment. |
| max_fee_total | integer as a string | The most network fee, in kwak, the delegate can make the delegator pay through this delegation. See [Fees paid by the delegator](#delegation-funded-fees). |
| status | `DelegationStatus` | `DELEGATION_STATUS_ACTIVE` or `DELEGATION_STATUS_SUSPENDED`. A revoked or expired delegation has no status, because its record is deleted. |
| created_at | timestamp | The block time at creation. |

Every field from `delegator` to `max_fee_total` is fixed at creation. A delegation that is active but whose `valid_from` is still ahead cannot be used yet.

<a id="delegation-type-permission"></a>

### Permission

```proto
message Permission {
  string description = 1;
  repeated string rules = 2;
  repeated SendRule chain_rules = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| description | string | What the task covers, for example "Book, modify, and cancel hotels." |
| rules | list of strings | Real-world conditions in plain language, one rule per entry. The chain does not interpret them. |
| chain_rules | list of [`SendRule`](#delegation-type-send-rule) | The rules the validators enforce. A rule's position in this list, counted from 0, is its `rule_index`. |

<a id="delegation-type-send-rule"></a>

### SEND rule

A SEND rule lets the delegate send one denom from the delegator's balance. Every SEND rule must have a cumulative cap, so an unlimited delegation cannot exist.

```proto
message SendRule {
  ChainRuleType type = 1;
  string denom = 2;
  RecipientScope recipient_scope = 3;
  repeated string recipients = 4;
  string max_total = 5;
  urinet.base.v1.LimitAmount max_per_tx = 6;
  google.protobuf.UInt64Value max_uses = 7;
}

enum RecipientScope {
  RECIPIENT_SCOPE_UNSPECIFIED = 0;
  RECIPIENT_SCOPE_ANY = 1;
  RECIPIENT_SCOPE_ALLOWLIST = 2;
}

enum ChainRuleType {
  CHAIN_RULE_TYPE_UNSPECIFIED = 0;
  CHAIN_RULE_TYPE_SEND = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| type | `ChainRuleType` | Always `CHAIN_RULE_TYPE_SEND`. |
| denom | string | The denom this rule limits, such as `kwak` or `token/<issuer address>/<subdenom>`. |
| recipient_scope | `RecipientScope` | `RECIPIENT_SCOPE_ANY` allows any recipient. `RECIPIENT_SCOPE_ALLOWLIST` allows only the addresses in `recipients`. |
| recipients | list of addresses | The allowed recipients when the scope is `RECIPIENT_SCOPE_ALLOWLIST`. Must be empty when the scope is `RECIPIENT_SCOPE_ANY`. |
| max_total | integer as a string | Required. The most the delegate may send through this rule over the whole life of the delegation. A value of 0 makes the rule unusable. |
| max_per_tx | `LimitAmount`, optional | The most one delegated send may move, written as `{"value": "..."}`. It applies to each message, not to the whole transaction. Leave it out for no per-send cap. |
| max_uses | number, optional | How many successful sends the rule allows. Leave it out for no count limit. |

When one denom has several SEND rules in the same permission, every one of them applies, and a send must satisfy them all. If you need one rule or another, put them in separate permissions.

Here is one permission with a SEND rule, as you pass it to `--permissions`. The agent may pay one shop address up to 50 URI in total, at most 10 URI per send, and at most 12 times.

```json
{
  "description": "Pay the monthly hosting invoice.",
  "rules": ["Pay only invoices issued by the hosting shop."],
  "chain_rules": [
    {
      "type": "CHAIN_RULE_TYPE_SEND",
      "denom": "kwak",
      "recipient_scope": "RECIPIENT_SCOPE_ALLOWLIST",
      "recipients": ["uri1..."],
      "max_total": "50000000000",
      "max_per_tx": {"value": "10000000000"},
      "max_uses": "12"
    }
  ]
}
```

<a id="delegation-type-data-entry"></a>

### Data entries

The `data` list holds references to outside material, such as a hash of a payment mandate or a provider reference. The chain keeps the entries in the order given and does not interpret them. They are never a source of authority. Keys must be unique. Put a hash or a short reference here, not a large document.

```proto
message DataEntry {
  string key = 1;
  string value = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| key | string | The entry's key, unique within the delegation. |
| value | string | The entry's value. |

<a id="delegation-type-send-meter"></a>

### SEND meters

The chain keeps one meter for each SEND rule. It counts what the delegate has sent through that rule. A refund does not lower the meter, because it measures how much authority was used.

```proto
message SendMeter {
  uint64 delegation_id = 1;
  uint32 permission_index = 2;
  uint32 rule_index = 3;
  string spent_total = 4;
  google.protobuf.UInt64Value use_count = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation. |
| permission_index | number | The permission, counted from 0. |
| rule_index | number | The rule within that permission's `chain_rules`, counted from 0. |
| spent_total | integer as a string | The total sent through this rule so far, in the rule's denom. |
| use_count | number, optional | The number of successful sends. Present only when the rule has `max_uses`. |

<a id="delegation-type-delegation-ref"></a>

### DelegationRef

A `DelegationRef` points at one permission of one delegation. The token module's [approved transfer](#token-msg-approved-transfer) carries it when an agent moves the delegator's tokens under a delegation. That transfer checks the same SEND rules and moves the same meters as [MsgDelegatedSend](#delegation-msg-delegated-send).

```proto
message DelegationRef {
  uint64 delegation_id = 1;
  uint32 permission_index = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation, at least 1. |
| permission_index | number | The permission to use, counted from 0. |

<a id="delegation-funded-fees"></a>

### Fees paid by the delegator

A delegation can pay the network fee of the transactions sent under it. The delegate signs and is the fee payer, and the fee granter is set to the delegator's address. With `urid`, add `--fee-granter` with the delegator's address. The chain then takes the fee from the delegator's balance and adds it to the delegation's fee meter, `fee_spent_total`.

The fee meter may never pass the delegation's `max_fee_total`. A delegation with `max_fee_total` of 0 cannot pay fees. A transaction whose fee the delegator pays must use exactly one delegation and carry nothing else. Every message in it must be a [MsgDelegatedSend](#delegation-msg-delegated-send) or an [approved transfer](#token-msg-approved-transfer) with a `DelegationRef`, and all of them must name the same `delegation_id`. They may use different permissions. A transaction that uses a delegation and names its delegator as fee granter is refused if it breaks these rules, or if its fee payer is not the delegate.

Fees paid another way leave the fee meter alone. The delegate can pay its own fee, a sponsor can sign as the fee payer, or a third party can give a fee grant. If the message itself fails after the fee is charged, the fee and the fee meter stay charged, and the transfer and the SEND meters roll back.

<a id="delegation-transactions"></a>

## Transactions

Six messages create, control, and use delegations. One of them is for governance only.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgCreateDelegation](#delegation-msg-create-delegation) | Creates a new, active delegation. | The delegator |
| [MsgSuspendDelegation](#delegation-msg-suspend-delegation) | Pauses an active delegation. | The delegator |
| [MsgResumeDelegation](#delegation-msg-resume-delegation) | Reactivates a suspended delegation. | The delegator |
| [MsgRevokeDelegation](#delegation-msg-revoke-delegation) | Ends a delegation for good. | The delegator or the delegate |
| [MsgDelegatedSend](#delegation-msg-delegated-send) | Sends the delegator's coins under a permission. | The delegate |
| [MsgUpdateParams](#delegation-msg-update-params) | Replaces the module's parameters. | Governance only |

Each `urid tx delegation` command takes the signing account from `--from`. It can sign and send directly only for an account with one credential whose address is the key's address, and only with an explicit `--gas`. Otherwise, write the transaction with `--generate-only` and sign it with `urid tx sign`. See [Transactions with urid](#urid-tx) and [How transactions work](#how-transactions). When the chain refuses a message, the log carries an error code listed in [Errors](#appendix-errors).

<a id="delegation-tx-lifecycle"></a>

### The lifecycle

<a id="delegation-msg-create-delegation"></a>

#### MsgCreateDelegation

Creates a delegation from the signer to the delegate. The new delegation is active at once, and it can be used from `valid_from`. Its meters start at 0.

The response is empty. To find the new `delegation_id`, run [DelegationsByDelegator](#delegation-query-delegations-by-delegator) for the delegator, or read the `urinet.delegation.v1.EventDelegationCreated` event in the transaction result. That event carries the whole new record. See [Delegation events](#events-delegation).

Creating a delegation costs extra gas for the space it occupies. The extra gas is the size of `description`, `permissions`, and `data` in bytes, times the number of days from creation to `expires_at`, measured to the second, times `storage_gas_per_byte_day`. At the value of 1, a 300-byte delegation kept for 90 days costs 27,000 extra gas. It is charged once, and revoking early refunds none of it.

| | |
|---|---|
| Signer | `delegator`, the account that gives the authority |
| Before you send it | Both accounts have been created as SmartAccounts and neither is in a recovery. The delegator has fewer live delegations than the limit. |
| What comes back | Nothing. The new `delegation_id` is in the event and in DelegationsByDelegator. |
| urid | `urid tx delegation create-delegation [delegate] [valid-from] [expires-at] [max-fee-total]` |

The two times are RFC 3339, such as `2026-12-01T00:00:00Z`. `max-fee-total` is in kwak. The content goes in flags. `--description` takes a string. `--permissions` takes one JSON permission and can be repeated. `--data` takes one JSON entry `{"key": "...", "value": "..."}` and can be repeated, and the order is kept.

```sh
urid tx delegation create-delegation uri1... 2026-12-01T00:00:00Z 2027-02-01T00:00:00Z 2000000000 \
  --description "Hosting payments" \
  --permissions "$(cat permission.json)" \
  --from uri1... --generate-only > unsigned.json
```

```proto
message MsgCreateDelegation {
  string delegator = 1;
  string delegate = 2;
  string description = 3;
  repeated Permission permissions = 4;
  repeated DataEntry data = 5;
  google.protobuf.Timestamp valid_from = 6;
  google.protobuf.Timestamp expires_at = 7;
  string max_fee_total = 8;
}

message MsgCreateDelegationResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegator | address | The account that gives the authority. Must sign. |
| delegate | address | The account that receives it. |
| description | string | The overall purpose. |
| permissions | list of [`Permission`](#delegation-type-permission) | The allowed tasks. |
| data | list of [`DataEntry`](#delegation-type-data-entry) | Outside references, unique keys, kept in order. |
| valid_from | timestamp | The first moment of use. |
| expires_at | timestamp | The end, not included. |
| max_fee_total | integer as a string | The cap, in kwak, on fees the delegator pays for the delegate. |

**Refused when**

- The delegator and the delegate are the same account.
- Either account has not been created as a SmartAccount, or either is in a recovery.
- `valid_from` is not before `expires_at`, or `expires_at` has already passed.
- `expires_at` is more than 90 days after `valid_from`.
- There are too many permissions, rules, chain rules, or allowlist recipients.
- A text field, a data entry, or the whole content is larger than its limit, or two data entries share a key.
- A SEND rule is malformed, such as a type other than SEND, a bad denom, an unknown scope, or a negative cap.
- The delegator would pass `max_live_delegations_per_delegator`, which counts active and suspended delegations.

<a id="delegation-msg-suspend-delegation"></a>

#### MsgSuspendDelegation

Pauses an active delegation. The delegate cannot use it while it is suspended. The meters, the fee meter, and the expiry time stay as they are, and the expiry clock keeps running.

| | |
|---|---|
| Signer | `delegator`, the delegator of the delegation |
| Before you send it | The delegation is active and not expired. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx delegation suspend-delegation [delegation-id]` |

```proto
message MsgSuspendDelegation {
  string delegator = 1;
  uint64 delegation_id = 2;
}

message MsgSuspendDelegationResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegator | address | The delegator. Must sign. |
| delegation_id | number | The delegation to suspend. |

**Refused when**

- No delegation has this id.
- The signer is not the delegation's delegator.
- The delegation is already suspended.
- The delegation has expired.
- The delegator or the delegate is in a recovery.

<a id="delegation-msg-resume-delegation"></a>

#### MsgResumeDelegation

Makes a suspended delegation active again. It also works on a delegation that a recovery suspended. The expiry time does not move.

| | |
|---|---|
| Signer | `delegator`, the delegator of the delegation |
| Before you send it | The delegation is suspended and not expired. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx delegation resume-delegation [delegation-id]` |

```proto
message MsgResumeDelegation {
  string delegator = 1;
  uint64 delegation_id = 2;
}

message MsgResumeDelegationResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegator | address | The delegator. Must sign. |
| delegation_id | number | The delegation to resume. |

**Refused when**

- No delegation has this id.
- The signer is not the delegation's delegator.
- The delegation is not suspended.
- The delegation has expired.
- The delegator or the delegate is in a recovery.

<a id="delegation-msg-revoke-delegation"></a>

#### MsgRevokeDelegation

Ends a delegation for good. Either the delegator or the delegate can send it. The chain deletes the record and its meters, so queries on the id then answer not found. The history stays in blocks, events, and the indexer.

| | |
|---|---|
| Signer | `signer`, the delegator or the delegate |
| Before you send it | The delegation still exists. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx delegation revoke-delegation [delegation-id]` |

```proto
message MsgRevokeDelegation {
  string signer = 1;
  uint64 delegation_id = 2;
}

message MsgRevokeDelegationResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| signer | address | The delegator or the delegate. Must sign. |
| delegation_id | number | The delegation to end. |

**Refused when**

- No delegation has this id.
- The signer is neither the delegator nor the delegate.
- An account in the delegation is in a recovery.

<a id="delegation-tx-spending"></a>

### Spending under a delegation

<a id="delegation-msg-delegated-send"></a>

#### MsgDelegatedSend

Sends coins from the delegator's balance to a recipient, under one permission of a delegation. The delegate signs with its own credentials and its own sequence. One send can carry several denoms, and it succeeds or fails as a whole.

The chain checks each denom against the SEND rules of the chosen permission. Each denom needs at least one matching rule, and every matching rule must pass. On success the coins move and every matching meter goes up. When the agent pays the shop 5 URI under Alice's delegation, the 5 URI leave Alice's balance.

| | |
|---|---|
| Signer | `delegate`, the delegate of the delegation |
| Before you send it | The delegation is active, the block time is between `valid_from` and `expires_at`, and the delegator holds the coins. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx delegation delegated-send [delegation-id] [permission-index] [to-address] [amount]...` |

Give each amount as a coin, such as `5000000000kwak`. To have the delegator pay the fee, see [Fees paid by the delegator](#delegation-funded-fees).

```proto
message MsgDelegatedSend {
  string delegate = 1;
  uint64 delegation_id = 2;
  uint32 permission_index = 3;
  string to_address = 4;
  repeated cosmos.base.v1beta1.Coin amount = 5;
}

message MsgDelegatedSendResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegate | address | The delegate. Must sign. |
| delegation_id | number | The delegation to use. |
| permission_index | number | The permission to use, counted from 0. |
| to_address | address | The recipient. |
| amount | list of coins | What to send. Each denom appears once, the denoms are in ascending order, and every amount is positive. |

**Refused when**

- No delegation has this id, or the signer is not its delegate.
- The delegation is suspended, not yet valid, or expired.
- The delegator, the delegate, or the recipient is in a recovery.
- `permission_index` is past the end of the permission list.
- The amount is empty, repeats a denom, is out of order, or has an amount that is not positive.
- A denom has no matching SEND rule, or the recipient is not on the allowlist.
- The send would pass `max_per_tx`, `max_total`, or `max_uses`, or the delegator's balance is too low.
- The delegator pays the fee and the fee would push the fee meter past `max_fee_total`.

<a id="delegation-tx-governance"></a>

### Governance

<a id="delegation-msg-update-params"></a>

#### MsgUpdateParams

Replaces all the module's parameters at once. A field you leave out becomes 0, not its current value. Read the current values first with [Params](#delegation-query-params). A lower cap does not touch delegations that already exist. It applies to new delegations only.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values and change only the ones you mean to. |
| What comes back | Nothing. The proposal result tells you it went through. |
| urid | `urid tx delegation update-params-proposal [params]`, where `[params]` is the whole parameter set as JSON, with `--title`, `--summary`, `--deposit` and `--metadata`. It submits the proposal. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#delegation-parameters) | The whole new set of parameters. |

**Refused when**

- The signer is not the governance account.
- `params` is missing.
- A count or size limit is 0, or `max_delegation_duration` is not positive. Only `storage_gas_per_byte_day` may be 0.

<a id="delegation-queries"></a>

## Queries

These queries read delegations, their meters, and the module's parameters. An unknown `delegation_id` answers with the gRPC status `NotFound`.

| Query | What it answers |
|---|---|
| [Delegation](#delegation-query-delegation) | The whole record of one delegation. |
| [DelegationsByDelegator](#delegation-query-delegations-by-delegator) | Every live delegation an account has given. |
| [DelegationMeters](#delegation-query-delegation-meters) | The SEND meters and the fee meter of one delegation. |
| [DelegationStatus](#delegation-query-delegation-status) | Whether one delegation is active or suspended. |
| [ValidFrom](#delegation-query-valid-from) | When one delegation starts. |
| [ExpiresAt](#delegation-query-expires-at) | When one delegation ends. |
| [Params](#delegation-query-params) | The module's parameters. |

The queries return what is recorded and make no time decision. A delegation past its `expires_at` stays readable until it is cleaned up. That happens at the end of the first block at or after `expires_at`, or in a later block when many delegations expire at once. In that time its status reads as it was, but it can no longer be used. Compare `expires_at` with the block time yourself.

<a id="delegation-query-records"></a>

### Delegations

<a id="delegation-query-delegation"></a>

#### Delegation

Returns the whole record of one delegation, with its permissions and data.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/Delegation` |
| REST | `GET /urinet/delegation/v1/delegations/{delegation_id}` |
| urid | `urid query delegation delegation [delegation-id]` |

```proto
message QueryDelegationRequest {
  uint64 delegation_id = 1;
}

message QueryDelegationResponse {
  Delegation delegation = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation to read. |
| delegation | [`Delegation`](#delegation-type-delegation) | In the response. The record, with `delegation_id` set. |

<a id="delegation-query-delegations-by-delegator"></a>

#### DelegationsByDelegator

Lists every live delegation one account has given, active and suspended, by id from lowest to highest. Use it to find the id of a delegation you just created. The list is not paged, because an account can have at most `max_live_delegations_per_delegator` live delegations.

The chain has no query for the delegations an account has received. The indexer keeps that list.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/DelegationsByDelegator` |
| REST | `GET /urinet/delegation/v1/delegators/{delegator}/delegations` |
| urid | `urid query delegation delegations-by-delegator [delegator]` |

```proto
message QueryDelegationsByDelegatorRequest {
  string delegator = 1;
}

message QueryDelegationsByDelegatorResponse {
  repeated Delegation delegations = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator | address | The account whose delegations you want. |
| delegations | list of [`Delegation`](#delegation-type-delegation) | In the response. The live records, each with `delegation_id` set. Empty if the account has given none. |

<a id="delegation-query-meters-times"></a>

### Meters and times

<a id="delegation-query-delegation-meters"></a>

#### DelegationMeters

Returns how much of a delegation has been used: one meter per SEND rule, and the fee meter. Ask it before a send to see what room is left under each rule.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/DelegationMeters` |
| REST | `GET /urinet/delegation/v1/delegations/{delegation_id}/meters` |
| urid | `urid query delegation meters [delegation-id]` |

```proto
message QueryDelegationMetersRequest {
  uint64 delegation_id = 1;
}

message QueryDelegationMetersResponse {
  repeated SendMeter send_meters = 1;
  string fee_spent_total = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation to read. |
| send_meters | list of [`SendMeter`](#delegation-type-send-meter) | In the response. One meter per SEND rule, ordered by permission and then by rule. |
| fee_spent_total | integer as a string | In the response. The fees, in kwak, charged to the delegator so far under this delegation. |

<a id="delegation-query-delegation-status"></a>

#### DelegationStatus

Returns whether a delegation is active or suspended.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/DelegationStatus` |
| REST | `GET /urinet/delegation/v1/delegations/{delegation_id}/status` |
| urid | `urid query delegation status [delegation-id]` |

```proto
message QueryDelegationStatusRequest {
  uint64 delegation_id = 1;
}

message QueryDelegationStatusResponse {
  DelegationStatus status = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation to read. |
| status | `DelegationStatus` | In the response. `DELEGATION_STATUS_ACTIVE` or `DELEGATION_STATUS_SUSPENDED`. |

<a id="delegation-query-valid-from"></a>

#### ValidFrom

Returns the first moment a delegation can be used.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/ValidFrom` |
| REST | `GET /urinet/delegation/v1/delegations/{delegation_id}/valid_from` |
| urid | `urid query delegation valid-from [delegation-id]` |

```proto
message QueryValidFromRequest {
  uint64 delegation_id = 1;
}

message QueryValidFromResponse {
  google.protobuf.Timestamp valid_from = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation to read. |
| valid_from | timestamp | In the response. The start of use. |

<a id="delegation-query-expires-at"></a>

#### ExpiresAt

Returns the moment a delegation stops working. The delegation can be used only while the block time is before it.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/ExpiresAt` |
| REST | `GET /urinet/delegation/v1/delegations/{delegation_id}/expires_at` |
| urid | `urid query delegation expires-at [delegation-id]` |

```proto
message QueryExpiresAtRequest {
  uint64 delegation_id = 1;
}

message QueryExpiresAtResponse {
  google.protobuf.Timestamp expires_at = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegation_id | number | The delegation to read. |
| expires_at | timestamp | In the response. The end, not included. |

<a id="delegation-query-parameters"></a>

### Parameters

<a id="delegation-query-params"></a>

#### Params

Returns the module's current parameters.

| | |
|---|---|
| gRPC | `urinet.delegation.v1.Query/Params` |
| REST | `GET /urinet/delegation/v1/params` |
| urid | `urid query delegation params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#delegation-parameters) | In the response. The current parameters. |

<a id="delegation-parameters"></a>

## Parameters

The parameters set the limits on new delegations. Governance changes them with [MsgUpdateParams](#delegation-msg-update-params) inside a proposal. A lower limit never invalidates a delegation that already exists.

```proto
message Params {
  uint32 max_permissions_per_delegation = 1;
  uint32 max_rules_per_permission = 2;
  uint32 max_chain_rules_per_permission = 3;
  uint32 max_text_field_bytes = 4;
  uint32 max_data_entries = 5;
  uint32 max_data_key_bytes = 6;
  uint32 max_data_value_bytes = 7;
  uint32 max_data_total_bytes = 8;
  uint32 max_delegation_content_bytes = 9;
  uint32 max_recipients_per_send_rule = 10;
  uint32 max_live_delegations_per_delegator = 11;
  google.protobuf.Duration max_delegation_duration = 12;
  uint32 max_expiry_cleanup_per_block = 13;
  uint64 storage_gas_per_byte_day = 14;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| max_permissions_per_delegation | The most permissions in one delegation. | 10 |
| max_rules_per_permission | The most plain-language rules in one permission. | 5 |
| max_chain_rules_per_permission | The most SEND rules in one permission. | 5 |
| max_text_field_bytes | The most bytes in the delegation's description, a permission's description, or one rule. | 256 |
| max_data_entries | The most entries in `data`. | 8 |
| max_data_key_bytes | The most bytes in one data key. | 64 |
| max_data_value_bytes | The most bytes in one data value. | 256 |
| max_data_total_bytes | The most bytes in all of `data` together. | 512 |
| max_delegation_content_bytes | The most bytes of `description`, `permissions`, and `data` together, as encoded. | 8192 |
| max_recipients_per_send_rule | The most addresses in one SEND rule's allowlist. | 20 |
| max_live_delegations_per_delegator | The most active and suspended delegations one account can have given. | 32 |
| max_delegation_duration | The longest time from `valid_from` to `expires_at`. | 90 days |
| max_expiry_cleanup_per_block | The most expired delegations the chain deletes in one block. The rest wait for the next block. | 1000 |
| storage_gas_per_byte_day | Extra gas per byte of content per day, charged once at creation. | 1 |

---

<a id="ch-token"></a>

# Tokens: x/token

<a id="token-overview"></a>

## What it is

The token module lets anyone issue a token of their own on UriNet. A user token is any token created this way, as distinct from the native coin URI. The issuer is the account that creates the token. It chooses the token's character with nine settings. They decide whether the token can be minted, paused, frozen, burned or recovered, who may receive it, and how much one account may hold.

Balances and total supply stay in the bank module, like every other coin. The token module keeps the token's rules beside them. These are its settings, who may run which operation, the minters, the two lists, and the frozen amounts. It applies those rules every time the token moves, whichever module moves it.

A plain send of a user token has no message of its own here. You send it with the bank module's `MsgSend` or `MsgMultiSend`, or through a delegation, and the token's rules apply on the way. The messages in this chapter are for issuing and running a token, and for the few movements that need more than the holder's signature.

The chain has no token types. An ordinary token, a stablecoin and a security token are only different combinations of the same settings. The chain does not screen issuers either. Telling a real token from an imitation is the wallet's job. A wallet should identify a token by its full denom, never by its name or symbol.

<a id="token-denom"></a>

### Denom and identity

A user token's denom is `token/<issuer address>/<subdenom>`, for example `token/uri1.../usdc`. The chain builds it from the issuer's address and the subdenom, a short machine name the issuer picks. You never type the full denom when you create a token. The CreateToken response returns it.

The subdenom uses lower-case ASCII letters, digits, `.`, `_` and `-`, and starts with a letter or a digit. Its length limit is a module parameter, see [Parameters](#token-parameters). Because the issuer's address is part of the denom, two issuers can use the same subdenom without a clash.

The name, symbol and decimals are fixed for good at creation, even if they contain a typo. They live in the token's bank metadata, together with a description and a document reference that can change later. The `token/` prefix is reserved. Every message that names a token expects the full denom with that prefix.

<a id="token-issuer-root"></a>

### The issuer root and permissions

The issuer is the token's root, its top authority. That status comes from the token record itself, so nobody can grant it, revoke it or move it to another account. The root can run every management operation and can hand out nine ordinary permissions to other accounts. See [Permissions](#token-type-permissions).

A setting says whether a feature is available. A permission says who may run it. If a setting is off, nobody can use the feature, the root included. Even the root cannot break a change rule, a permanent lock, the maximum supply, the maximum holder count or a governance pause.

Minting is not a permission. An account may mint only if it has a minter record, which also carries how much it may still mint. The issuer needs its own minter record to mint, like anyone else.

Three sanctions can never target the issuer's account: a freeze, a blocklist entry, and a forced transfer out of it. The issuer has no permission record of its own either, because it never needs one. On the allowlist the issuer is an ordinary account. Under the "allowlist only" rule it needs its own entry to receive the token.

<a id="token-movement-rules"></a>

### Rules on every movement

Whenever a user token moves through the bank module, the chain checks these rules in order:

- The source and the recipient must be different accounts.
- The token must not be paused by governance or by the issuer.
- Neither the source nor the recipient may be on the blocklist.
- Under "allowlist only", the recipient must be on the allowlist. Under "approval required", a plain send is refused, and the move must use [MsgApprovedTransfer](#token-msg-approved-transfer).
- The amount must fit in the source's usable balance.
- If a holding limit is on, the recipient's new balance must not exceed it.

If an account is on both lists, the blocklist wins. The maximum holder count works differently. The chain counts holders once, at the end of each transaction. A count that goes over the cap and back within one transaction is fine. A holder is a non-module account with a balance above zero.

<a id="token-pauses"></a>

### Two kinds of pause

The issuer pause is the issuer's own switch, available while the pause setting is on. It stops plain transfers, approved transfers, minting and burning. Forced transfers and management operations still work, so the issuer can clean up an incident while the token is stopped.

The governance pause is the chain's emergency brake, and it needs no opt-in from the issuer. While it is on, every movement and every change to that token is refused, forced transfers included. Queries still work. Only governance can lift it, through [MsgSetGovernanceHardPause](#token-msg-set-governance-hard-pause). The two pauses are independent, and lifting one leaves the other as it was.

<a id="token-usable-balance"></a>

### Usable balance

The usable balance is what an account can spend of a token right now. It is the balance minus the part still locked by vesting, minus the frozen amount, and never below zero. A frozen amount may be larger than the balance. The excess then also holds back coins that arrive later. [UsableBalance](#token-query-usable-balance) reports the parts as `vesting_locked`, `frozen_amount` and `token_usable`. The figure looks only at the balance, vesting and freeze. A positive one does not promise that a transfer will pass the other rules.

<a id="token-settings-lock"></a>

### Locking settings for good

Every setting has a value, a change rule and a permanent lock. The change rule is chosen at creation and never changes. It says whether the value may later go up, go down, both, or neither. The permanent lock fixes the current value forever once it is on. Nobody can unlock it: not the issuer, not an admin, not governance.

There are two ways to lock a setting. Set `permanent_lock` to `true` in the setting when you send [MsgCreateToken](#token-msg-create-token). Or, later, have the issuer send [MsgLockTokenSettings](#token-msg-lock-token-settings). An admin cannot lock. A lock also wins over the change rule. A maximum supply under "decrease only" can no longer be lowered once it is locked.

<a id="token-storage-gas"></a>

### Storage gas

There is no rent and no token-only fee. A transaction pays extra gas when it creates a new token record, permission record, minter record, freeze or list entry. The extra gas is `storage_entry_create_gas` plus `storage_growth_per_byte_gas` for each byte stored. Their starting values are 20,000 and 100. Growing an existing entry costs `storage_growth_per_byte_gas` per added byte. Shrinking or deleting costs nothing extra and refunds nothing. Changes to bank metadata pay only the normal gas.

<a id="token-type-token"></a>

### The Token record

The token record holds a token's identity, settings and pause states. Queries such as [Token](#token-query-token) return it.

```proto
message Token {
  string denom = 1;
  string issuer = 2;
  TokenSettings settings = 3;
  bool paused_by_issuer = 4;
  bool paused_by_gov = 5;
  google.protobuf.Timestamp created_at = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The full denom, `token/<issuer address>/<subdenom>`. |
| issuer | address | The account that created the token, its root. It never changes. |
| settings | [`TokenSettings`](#token-type-settings) | The nine settings. |
| paused_by_issuer | boolean | Whether the issuer pause is on. |
| paused_by_gov | boolean | Whether the governance pause is on. |
| created_at | timestamp | When the token was created, in block time. |

<a id="token-type-settings"></a>

### TokenSettings

`TokenSettings` holds the nine settings. When you create a token, all nine must be present, each with a value and a change rule.

```proto
message TokenSettings {
  ToggleSetting mint_enabled = 1;
  MaxSupplySetting max_supply = 2;
  ToggleSetting pause_enabled = 3;
  ToggleSetting freeze_enabled = 4;
  ToggleSetting forced_transfer_enabled = 5;
  ToggleSetting burn_enabled = 6;
  TransferRestrictionSetting transfer_restriction = 7;
  AmountLimitSetting holding_limit = 8;
  CountLimitSetting max_holder_count = 9;
}
```
| Field | Type | Meaning |
|---|---|---|
| mint_enabled | [`ToggleSetting`](#token-type-setting-records) | Whether minters may mint. Turning it off keeps the minter records. |
| max_supply | [`MaxSupplySetting`](#token-type-setting-records) | The most that may exist at once, in base units. It can never be set below the current supply. |
| pause_enabled | [`ToggleSetting`](#token-type-setting-records) | Whether the issuer pause may be used. It can only be turned off while the token is not paused. |
| freeze_enabled | [`ToggleSetting`](#token-type-setting-records) | Whether frozen amounts may be raised and blocklist entries added or changed. While it is off, existing freezes and entries stay, and they can still be lowered or removed. |
| forced_transfer_enabled | [`ToggleSetting`](#token-type-setting-records) | Whether [MsgForcedTransfer](#token-msg-forced-transfer) may be used. |
| burn_enabled | [`ToggleSetting`](#token-type-setting-records) | Whether the issuer and burn holders may burn their own coins. It does not stop [MsgRejectVestingSchedules](#token-msg-reject-vesting-schedules). |
| transfer_restriction | [`TransferRestrictionSetting`](#token-type-setting-records) | Who may receive the token: anyone, only allowlisted accounts, or only through an approved transfer. |
| holding_limit | [`AmountLimitSetting`](#token-type-setting-records) | The most one non-module account may hold, or no limit. Lowering it never takes coins away. It only refuses new receipts above it. |
| max_holder_count | [`CountLimitSetting`](#token-type-setting-records) | The most non-module accounts that may hold the token at once, or no limit. It can never be set below the current count. |

This is what `--settings` on `create-token` looks like. The values are only an example. Add `"permanent_lock": true` to a setting to lock it from the start.

```json
{
  "mint_enabled": {"value": {"value": "TOGGLE_ON"}, "change_rule": "TOGGLE_CHANGE_RULE_FREE"},
  "max_supply": {"value": {"amount": "1000000"}, "change_rule": "NUMBER_CHANGE_RULE_INCREASE_ONLY"},
  "pause_enabled": {"value": {"value": "TOGGLE_ON"}, "change_rule": "TOGGLE_CHANGE_RULE_DISABLE_ONLY"},
  "freeze_enabled": {"value": {"value": "TOGGLE_ON"}, "change_rule": "TOGGLE_CHANGE_RULE_FREE"},
  "forced_transfer_enabled": {"value": {"value": "TOGGLE_OFF"}, "change_rule": "TOGGLE_CHANGE_RULE_FIXED"},
  "burn_enabled": {"value": {"value": "TOGGLE_ON"}, "change_rule": "TOGGLE_CHANGE_RULE_ENABLE_ONLY"},
  "transfer_restriction": {"value": {"value": "TRANSFER_RESTRICTION_ALLOWLIST_ONLY"}, "change_rule": "TRANSFER_RESTRICTION_CHANGE_RULE_RELAX_ONLY"},
  "holding_limit": {"value": {"mode": "LIMIT_MODE_ENABLED", "amount": {"value": "500000"}}, "change_rule": "NUMBER_CHANGE_RULE_FREE"},
  "max_holder_count": {"value": {"mode": "LIMIT_MODE_DISABLED"}, "change_rule": "NUMBER_CHANGE_RULE_DECREASE_ONLY"}
}
```

<a id="token-type-setting-records"></a>

### Setting records

Each setting is one of five records. They all have the same three fields and differ only in the kind of value and change rule.

```proto
message ToggleSetting {
  ToggleValue value = 1;
  ToggleChangeRule change_rule = 2;
  bool permanent_lock = 3;
}

message MaxSupplySetting {
  MaxSupplyValue value = 1;
  NumberChangeRule change_rule = 2;
  bool permanent_lock = 3;
}

message TransferRestrictionSetting {
  TransferRestrictionValue value = 1;
  TransferRestrictionChangeRule change_rule = 2;
  bool permanent_lock = 3;
}

message AmountLimitSetting {
  AmountLimitValue value = 1;
  NumberChangeRule change_rule = 2;
  bool permanent_lock = 3;
}

message CountLimitSetting {
  CountLimitValue value = 1;
  NumberChangeRule change_rule = 2;
  bool permanent_lock = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| value | a value type, see [Setting values](#token-type-setting-values) | The current value. |
| change_rule | enum, see below | Which way the value may change later. Chosen at creation and fixed for good. `UNSPECIFIED` is refused. |
| permanent_lock | boolean | Once `true`, the value can never change again. It can only go from `false` to `true`. Left out, it is `false`. |

The change rules come in three families:

```proto
enum ToggleChangeRule {
  TOGGLE_CHANGE_RULE_UNSPECIFIED = 0;
  TOGGLE_CHANGE_RULE_FIXED = 1;
  TOGGLE_CHANGE_RULE_DISABLE_ONLY = 2;
  TOGGLE_CHANGE_RULE_ENABLE_ONLY = 3;
  TOGGLE_CHANGE_RULE_FREE = 4;
}

enum NumberChangeRule {
  NUMBER_CHANGE_RULE_UNSPECIFIED = 0;
  NUMBER_CHANGE_RULE_FIXED = 1;
  NUMBER_CHANGE_RULE_INCREASE_ONLY = 2;
  NUMBER_CHANGE_RULE_DECREASE_ONLY = 3;
  NUMBER_CHANGE_RULE_FREE = 4;
}

enum TransferRestrictionChangeRule {
  TRANSFER_RESTRICTION_CHANGE_RULE_UNSPECIFIED = 0;
  TRANSFER_RESTRICTION_CHANGE_RULE_FIXED = 1;
  TRANSFER_RESTRICTION_CHANGE_RULE_TIGHTEN_ONLY = 2;
  TRANSFER_RESTRICTION_CHANGE_RULE_RELAX_ONLY = 3;
  TRANSFER_RESTRICTION_CHANGE_RULE_FREE = 4;
}
```
| Rule | Meaning |
|---|---|
| `FIXED` | The value never changes from its initial value. |
| `FREE` | The value may change in either direction. |
| `DISABLE_ONLY`, `ENABLE_ONLY` | For on and off settings. Only on to off, or only off to on. |
| `INCREASE_ONLY`, `DECREASE_ONLY` | For number settings. Only up, or only down. |
| `TIGHTEN_ONLY`, `RELAX_ONLY` | For the transfer restriction. Only toward stricter, or only toward looser. Skipping a step is allowed. |

For the holding limit and the maximum holder count, "no limit" counts as infinity when the chain compares directions. Under "decrease only", a setting that starts with no limit may later get a number and go lower. It can never go back to no limit.

<a id="token-type-setting-values"></a>

### Setting values

A value type carries just the value. `MsgUpdateTokenSettings` uses the same types for the new values, and the `update-token-settings` flags take them as JSON.

```proto
enum Toggle {
  TOGGLE_UNSPECIFIED = 0;
  TOGGLE_OFF = 1;
  TOGGLE_ON = 2;
}

enum TransferRestriction {
  TRANSFER_RESTRICTION_UNSPECIFIED = 0;
  TRANSFER_RESTRICTION_FREE = 1;
  TRANSFER_RESTRICTION_ALLOWLIST_ONLY = 2;
  TRANSFER_RESTRICTION_APPROVAL_REQUIRED = 3;
}

enum LimitMode {
  LIMIT_MODE_UNSPECIFIED = 0;
  LIMIT_MODE_DISABLED = 1;
  LIMIT_MODE_ENABLED = 2;
}

message ToggleValue {
  Toggle value = 1;
}

message MaxSupplyValue {
  string amount = 1;
}

message TransferRestrictionValue {
  TransferRestriction value = 1;
}

message AmountLimitValue {
  LimitMode mode = 1;
  urinet.base.v1.LimitAmount amount = 2;
}

message CountLimitValue {
  LimitMode mode = 1;
  google.protobuf.UInt64Value count = 2;
}
```
```proto
message LimitAmount {
  string value = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| `ToggleValue.value` | enum | `TOGGLE_ON` or `TOGGLE_OFF`. `TOGGLE_UNSPECIFIED`, or an empty `{}`, is refused. |
| `MaxSupplyValue.amount` | integer as a string | The maximum supply in base units. Zero or more. |
| `TransferRestrictionValue.value` | enum | `TRANSFER_RESTRICTION_FREE`: anyone may receive it. `TRANSFER_RESTRICTION_ALLOWLIST_ONLY`: the recipient must be on the allowlist. `TRANSFER_RESTRICTION_APPROVAL_REQUIRED`: every move needs an approved transfer. The order from loose to strict is free, allowlist only, approval required. |
| `AmountLimitValue.mode` | enum | `LIMIT_MODE_DISABLED` for no limit, or `LIMIT_MODE_ENABLED` for a limit. `LIMIT_MODE_UNSPECIFIED` is refused. |
| `AmountLimitValue.amount` | `LimitAmount` | The limit in base units, in `value`. Required when the mode is enabled and forbidden when it is disabled. Zero is valid and means no account can newly receive the token. |
| `CountLimitValue.mode` | enum | As for `AmountLimitValue.mode`. |
| `CountLimitValue.count` | number | The cap on holders. Required when the mode is enabled and forbidden when it is disabled. Zero is valid. In JSON, write it as a string, such as `"50"`. |

<a id="token-type-setting-key"></a>

### TokenSettingKey

`TokenSettingKey` names one setting. You use it to choose what to lock, and responses use it to say what changed. The `--keys` flag of `lock-token-settings` takes the short form, such as `max-supply` or `holding-limit`.

```proto
enum TokenSettingKey {
  TOKEN_SETTING_KEY_UNSPECIFIED = 0;
  TOKEN_SETTING_KEY_MINT_ENABLED = 1;
  TOKEN_SETTING_KEY_MAX_SUPPLY = 2;
  TOKEN_SETTING_KEY_PAUSE_ENABLED = 3;
  TOKEN_SETTING_KEY_FREEZE_ENABLED = 4;
  TOKEN_SETTING_KEY_FORCED_TRANSFER_ENABLED = 5;
  TOKEN_SETTING_KEY_BURN_ENABLED = 6;
  TOKEN_SETTING_KEY_TRANSFER_RESTRICTION = 7;
  TOKEN_SETTING_KEY_HOLDING_LIMIT = 8;
  TOKEN_SETTING_KEY_MAX_HOLDER_COUNT = 9;
}
```
The description and the document reference have no key, because they cannot be locked.

<a id="token-type-permissions"></a>

### Permissions

The root can grant nine ordinary permissions. One account can hold several of them.

```proto
enum TokenPermission {
  TOKEN_PERMISSION_UNSPECIFIED = 0;
  TOKEN_PERMISSION_ADMIN = 1;
  TOKEN_PERMISSION_MANAGE_MINTERS = 2;
  TOKEN_PERMISSION_BURN = 3;
  TOKEN_PERMISSION_PAUSE = 4;
  TOKEN_PERMISSION_FREEZE = 5;
  TOKEN_PERMISSION_FORCED_TRANSFER = 6;
  TOKEN_PERMISSION_UPDATE_METADATA = 7;
  TOKEN_PERMISSION_MANAGE_ALLOWLIST = 8;
  TOKEN_PERMISSION_APPROVE_TRANSFER = 9;
}

message AccountPermissions {
  string account = 1;
  repeated TokenPermission permissions = 2;
}
```
| Permission | What its holder may do |
|---|---|
| `TOKEN_PERMISSION_ADMIN` | Change setting values, and grant or revoke the other eight permissions, to itself as well. Only the root grants ADMIN or takes it from someone else. An admin can resign its own ADMIN. |
| `TOKEN_PERMISSION_MANAGE_MINTERS` | Create, change and remove minter records, except the issuer's own. |
| `TOKEN_PERMISSION_BURN` | Burn its own balance. |
| `TOKEN_PERMISSION_PAUSE` | Turn the issuer pause on and off. |
| `TOKEN_PERMISSION_FREEZE` | Set frozen amounts and manage the blocklist. |
| `TOKEN_PERMISSION_FORCED_TRANSFER` | Run forced transfers. |
| `TOKEN_PERMISSION_UPDATE_METADATA` | Change the description and the document reference. |
| `TOKEN_PERMISSION_MANAGE_ALLOWLIST` | Manage the allowlist. |
| `TOKEN_PERMISSION_APPROVE_TRANSFER` | Approve transfers of a token under "approval required". |

`AccountPermissions` pairs one account with a set of permissions. Grants, revocations and the initial grants of a new token all use it.

| Field | Type | Meaning |
|---|---|---|
| account | address | The account that gains or loses the permissions. It cannot be the issuer or a module account. It may be an address that has no account on the chain yet. |
| permissions | list of `TokenPermission` | At least one, with no duplicate. `TOKEN_PERMISSION_UNSPECIFIED` is refused. |

A permission can be granted while its feature is off. It simply has no effect until the setting allows the feature.

<a id="token-type-document"></a>

### DocumentReference

A document reference points to an outside document about the token, such as a prospectus, and carries its hash so readers can check it. The URI and the hash always travel as a pair.

```proto
message DocumentReference {
  string uri = 1;
  string uri_hash = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| uri | string | The document's URI. It is ASCII with a scheme, such as `https:` or `ipfs:`, and no spaces. Its length limit is a [parameter](#token-parameters). |
| uri_hash | string | The SHA-256 of the document, as exactly 64 lower-case hex characters with no `0x`. |

Both empty means no reference. Both filled is a reference. One without the other is refused. The chain checks only the format. It never fetches the document.

<a id="token-type-creation-entries"></a>

### Creation entries

`MsgCreateToken` takes three lists that set up the token's first allowlist, minters and supply in the same step.

```proto
message InitialAllowlistEntry {
  string account = 1;
  bytes data = 2;
}

message InitialMinter {
  string minter = 1;
  string remaining_mint_allowance = 2;
}

message InitialAllocation {
  string recipient = 1;
  string unlocked_amount = 2;
  repeated urinet.vesting.v1.VestingScheduleSpec vesting_schedules = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| `InitialAllowlistEntry.account` | address | An account to put on the allowlist. The issuer and module accounts are allowed. |
| `InitialAllowlistEntry.data` | bytes, base64 in JSON | Any data the issuer wants to keep with the entry, such as a case number. The chain does not read it. It may be empty. |
| `InitialMinter.minter` | address | An account that becomes a minter. It may be the issuer, but not a module account. |
| `InitialMinter.remaining_mint_allowance` | integer as a string | How much it may mint. Zero is valid. The allowances are not compared with the maximum supply. |
| `InitialAllocation.recipient` | address | A first holder. It may be the issuer, but not a module account. Each recipient appears once. |
| `InitialAllocation.unlocked_amount` | integer as a string | The part the recipient can use at once. |
| `InitialAllocation.vesting_schedules` | list of [`VestingScheduleSpec`](#vesting-schedule-input) | Parts that unlock over time. Each schedule's denom must be the token being created. |

Each recipient receives the unlocked amount plus the total of its schedules. An allocation with neither is refused. The initial supply is the sum of all allocations, and it must not exceed the maximum supply. It skips the mint setting and the minter records, because it happens once in the token's life. It still has to obey the holding limit and, under "allowlist only", the allowlist.

<a id="token-type-list-entries"></a>

### List entries and permission changes

```proto
message BlocklistEntry {
  string account = 1;
  bytes data = 2;
}

message AllowlistEntry {
  string account = 1;
  bytes data = 2;
}

message PermissionChange {
  string account = 1;
  repeated TokenPermission changed_permissions = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| `BlocklistEntry.account` | address | The account to block. It cannot be the issuer or a module account. It may have no account on the chain yet. |
| `BlocklistEntry.data` | bytes, base64 in JSON | Issuer data kept with the entry. Its size limit is a [parameter](#token-parameters). It may be empty. |
| `AllowlistEntry.account` | address | The account to allow. The issuer and module accounts are allowed. |
| `AllowlistEntry.data` | bytes, base64 in JSON | Issuer data kept with the entry. Its size limit is a [parameter](#token-parameters). It may be empty. |
| `PermissionChange.account` | address | In a response. An account whose permissions actually changed. |
| `PermissionChange.changed_permissions` | list of `TokenPermission` | In a response. Only the permissions added or removed this time. |

The chain uses only whether an entry exists. It never interprets the data.

<a id="token-type-vesting"></a>

### Vesting types carried by token messages

A vesting schedule keeps coins locked until set times. It is a list of releases, each an amount that unlocks at a set time. Two token messages carry vesting data, in types that the vesting module defines.

[MsgCreateToken](#token-msg-create-token) uses `VestingScheduleSpec` and `VestingReleaseSpec` to lock part of an initial allocation. Their fields are in [Schedule input](#vesting-schedule-input). [MsgForcedTransfer](#token-msg-forced-transfer) uses `VestingReleaseSelection` to pick locked releases at the source and `DestinationVesting` to say how the coins arrive. Its response reports what happened with `ReducedVestingRelease` and `PreservedScheduleMapping`. These are in [Vesting in a forced transfer](#vesting-forced-transfer). The rest of vesting is in [Vesting](#ch-vesting).

<a id="token-transactions"></a>

## Transactions

The token module accepts 21 messages, two of them for governance only.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgCreateToken](#token-msg-create-token) | Creates a token with its settings, first holders, minters and lists. | The issuer |
| [MsgUpdateTokenSettings](#token-msg-update-token-settings) | Changes one or more setting values. | The issuer or an admin |
| [MsgLockTokenSettings](#token-msg-lock-token-settings) | Locks chosen settings for good. | The issuer |
| [MsgUpdateTokenMetadata](#token-msg-update-token-metadata) | Changes the description or the document reference. | The issuer or a metadata holder |
| [MsgGrantPermissions](#token-msg-grant-permissions) | Grants permissions to accounts. | The issuer or an admin |
| [MsgRevokePermissions](#token-msg-revoke-permissions) | Revokes permissions from accounts. | The issuer or an admin |
| [MsgSetMinter](#token-msg-set-minter) | Creates a minter or sets its allowance. | The issuer or a minter manager |
| [MsgRemoveMinter](#token-msg-remove-minter) | Removes a minter. | The issuer or a minter manager |
| [MsgMint](#token-msg-mint) | Creates new coins for a recipient. | A minter |
| [MsgBurn](#token-msg-burn) | Destroys coins from the sender's own balance. | The issuer or a burn holder |
| [MsgSetIssuerPause](#token-msg-set-issuer-pause) | Turns the issuer pause on or off. | The issuer or a pause holder |
| [MsgSetFrozenAmount](#token-msg-set-frozen-amount) | Sets how much of an account's balance is frozen. | The issuer or a freeze holder |
| [MsgUpsertBlocklist](#token-msg-upsert-blocklist) | Adds or updates blocklist entries. | The issuer or a freeze holder |
| [MsgDeleteBlocklist](#token-msg-delete-blocklist) | Removes blocklist entries. | The issuer or a freeze holder |
| [MsgUpsertAllowlist](#token-msg-upsert-allowlist) | Adds or updates allowlist entries. | The issuer or an allowlist holder |
| [MsgDeleteAllowlist](#token-msg-delete-allowlist) | Removes allowlist entries. | The issuer or an allowlist holder |
| [MsgApprovedTransfer](#token-msg-approved-transfer) | Moves a token that needs approval. | The holder or its agent, and an approver |
| [MsgForcedTransfer](#token-msg-forced-transfer) | Moves a holder's coins without the holder's signature. | The issuer or a forced transfer holder |
| [MsgRejectVestingSchedules](#token-msg-reject-vesting-schedules) | Removes unwanted vesting schedules and burns their locked coins. | The holder |
| [MsgSetGovernanceHardPause](#token-msg-set-governance-hard-pause) | Turns the governance pause on or off. | Governance |
| [MsgUpdateParams](#token-msg-update-params) | Replaces the module's parameters. | Governance |

Each `urid tx token` command takes its signer from `--from`. To sign, generate the transaction with `--generate-only` and sign it with `urid tx sign`, which estimates the gas for you. See [Transactions with urid](#urid-tx). Signing directly with `--from` works only for an account with a single credential whose address is the key's address, and only with an explicit `--gas`. In the examples, upper-case words such as `KEY` and `SOURCE_ADDRESS` stand for your own values. Error names and codes are listed in [Errors](#appendix-errors).

Most management messages succeed without doing anything when the requested state is already in place. Granting a permission the account already has is one case. Such a no-op writes nothing and emits no event, and the response's counts or flags show that nothing changed. Every message on a token is refused while governance has paused it, except the unpause itself.

<a id="token-tx-create"></a>

### Creating a token

<a id="token-msg-create-token"></a>

#### MsgCreateToken

Creates a new user token in one step. The signer becomes the issuer and root for good. The same message can set up the first allowlist, grant permissions, appoint minters, and hand out the initial supply with optional vesting. Anyone may create a token. There is no fee beyond the transaction's gas.

The issuer picks the subdenom, name, symbol and decimals, and these never change afterwards. The chain builds the bank metadata from them. With decimals above zero, the symbol becomes the display unit at that exponent. With zero decimals, the display unit is the full denom. A new token starts with both pauses off. Check every value before you sign, because a mistake means creating a new token.

In the example, `SETTINGS` holds the JSON shown under [TokenSettings](#token-type-settings). Its "allowlist only" rule is why the issuer puts itself on the allowlist before taking the first allocation.

```sh
urid tx token create-token usdc "Shop Dollar" SHOPD 6 \
  --settings "$SETTINGS" \
  --description "Store credit of the shop" \
  --initial-allowlist '{"account":"ISSUER_ADDRESS"}' \
  --initial-grants '{"account":"OPERATOR_ADDRESS","permissions":["TOKEN_PERMISSION_FREEZE"]}' \
  --initial-minters '{"minter":"ISSUER_ADDRESS","remaining_mint_allowance":"500000"}' \
  --initial-allocations '{"recipient":"ISSUER_ADDRESS","unlocked_amount":"100000"}' \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `issuer`, the account that creates the token |
| Before you send it | The denom `token/<issuer>/<subdenom>` must not exist yet, not even as bank supply or metadata. |
| What comes back | The full denom and the initial supply. |
| urid | `urid tx token create-token [subdenom] [name] [symbol] [decimals]` with `--settings`, `--description`, `--document`, and the repeatable `--initial-allowlist`, `--initial-grants`, `--initial-minters` and `--initial-allocations`, one JSON entry per flag |

```proto
message MsgCreateToken {
  string issuer = 1;
  string subdenom = 2;
  string name = 3;
  string symbol = 4;
  uint32 decimals = 5;
  TokenSettings settings = 6;
  string description = 7;
  DocumentReference document = 8;
  repeated InitialAllowlistEntry initial_allowlist = 9;
  repeated AccountPermissions initial_grants = 10;
  repeated InitialMinter initial_minters = 11;
  repeated InitialAllocation initial_allocations = 12;
}

message MsgCreateTokenResponse {
  string denom = 1;
  string initial_supply = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| issuer | address | The creator, who becomes the root. Must sign. Cannot be a module account. |
| subdenom | string | The short machine name inside the issuer's namespace. See [Denom and identity](#token-denom). |
| name | string | The display name. Any valid UTF-8, with no leading or trailing spaces and no invisible or control characters. Its length limit is a [parameter](#token-parameters). |
| symbol | string | The short label. At least 3 characters, starting with an ASCII letter, then letters, digits, `.`, `_` or `-`. Case is kept as typed. Its maximum length is a [parameter](#token-parameters). |
| decimals | number | 0 to 18. How many decimal places the display unit has. |
| settings | [`TokenSettings`](#token-type-settings) | All nine settings. Required. |
| description | string | The first description. May be empty. Its length limit is a [parameter](#token-parameters). |
| document | [`DocumentReference`](#token-type-document) | The first document reference. Leave it out for none. |
| initial_allowlist | list of [`InitialAllowlistEntry`](#token-type-creation-entries) | Accounts put on the allowlist at once. |
| initial_grants | list of [`AccountPermissions`](#token-type-permissions) | Permissions granted at once, ADMIN included. |
| initial_minters | list of [`InitialMinter`](#token-type-creation-entries) | Minter records created at once. |
| initial_allocations | list of [`InitialAllocation`](#token-type-creation-entries) | The initial supply and who receives it. Empty means the token starts with no supply. |
| denom | string | In the response. The full denom, to use in every later message. |
| initial_supply | integer as a string | In the response. The total of all allocations. |

**Refused when**

- The settings are missing, a setting is incomplete, or a value or change rule is `UNSPECIFIED`.
- The subdenom breaks the character rules or is too long, or the denom is already taken.
- The name or symbol is invalid, or matches a reserved word such as the native coin's name or symbol, ignoring ASCII case.
- The decimals are above 18, or the document reference has only one of its two parts.
- A list has a duplicate account, a grant targets the issuer, or an entry targets a module account where that is not allowed.
- The initial allowlist or the initial grants have more entries than a [parameter](#token-parameters) allows.
- An allocation is empty, names the same recipient twice, or has a vesting schedule for another denom.
- The initial supply would exceed the maximum supply.
- A recipient would go over the holding limit, or is not allowlisted under "allowlist only".

The maximum holder count is checked at the end of the transaction. A later message in the same transaction can still bring the count down.

<a id="token-tx-settings"></a>

### Settings and locks

<a id="token-msg-update-token-settings"></a>

#### MsgUpdateTokenSettings

Changes the values of one or more settings of an existing token. Each setting has its own optional field. A field you leave out keeps its value, and at least one must be present. The change must follow the setting's change rule, and a locked setting cannot change at all. The issuer pause does not block this message.

On the command line each setting is its own JSON flag, holding one of the [setting values](#token-type-setting-values):

```sh
urid tx token update-token-settings token/uri1.../usdc \
  --max-supply '{"amount":"2000000"}' \
  --mint-enabled '{"value":"TOGGLE_OFF"}' \
  --holding-limit '{"mode":"LIMIT_MODE_ENABLED","amount":{"value":"800"}}' \
  --max-holder-count '{"mode":"LIMIT_MODE_DISABLED"}' \
  --from KEY --generate-only
```

To turn the pause setting off while the token is paused, send the unpause first. Both messages can go in one transaction, in that order.

| | |
|---|---|
| Signer | `sender`, the issuer or an ADMIN holder |
| Before you send it | Each change must move in a direction the setting's change rule allows, and the setting must not be locked. |
| What comes back | How many settings changed, and which ones. A value equal to the current one is not counted. |
| urid | `urid tx token update-token-settings [denom]` with any of `--mint-enabled`, `--max-supply`, `--pause-enabled`, `--freeze-enabled`, `--forced-transfer-enabled`, `--burn-enabled`, `--transfer-restriction`, `--holding-limit`, `--max-holder-count` |

```proto
message MsgUpdateTokenSettings {
  string sender = 1;
  string denom = 2;
  ToggleValue mint_enabled = 3;
  MaxSupplyValue max_supply = 4;
  ToggleValue pause_enabled = 5;
  ToggleValue freeze_enabled = 6;
  ToggleValue forced_transfer_enabled = 7;
  ToggleValue burn_enabled = 8;
  TransferRestrictionValue transfer_restriction = 9;
  AmountLimitValue holding_limit = 10;
  CountLimitValue max_holder_count = 11;
}

message MsgUpdateTokenSettingsResponse {
  uint32 changed_count = 1;
  repeated TokenSettingKey changed_keys = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or an admin. Must sign. |
| denom | string | The token. |
| mint_enabled, pause_enabled, freeze_enabled, forced_transfer_enabled, burn_enabled | [`ToggleValue`](#token-type-setting-values) | New on or off values. Leave out to keep. |
| max_supply | [`MaxSupplyValue`](#token-type-setting-values) | The new maximum supply. |
| transfer_restriction | [`TransferRestrictionValue`](#token-type-setting-values) | The new transfer restriction. |
| holding_limit | [`AmountLimitValue`](#token-type-setting-values) | The new holding limit, or no limit. |
| max_holder_count | [`CountLimitValue`](#token-type-setting-values) | The new holder cap, or no cap. |
| changed_count | number | In the response. How many values actually changed. |
| changed_keys | list of [`TokenSettingKey`](#token-type-setting-key) | In the response. The settings that actually changed. |

**Refused when**

- The sender is neither the issuer nor an admin.
- No setting field is present, or a value is malformed or `UNSPECIFIED`.
- A requested setting is permanently locked. This holds even if you send its current value.
- The change goes against the setting's change rule.
- The new maximum supply is below the current supply.
- The new maximum holder count is below the current number of holders.
- The pause setting would go off while the issuer pause is on.

<a id="token-msg-lock-token-settings"></a>

#### MsgLockTokenSettings

Locks the current values of chosen settings for good. Only the issuer can send it. No admin can, and nothing can undo it. A setting with the `FIXED` change rule can be locked too. You can change a value and lock it in the same transaction by sending the update first. The issuer pause does not block this message.

```sh
urid tx token lock-token-settings token/uri1.../usdc --keys max-supply --keys mint-enabled \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `sender`, the issuer |
| Before you send it | None of the chosen settings may be locked already. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx token lock-token-settings [denom] --keys <key>`, repeatable, with keys `mint-enabled`, `max-supply`, `pause-enabled`, `freeze-enabled`, `forced-transfer-enabled`, `burn-enabled`, `transfer-restriction`, `holding-limit`, `max-holder-count` |

```proto
message MsgLockTokenSettings {
  string sender = 1;
  string denom = 2;
  repeated TokenSettingKey keys = 3;
}

message MsgLockTokenSettingsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer. Must sign. |
| denom | string | The token. |
| keys | list of [`TokenSettingKey`](#token-type-setting-key) | The settings to lock. At least one, with no duplicate. |

**Refused when**

- The sender is not the issuer.
- The list of keys is empty, has a duplicate, or holds `UNSPECIFIED` or an unknown key.
- Any chosen setting is already locked. Then nothing is locked.

<a id="token-tx-metadata"></a>

### Metadata

<a id="token-msg-update-token-metadata"></a>

#### MsgUpdateTokenMetadata

Changes the token's description, its document reference, or both. The name, symbol and decimals cannot change. The issuer pause does not block this message. The token keeps no history of old documents, but each change emits an [event](#events-token) with the old and new reference.

A field you leave out keeps its value, and at least one of the two must be given. On the command line, `--description` alone sets the description, and `--description=` with nothing after it clears it. `--document '{"uri":"","uri_hash":""}'` removes the document reference.

```sh
urid tx token update-token-metadata token/uri1.../usdc \
  --document '{"uri":"ipfs://DOCUMENT_CID","uri_hash":"SHA256_HEX"}' \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `sender`, the issuer or an UPDATE_METADATA holder |
| Before you send it | Prepare the SHA-256 of the document if you set a reference. |
| What comes back | Whether the description changed, and whether the document reference changed. |
| urid | `urid tx token update-token-metadata [denom]` with `--description` and `--document` |

```proto
message MsgUpdateTokenMetadata {
  string sender = 1;
  string denom = 2;
  google.protobuf.StringValue description = 3;
  DocumentReference document = 4;
}

message MsgUpdateTokenMetadataResponse {
  bool description_changed = 1;
  bool document_reference_changed = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a metadata holder. Must sign. |
| denom | string | The token. |
| description | string, optional | The new description. Leave out to keep. An empty string clears it. |
| document | [`DocumentReference`](#token-type-document) | The new reference. Leave out to keep. Both parts empty removes it. |
| description_changed | boolean | In the response. Whether the description actually changed. |
| document_reference_changed | boolean | In the response. Whether the reference actually changed. |

**Refused when**

- The sender is neither the issuer nor a metadata holder.
- Neither the description nor the document is given.
- The description is too long.
- The URI is malformed or too long, or the hash is not 64 lower-case hex characters.
- Only one of the URI and the hash is filled.

<a id="token-tx-permissions"></a>

### Permissions

<a id="token-msg-grant-permissions"></a>

#### MsgGrantPermissions

Grants ordinary [permissions](#token-type-permissions) to one or more accounts. The new permissions add to what each account already has, and take effect at once. Only the issuer may grant ADMIN. An admin may grant the other eight, to itself as well. The number of accounts in one message is limited by a [parameter](#token-parameters).

```sh
urid tx token grant-permissions token/uri1.../usdc \
  --grants '{"account":"OPERATOR_ADDRESS","permissions":["TOKEN_PERMISSION_PAUSE","TOKEN_PERMISSION_FREEZE"]}' \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `sender`, the issuer or an ADMIN holder |
| Before you send it | Nothing beyond the signer's own authority. The target account need not exist on the chain yet. |
| What comes back | How many accounts were in the request, how many changed, and what each changed account gained. |
| urid | `urid tx token grant-permissions [denom] --grants <json>`, repeatable, one `{account, permissions}` per flag |

```proto
message MsgGrantPermissions {
  string sender = 1;
  string denom = 2;
  repeated AccountPermissions grants = 3;
}

message MsgGrantPermissionsResponse {
  uint32 requested_account_count = 1;
  uint32 changed_account_count = 2;
  repeated PermissionChange changes = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or an admin. Must sign. |
| denom | string | The token. |
| grants | list of [`AccountPermissions`](#token-type-permissions) | Each account and the permissions to add. |
| requested_account_count | number | In the response. Accounts in the request. |
| changed_account_count | number | In the response. Accounts that gained at least one permission. |
| changes | list of [`PermissionChange`](#token-type-list-entries) | In the response. What each changed account gained. |

**Refused when**

- The sender is neither the issuer nor an admin.
- The request includes ADMIN and the sender is not the issuer.
- The list is empty, names an account twice, or an account's permissions are empty, repeated or `UNSPECIFIED`.
- A target is the issuer or a module account.
- The request has more accounts than the parameter allows.

<a id="token-msg-revoke-permissions"></a>

#### MsgRevokePermissions

Removes permissions from one or more accounts. Removing a permission the account does not have is not an error. When an account loses its last permission, its permission record is removed. Only the issuer may take ADMIN from another account.

An admin who is not the issuer may resign its own ADMIN. That request must contain exactly one entry, for the sender's own account, with ADMIN as its only permission. The root can always appoint a new admin, even if none is left.

| | |
|---|---|
| Signer | `sender`, the issuer or an ADMIN holder |
| Before you send it | Nothing beyond the signer's own authority. |
| What comes back | How many accounts were in the request, how many changed, and what each changed account lost. |
| urid | `urid tx token revoke-permissions [denom] --revokes <json>`, repeatable, one `{account, permissions}` per flag |

```proto
message MsgRevokePermissions {
  string sender = 1;
  string denom = 2;
  repeated AccountPermissions revokes = 3;
}

message MsgRevokePermissionsResponse {
  uint32 requested_account_count = 1;
  uint32 changed_account_count = 2;
  repeated PermissionChange changes = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer, an admin, or an admin resigning. Must sign. |
| denom | string | The token. |
| revokes | list of [`AccountPermissions`](#token-type-permissions) | Each account and the permissions to remove. |
| requested_account_count | number | In the response. Accounts in the request. |
| changed_account_count | number | In the response. Accounts that lost at least one permission. |
| changes | list of [`PermissionChange`](#token-type-list-entries) | In the response. What each changed account lost. |

**Refused when**

- The sender is neither the issuer nor an admin.
- A non-issuer tries to take ADMIN from another account.
- An admin's resignation mixes ADMIN with other permissions or other accounts.
- The list is empty, names an account twice, or an account's permissions are empty, repeated or `UNSPECIFIED`.
- A target is the issuer or a module account, or the request has too many accounts.

<a id="token-tx-mint-burn"></a>

### Minters, mint and burn

<a id="token-msg-set-minter"></a>

#### MsgSetMinter

Creates a minter record, or sets the allowance of an existing one. The allowance is the new total the minter may still mint, not an amount to add. Setting it to zero keeps the minter but stops it minting for now. Only the issuer can create or change the issuer's own minter record. The issuer pause and the mint setting do not block this message.

| | |
|---|---|
| Signer | `sender`, the issuer or a MANAGE_MINTERS holder |
| Before you send it | Nothing. The minter's account need not exist on the chain yet. |
| What comes back | Whether a new record was created, and whether anything changed. |
| urid | `urid tx token set-minter [denom] [minter] [remaining-mint-allowance]` |

```proto
message MsgSetMinter {
  string sender = 1;
  string denom = 2;
  string minter = 3;
  string remaining_mint_allowance = 4;
}

message MsgSetMinterResponse {
  bool created = 1;
  bool changed = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a minter manager. Must sign. |
| denom | string | The token. |
| minter | address | The account to appoint or adjust. It may be the issuer. |
| remaining_mint_allowance | integer as a string | The new allowance in base units. Zero or more. |
| created | boolean | In the response. Whether a new minter record was created. |
| changed | boolean | In the response. Whether the record actually changed. |

**Refused when**

- The sender is neither the issuer nor a minter manager.
- The minter is the issuer and the sender is not the issuer.
- The minter is a module account.
- The allowance is missing or negative.

<a id="token-msg-remove-minter"></a>

#### MsgRemoveMinter

Deletes a minter record. This is the only way to take away an account's right to mint. Removing a record that does not exist is not an error. The sender's authority is checked first, so a minter manager trying to remove the issuer's record is refused even when there is no record.

| | |
|---|---|
| Signer | `sender`, the issuer or a MANAGE_MINTERS holder |
| Before you send it | Nothing. |
| What comes back | Whether a record was deleted. |
| urid | `urid tx token remove-minter [denom] [minter]` |

```proto
message MsgRemoveMinter {
  string sender = 1;
  string denom = 2;
  string minter = 3;
}

message MsgRemoveMinterResponse {
  bool removed = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a minter manager. Must sign. |
| denom | string | The token. |
| minter | address | The minter to remove. |
| removed | boolean | In the response. Whether an existing record was deleted. |

**Refused when**

- The sender is neither the issuer nor a minter manager.
- The minter is the issuer and the sender is not the issuer.
- The minter is a module account.

<a id="token-msg-mint"></a>

#### MsgMint

Creates new coins of a token and delivers them to a recipient. The minter's allowance goes down by the amount. When it reaches zero, the record stays. The recipient may be the minter itself or the issuer. For example, the shop's treasury account is a minter, and it mints 1,000 base units of store credit to Bob.

| | |
|---|---|
| Signer | `sender`, a minter of the token |
| Before you send it | The sender needs a minter record with enough allowance. The issuer is no exception. |
| What comes back | The minter's allowance after the mint. |
| urid | `urid tx token mint [denom] [recipient] [amount]` |

```proto
message MsgMint {
  string sender = 1;
  string denom = 2;
  string recipient = 3;
  string amount = 4;
}

message MsgMintResponse {
  string new_remaining_mint_allowance = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The minter. Must sign. |
| denom | string | The token. |
| recipient | address | Who receives the new coins. Not a module account. |
| amount | integer as a string | How many base units to create. Must be above zero. |
| new_remaining_mint_allowance | integer as a string | In the response. What the minter may still mint. |

**Refused when**

- The sender has no minter record, or its allowance is less than the amount.
- Minting is turned off, or the token is paused.
- The new supply would exceed the maximum supply.
- The recipient is a module account, is on the blocklist, or is not allowlisted under "allowlist only".
- The recipient would go over the holding limit.
- The amount is zero.

A minter on the blocklist can still mint. To stop a minter, remove its record or set its allowance to zero.

<a id="token-msg-burn"></a>

#### MsgBurn

Destroys coins from the sender's own balance, which lowers the total supply. Nobody can burn another account's coins directly. To remove a holder's coins, the issuer side first moves them to its own account with [MsgForcedTransfer](#token-msg-forced-transfer) and then burns them. The burn comes out of the usable balance, so locked and frozen coins cannot be burned.

| | |
|---|---|
| Signer | `sender`, the issuer or a BURN holder |
| Before you send it | The sender's usable balance must cover the amount. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx token burn [denom] [amount]` |

```proto
message MsgBurn {
  string sender = 1;
  string denom = 2;
  string amount = 3;
}

message MsgBurnResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The burner, whose own coins are destroyed. Must sign. |
| denom | string | The token. |
| amount | integer as a string | How many base units to destroy. Must be above zero. |

**Refused when**

- Burning is turned off, or the token is paused.
- The sender is neither the issuer nor a burn holder.
- The sender is on the blocklist, or is a module account.
- The amount exceeds the sender's usable balance, or is zero.

<a id="token-tx-pause-freeze"></a>

### Pause and freeze

<a id="token-msg-set-issuer-pause"></a>

#### MsgSetIssuerPause

Turns the issuer pause on or off. While it is on, plain transfers, approved transfers, minting and burning stop. Forced transfers and management messages still work. See [Two kinds of pause](#token-pauses). The pause setting must be on, even to unpause.

| | |
|---|---|
| Signer | `sender`, the issuer or a PAUSE holder |
| Before you send it | The token's pause setting must be on. |
| What comes back | Whether the pause state changed. |
| urid | `urid tx token set-issuer-pause [denom] [paused]`, where `paused` is `true` or `false` |

```proto
message MsgSetIssuerPause {
  string sender = 1;
  string denom = 2;
  bool paused = 3;
}

message MsgSetIssuerPauseResponse {
  bool changed = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a pause holder. Must sign. |
| denom | string | The token. |
| paused | boolean | `true` to pause, `false` to unpause. |
| changed | boolean | In the response. Whether the state actually changed. |

**Refused when**

- The pause setting is off.
- The sender is neither the issuer nor a pause holder.

<a id="token-msg-set-frozen-amount"></a>

#### MsgSetFrozenAmount

Sets how much of one account's balance of a token is frozen. You give the final amount, not a change. Zero removes the freeze. The amount may be larger than the balance, and the account need not exist yet. That way a freeze can also hold back coins that arrive later. For example, the issuer freezes 500 base units of Bob's balance while a dispute is open. Bob can still spend what is above that.

Raising a freeze needs the freeze setting on. Lowering or removing one is always allowed, so a freeze never gets stuck after the setting is turned off.

| | |
|---|---|
| Signer | `sender`, the issuer or a FREEZE holder |
| Before you send it | To raise a freeze, the freeze setting must be on. |
| What comes back | Whether the frozen amount changed. |
| urid | `urid tx token set-frozen-amount [denom] [account] [frozen-amount]` |

```proto
message MsgSetFrozenAmount {
  string sender = 1;
  string denom = 2;
  string account = 3;
  string frozen_amount = 4;
}

message MsgSetFrozenAmountResponse {
  bool changed = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a freeze holder. Must sign. |
| denom | string | The token. |
| account | address | The account to freeze. Not the issuer and not a module account. |
| frozen_amount | integer as a string | The new frozen amount in base units. Zero removes the freeze. |
| changed | boolean | In the response. Whether the amount actually changed. |

**Refused when**

- The sender is neither the issuer nor a freeze holder.
- The account is the issuer or a module account.
- The amount is missing or negative.
- The amount goes up while the freeze setting is off.

<a id="token-tx-lists"></a>

### Blocklist and allowlist

The blocklist stops an account from sending or receiving the token in the normal way. The allowlist names the accounts that may receive it under "allowlist only". Allowlist entries may exist under any transfer restriction, and they are read only under "allowlist only". Each list message works as one batch. If one entry fails, nothing changes. The number of entries in one batch and the size of each entry's data are limited by [parameters](#token-parameters).

<a id="token-msg-upsert-blocklist"></a>

#### MsgUpsertBlocklist

Adds accounts to the blocklist, or replaces the data of existing entries. Managing the blocklist follows the freeze setting. While it is off, you cannot add an entry or change one, but sending identical data again is a harmless no-op.

```sh
urid tx token upsert-blocklist token/uri1.../usdc \
  --entries '{"account":"ACCOUNT_ADDRESS","data":"Y2FzZS0x"}' \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `sender`, the issuer or a FREEZE holder |
| Before you send it | The freeze setting must be on to add or change an entry. |
| What comes back | How many entries were requested, and how many were created or changed. |
| urid | `urid tx token upsert-blocklist [denom] --entries <json>`, repeatable, one `{account, data}` per flag |

```proto
message MsgUpsertBlocklist {
  string sender = 1;
  string denom = 2;
  repeated BlocklistEntry entries = 3;
}

message MsgUpsertBlocklistResponse {
  uint32 requested_count = 1;
  uint32 changed_count = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a freeze holder. Must sign. |
| denom | string | The token. |
| entries | list of [`BlocklistEntry`](#token-type-list-entries) | The accounts and their data. At least one, with no repeated account. |
| requested_count | number | In the response. Entries in the request. |
| changed_count | number | In the response. Entries created or changed. |

**Refused when**

- The sender is neither the issuer nor a freeze holder.
- The freeze setting is off and an entry would be created or changed.
- An entry targets the issuer or a module account.
- The batch is empty, repeats an account, or has too many entries.
- An entry's data is too large.

<a id="token-msg-delete-blocklist"></a>

#### MsgDeleteBlocklist

Removes accounts from the blocklist. This works whatever the freeze setting is. Removing an account that is not on the list is not an error.

| | |
|---|---|
| Signer | `sender`, the issuer or a FREEZE holder |
| Before you send it | Nothing. |
| What comes back | How many accounts were requested, and how many entries were deleted. |
| urid | `urid tx token delete-blocklist [denom] [account]...` |

```proto
message MsgDeleteBlocklist {
  string sender = 1;
  string denom = 2;
  repeated string accounts = 3;
}

message MsgDeleteBlocklistResponse {
  uint32 requested_count = 1;
  uint32 deleted_count = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a freeze holder. Must sign. |
| denom | string | The token. |
| accounts | list of addresses | The accounts to remove. At least one, with no repeat. |
| requested_count | number | In the response. Accounts in the request. |
| deleted_count | number | In the response. Entries that existed and were deleted. |

**Refused when**

- The sender is neither the issuer nor a freeze holder.
- The list is empty, repeats an account, or has too many accounts.

<a id="token-msg-upsert-allowlist"></a>

#### MsgUpsertAllowlist

Adds accounts to the allowlist, or replaces the data of existing entries. The allowlist has no on and off setting, so it can be managed at any time. The issuer is not on the allowlist by default. Under "allowlist only" it needs its own entry to receive the token.

| | |
|---|---|
| Signer | `sender`, the issuer or a MANAGE_ALLOWLIST holder |
| Before you send it | Nothing. |
| What comes back | How many entries were requested, and how many were created or changed. |
| urid | `urid tx token upsert-allowlist [denom] --entries <json>`, repeatable, one `{account, data}` per flag |

```proto
message MsgUpsertAllowlist {
  string sender = 1;
  string denom = 2;
  repeated AllowlistEntry entries = 3;
}

message MsgUpsertAllowlistResponse {
  uint32 requested_count = 1;
  uint32 changed_count = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or an allowlist holder. Must sign. |
| denom | string | The token. |
| entries | list of [`AllowlistEntry`](#token-type-list-entries) | The accounts and their data. At least one, with no repeated account. |
| requested_count | number | In the response. Entries in the request. |
| changed_count | number | In the response. Entries created or changed. |

**Refused when**

- The sender is neither the issuer nor an allowlist holder.
- The batch is empty, repeats an account, or has too many entries.
- An entry's data is too large.

<a id="token-msg-delete-allowlist"></a>

#### MsgDeleteAllowlist

Removes accounts from the allowlist. Removing an account that is not on the list is not an error. Changing the transfer restriction never clears the allowlist, so use this message to tidy it up.

| | |
|---|---|
| Signer | `sender`, the issuer or a MANAGE_ALLOWLIST holder |
| Before you send it | Nothing. |
| What comes back | How many accounts were requested, and how many entries were deleted. |
| urid | `urid tx token delete-allowlist [denom] [account]...` |

```proto
message MsgDeleteAllowlist {
  string sender = 1;
  string denom = 2;
  repeated string accounts = 3;
}

message MsgDeleteAllowlistResponse {
  uint32 requested_count = 1;
  uint32 deleted_count = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or an allowlist holder. Must sign. |
| denom | string | The token. |
| accounts | list of addresses | The accounts to remove. At least one, with no repeat. |
| requested_count | number | In the response. Accounts in the request. |
| deleted_count | number | In the response. Entries that existed and were deleted. |

**Refused when**

- The sender is neither the issuer nor an allowlist holder.
- The list is empty, repeats an account, or has too many accounts.

<a id="token-tx-moving"></a>

### Moving tokens without the holder's own signature

<a id="token-msg-approved-transfer"></a>

#### MsgApprovedTransfer

Moves a token whose transfer restriction is "approval required". Two parties must agree. The asset authorizer is the source itself or an agent acting for it under a delegation. The approver is the issuer or an APPROVE_TRANSFER holder. The approver alone cannot move anyone's coins, and the asset authorizer alone cannot move the token. The recipient need not be on the allowlist, but every other movement rule applies.

When the agent works for Alice, it signs as the asset authorizer and names Alice as the source. The message must then point at the delegation with `delegation`, and the delegation module checks and uses up that permission in the same step. See [Delegations](#ch-delegation). When Alice signs for herself, `delegation` must be left out. One account may hold both roles, and then it signs once.

Both are signers. `urid` takes the asset authorizer from `--from` and the approver as an argument. Generate the transaction, have each signer sign it with `urid tx sign`, then broadcast it. By default the asset authorizer pays the fee. The approver or a sponsor can pay instead if the transaction names them as fee payer. The fee payer signs with DIRECT, and the other signer may sign with DIRECT_AUX.

```sh
urid tx token approved-transfer APPROVER_ADDRESS SOURCE_ADDRESS RECIPIENT_ADDRESS token/uri1.../sec 250 \
  --delegation '{"delegation_id":"7","permission_index":0}' \
  --from AGENT_KEY --generate-only
```

| | |
|---|---|
| Signer | `asset_authorizer`, the source or its agent, and `approver`, the issuer or an APPROVE_TRANSFER holder |
| Before you send it | The token must be under "approval required". An agent needs a live delegation from the source that covers this send. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx token approved-transfer [approver] [source] [recipient] [denom] [amount]` with `--delegation` when the asset authorizer is not the source |

```proto
message MsgApprovedTransfer {
  string asset_authorizer = 1;
  string approver = 2;
  string source = 3;
  string recipient = 4;
  string denom = 5;
  string amount = 6;
  urinet.delegation.v1.DelegationRef delegation = 7;
}

message MsgApprovedTransferResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| asset_authorizer | address | The source, or its agent. First signer. |
| approver | address | The issuer or an approval holder. Second signer. |
| source | address | Whose coins leave. Not a module account. |
| recipient | address | Who receives them. Not a module account, and not the source. |
| denom | string | The token. |
| amount | integer as a string | How many base units to move. Must be above zero. |
| delegation | [`DelegationRef`](#ch-delegation) | The delegation and the permission in it that the agent uses. Present exactly when the asset authorizer is not the source. |

**Refused when**

- The token is not under "approval required".
- The source and the recipient are the same account.
- The token is paused.
- `delegation` is missing although an agent signs, or present although the source signs. The delegation module refuses a delegation that does not cover the send.
- The approver is neither the issuer nor an approval holder.
- The source or the recipient is on the blocklist or is a module account.
- The amount exceeds the source's usable balance, or the recipient would go over the holding limit.

<a id="token-msg-forced-transfer"></a>

#### MsgForcedTransfer

Moves coins from a holder to another account without the holder's signature. The issuer side uses it to recover coins, for example after a court order or when coins went to the wrong place. It works during the issuer pause. It skips the source's blocklist entry and freeze, the transfer restriction and the recipient's holding limit. It still obeys the governance pause, the recipient's blocklist entry, vesting, and the maximum holder count at the end of the transaction. The source can never be the issuer, but the recipient may be.

You choose which coins leave. With no `source_selection`, the chain first takes the source's unlocked coins, then takes locked future releases, earliest unlock time first. With `source_selection`, you name the releases and amounts yourself. The rest of the amount must then fit in the source's unlocked coins.

You also choose how the coins arrive, with `destination_vesting`. Leave it out to preserve the original terms. Coins that were unlocked arrive unlocked, and coins taken from releases keep their unlock times, as one new schedule per source schedule. Send `{}` to make everything usable at once. Send `{"schedules":[...]}` to attach new schedules, and any amount they leave over arrives unlocked. See [Vesting in a forced transfer](#vesting-forced-transfer).

```sh
urid tx token forced-transfer token/uri1.../usdc SOURCE_ADDRESS ISSUER_ADDRESS 1000 \
  --destination-vesting '{}' \
  --from KEY --generate-only
```

| | |
|---|---|
| Signer | `sender`, the issuer or a FORCED_TRANSFER holder |
| Before you send it | The forced transfer setting must be on. If the holder cap is full, recover into an account that already holds the token. |
| What comes back | How much came from the source's unlocked coins, the releases that were reduced, and the schedules created at the recipient. |
| urid | `urid tx token forced-transfer [denom] [source] [recipient] [amount]` with `--source-selection`, repeatable, and `--destination-vesting` |

```proto
message MsgForcedTransfer {
  string sender = 1;
  string denom = 2;
  string source = 3;
  string recipient = 4;
  string amount = 5;
  repeated urinet.vesting.v1.VestingReleaseSelection source_selection = 6;
  urinet.vesting.v1.DestinationVesting destination_vesting = 7;
}

message MsgForcedTransferResponse {
  string source_free_amount_used = 1;
  repeated urinet.vesting.v1.ReducedVestingRelease reduced_source_releases = 2;
  repeated urinet.vesting.v1.PreservedScheduleMapping preserved_schedule_mappings = 3;
  repeated uint64 created_destination_schedule_ids = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| sender | address | The issuer or a forced transfer holder. Must sign. May be the source. |
| denom | string | The token. |
| source | address | Whose coins are taken. Not the issuer and not a module account. |
| recipient | address | Who receives them. Not the source and not a module account. |
| amount | integer as a string | The total to move, in base units. Must be above zero. |
| source_selection | list of [`VestingReleaseSelection`](#vesting-forced-transfer) | Locked releases to take from, and how much from each. Empty means the chain chooses. |
| destination_vesting | [`DestinationVesting`](#vesting-forced-transfer) | How the coins arrive. Absent preserves the original terms, empty makes them usable, schedules attach new terms. |
| source_free_amount_used | integer as a string | In the response. The part taken from the source's unlocked coins. |
| reduced_source_releases | list of [`ReducedVestingRelease`](#vesting-forced-transfer) | In the response. Each source release reduced, and by how much. |
| preserved_schedule_mappings | list of [`PreservedScheduleMapping`](#vesting-forced-transfer) | In the response. When terms are preserved, which source schedule became which recipient schedule. |
| created_destination_schedule_ids | list of numbers | In the response. The ids of new schedules you specified. |

**Refused when**

- The forced transfer setting is off.
- The sender is neither the issuer nor a forced transfer holder.
- The source is the issuer, or the source and the recipient are the same.
- The source or the recipient is a module account, or the recipient is on the blocklist.
- The source does not hold enough, or a selected release does not exist, is already unlocked, or is smaller than the amount taken.
- The new schedules are invalid, total more than the amount, or would push the recipient past its vesting limits.

<a id="token-tx-holder"></a>

### The holder's own action

<a id="token-msg-reject-vesting-schedules"></a>

#### MsgRejectVestingSchedules

Lets a holder refuse vesting it never asked for. A token's creation and a forced transfer can attach vesting schedules to an account without its consent. With this message the holder removes whole schedules of one user token from its own account. The coins still locked in them are burned and go to no one. Only those locked coins are touched. The holder's unlocked balance, other tokens, and vesting of the native coin stay as they are.

No permission is needed, and the issuer cannot stop it. The burn setting, the issuer pause, freezes and the blocklist do not apply. The issuer cannot recover burned coins. An issuer that has a minter record with allowance left could mint the same amount again. For example, Bob received shares that unlock over four years and does not want them. He rejects the schedule, and the locked shares are burned.

| | |
|---|---|
| Signer | `holder`, the account that holds the schedules |
| Before you send it | Read your schedule ids with [SchedulesByAccount](#vesting-query-schedules-by-account). |
| What comes back | The ids removed, in ascending order, and the amount burned. |
| urid | `urid tx token reject-vesting-schedules [denom] [schedule-id]...`. With no id, every schedule of that denom is removed. |

```proto
message MsgRejectVestingSchedules {
  string holder = 1;
  string denom = 2;
  repeated uint64 schedule_ids = 3;
}

message MsgRejectVestingSchedulesResponse {
  repeated uint64 removed_schedule_ids = 1;
  string burned_amount = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| holder | address | The holder. Must sign. |
| denom | string | A user token, `token/<issuer>/<subdenom>`. The native coin is refused. |
| schedule_ids | list of numbers | The schedules to remove. Empty means every schedule of this denom. |
| removed_schedule_ids | list of numbers | In the response. The schedules actually removed, ascending. |
| burned_amount | integer as a string | In the response. The locked amount burned. |

**Refused when**

- The denom is not a user token, or the token does not exist.
- An id is zero, repeated, unknown, or belongs to another denom.
- The list is empty and the holder has no schedule of this denom.
- The holder has no account on the chain, or is a module account.

<a id="token-tx-governance"></a>

### Governance

<a id="token-msg-set-governance-hard-pause"></a>

#### MsgSetGovernanceHardPause

Turns the governance pause of one token on or off. While it is on, every movement and every change to the token stops, forced transfers included. See [Two kinds of pause](#token-pauses). This is the only power governance has over a single token. It cannot appoint admins, change the supply or move anyone's coins. One message covers one token, so a proposal that pauses several tokens carries several messages. Asking for the state that is already set succeeds and changes nothing.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | The token must exist. |
| What comes back | Whether the governance pause changed. |
| urid | `urid tx token set-governance-hard-pause-proposal [denom] [paused]` with `--deposit`, `--title`, `--summary` and `--metadata` |

```proto
message MsgSetGovernanceHardPause {
  string authority = 1;
  string denom = 2;
  bool paused = 3;
}

message MsgSetGovernanceHardPauseResponse {
  bool changed = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| denom | string | The token's full denom, with the `token/` prefix. |
| paused | boolean | `true` to pause, `false` to lift the pause. |
| changed | boolean | In the response. Whether the state actually changed. |

**Refused when**

- The authority is not the governance account.
- The denom is empty, invalid, or lacks the `token/` prefix. This is caught when the proposal is submitted.
- No token has this denom.

<a id="token-msg-update-params"></a>

#### MsgUpdateParams

Replaces the token module's parameters, such as the length limits, the batch sizes, the reserved symbols and the storage gas. See [Parameters](#token-parameters). It replaces all of them at once. A field you leave out becomes zero, not its current value. So read the current values with `urid query token params --output json`, change what you need, and send the whole set. New values apply only to later transactions. Existing tokens and entries stay valid.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current parameters and build the full set. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx token update-params-proposal [params]`, where `params` is the whole set as JSON, with `--deposit`, `--title`, `--summary` and `--metadata` |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | `Params`, see [Parameters](#token-parameters) | The complete new parameter set. Required. |

**Refused when**

- The authority is not the governance account.
- The parameters are missing or fail validation.

---

<a id="token-queries"></a>

## Queries

The token module answers 14 queries about tokens, their permission holders, minters, freezes, lists, and usable balances.

| Query | What it answers |
|---|---|
| [Token](#token-query-token) | One token's record, coin metadata, decimals, supply, and holder count. |
| [TokensByIssuer](#token-query-tokens-by-issuer) | The denoms one issuer created. |
| [Permissions](#token-query-permissions) | One account's permissions on a token, and whether it is the issuer. |
| [PermissionHolders](#token-query-permission-holders) | Every account that holds permissions on a token. |
| [Minter](#token-query-minter) | Whether one account is a minter, and how much it may still mint. |
| [Minters](#token-query-minters) | Every minter of a token, with the supply figures. |
| [FrozenAmount](#token-query-frozen-amount) | How much of one account's balance is frozen. |
| [FrozenAccounts](#token-query-frozen-accounts) | Every account with a freeze on a token. |
| [BlocklistEntry](#token-query-blocklist-entry) | Whether one account is on a token's blocklist. |
| [Blocklist](#token-query-blocklist) | The whole blocklist of a token. |
| [AllowlistEntry](#token-query-allowlist-entry) | Whether one account is on a token's allowlist. |
| [Allowlist](#token-query-allowlist) | The whole allowlist of a token. |
| [UsableBalance](#token-query-usable-balance) | An account's balance, locked amount, frozen amount, and usable amount. |
| [Params](#token-query-params) | The module's current parameters. |

Queries change nothing on the chain. They all work while a token is paused, by its issuer or by governance.

Every REST path starts with `/urinet/token/v1/`. A path that takes a denom ends with it, and the denom goes into the path as it is, slashes included. For example, the usable balance of Alice's holding of a token is read from `https://<node>:1317/urinet/token/v1/accounts/<account>/usable_balance/token/<issuer>/<subdenom>`.

A malformed denom or address is refused with `InvalidArgument`. A well-formed denom of a token that does not exist answers `NotFound`. A missing per-account record is a normal answer, not an error: no permissions, not a minter, nothing frozen, not listed.

<a id="token-queries-one-token"></a>

### One token

<a id="token-query-token"></a>

#### Token

Reads the current state of one token in a single answer. Wallets and explorers use it to show a token's name, settings, supply, and holder count.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Token` |
| REST | `GET /urinet/token/v1/tokens/{denom}` |
| urid | `urid query token token [denom]` |

```proto
message QueryTokenRequest {
  string denom = 1;
}

message QueryTokenResponse {
  Token token = 1;
  cosmos.bank.v1beta1.Metadata metadata = 2;
  uint32 decimals = 3;
  string current_total_supply = 4;
  uint64 holder_count = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token, as `token/<issuer>/<subdenom>`. |
| token | [`Token`](#token-type-token) | In the response. The token record: denom, issuer, the nine settings, the two pause flags, and the creation time. |
| metadata | bank `Metadata` | In the response. The token's coin metadata in the standard [bank module](#std-bank): name, symbol, description, denom units, and document reference. |
| decimals | number | In the response. Taken from the exponent of the display unit in the metadata. |
| current_total_supply | integer as a string | In the response. The total supply now, in base units. 0 when nothing has been minted. |
| holder_count | number | In the response. How many accounts hold a balance above 0. Module accounts are not counted. |

The answer does not say whether a given account can send or mint right now. Use [UsableBalance](#token-query-usable-balance) and [Minter](#token-query-minter) for the amounts, and remember that other rules still apply to a real transaction.

<a id="token-query-tokens-by-issuer"></a>

#### TokensByIssuer

Lists the denoms of every token one issuer created. The issuer's wallet uses it to show "my tokens".

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/TokensByIssuer` |
| REST | `GET /urinet/token/v1/issuers/{issuer}/tokens` |
| urid | `urid query token tokens-by-issuer [issuer]` |

```proto
message QueryTokensByIssuerRequest {
  string issuer = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryTokensByIssuerResponse {
  repeated string denoms = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| issuer | address | The issuer. Any valid `uri1` address is accepted. |
| pagination | `PageRequest` | Which page to read. See below. |
| denoms | list of strings | In the response. The issuer's denoms, in byte order of the denom. Empty if the issuer created none. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

This is the first of the six list queries. Each takes `pagination` with `key` and `limit`. The limit is 100 when you leave it out or send 0, and it is capped at 200. `reverse` is allowed. `offset` and `count_total` are refused with `InvalidArgument`, so `total` in the response is always 0. To read on, send the `next_key` of the last answer as the next `key`. Over REST, pass them as query parameters such as `pagination.key` and `pagination.limit`. With `urid`, use `--page-key`, `--page-limit`, and `--page-reverse`. More in [Reading lists](#connect-conventions).

<a id="token-queries-permissions"></a>

### Permissions and minters

<a id="token-query-permissions"></a>

#### Permissions

Reads the permissions one account holds on a token, and whether that account is the issuer. The issuer holds every power by being the issuer, so it never has a permission list of its own. `is_issuer` tells the issuer apart from an account with no permissions.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Permissions` |
| REST | `GET /urinet/token/v1/accounts/{account}/permissions/{denom}` |
| urid | `urid query token permissions [account] [denom]` |

```proto
message QueryPermissionsRequest {
  string denom = 1;
  string account = 2;
}

message QueryPermissionsResponse {
  bool is_issuer = 1;
  repeated TokenPermission permissions = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The account to look up. |
| is_issuer | boolean | In the response. True if the account is the token's issuer. |
| permissions | list of [`TokenPermission`](#token-type-permissions) | In the response. The account's permissions, in ascending order: `TOKEN_PERMISSION_ADMIN`, `TOKEN_PERMISSION_MANAGE_MINTERS`, `TOKEN_PERMISSION_BURN`, `TOKEN_PERMISSION_PAUSE`, `TOKEN_PERMISSION_FREEZE`, `TOKEN_PERMISSION_FORCED_TRANSFER`, `TOKEN_PERMISSION_UPDATE_METADATA`, `TOKEN_PERMISSION_MANAGE_ALLOWLIST`, `TOKEN_PERMISSION_APPROVE_TRANSFER`. Always empty for the issuer. |

<a id="token-query-permission-holders"></a>

#### PermissionHolders

Lists every account that holds permissions on a token, with its permissions. The issuer or an admin uses it to check who can freeze, force transfers, or manage minters, and to find whom to revoke.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/PermissionHolders` |
| REST | `GET /urinet/token/v1/permission_holders/{denom}` |
| urid | `urid query token permission-holders [denom]` |

```proto
message QueryPermissionHoldersRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryPermissionHoldersResponse {
  repeated AccountPermissions holders = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| pagination | `PageRequest` | Paged, see [Reading lists](#connect-conventions). |
| holders | list of [`AccountPermissions`](#token-type-permissions) | In the response. Each entry is an `account` and its `permissions`, the same shape the grant and revoke messages use. The issuer is not listed. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

<a id="token-query-minter"></a>

#### Minter

Reads whether one account is a minter of a token and how much more it could mint. Being a minter means having a minter record. A minter whose allowance is 0 is still a minter.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Minter` |
| REST | `GET /urinet/token/v1/accounts/{account}/minter/{denom}` |
| urid | `urid query token minter [account] [denom]` |

```proto
message QueryMinterRequest {
  string denom = 1;
  string account = 2;
}

message QueryMinterResponse {
  bool is_minter = 1;
  string remaining_mint_allowance = 2;
  string supply_room = 3;
  string effective_mint_capacity = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The account to look up. |
| is_minter | boolean | In the response. True if the account has a minter record. |
| remaining_mint_allowance | integer as a string | In the response. What the minter may still mint. 0 for an account that is not a minter. |
| supply_room | integer as a string | In the response. The token's maximum supply minus its current total supply. The same for every account. |
| effective_mint_capacity | integer as a string | In the response. The smaller of `remaining_mint_allowance` and `supply_room`. 0 for an account that is not a minter. |

`effective_mint_capacity` counts quantities only. It ignores pauses, the mint setting, and the recipient's lists, holding limit, and holder cap. A positive value does not promise that a mint will go through.

<a id="token-query-minters"></a>

#### Minters

Lists every minter of a token with its allowance. The supply figures that apply to all of them come once, at the top of the answer.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Minters` |
| REST | `GET /urinet/token/v1/minters/{denom}` |
| urid | `urid query token minters [denom]` |

```proto
message QueryMintersRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message MinterInfo {
  string account = 1;
  string remaining_mint_allowance = 2;
  string effective_mint_capacity = 3;
}

message QueryMintersResponse {
  string current_total_supply = 1;
  string max_supply = 2;
  string supply_room = 3;
  repeated MinterInfo minters = 4;
  cosmos.base.query.v1beta1.PageResponse pagination = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| pagination | `PageRequest` | Paged, see [Reading lists](#connect-conventions). |
| current_total_supply | integer as a string | In the response. The total supply now. |
| max_supply | integer as a string | In the response. The token's maximum supply setting. |
| supply_room | integer as a string | In the response. `max_supply` minus `current_total_supply`. |
| minters | list of `MinterInfo` | In the response. One entry per minter. Being listed means the account is a minter. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

`MinterInfo` has these fields.

| Field | Type | Meaning |
|---|---|---|
| account | address | The minter. |
| remaining_mint_allowance | integer as a string | What the minter may still mint. |
| effective_mint_capacity | integer as a string | The smaller of the allowance and `supply_room`. Quantities only, as in [Minter](#token-query-minter). |

<a id="token-queries-freeze-lists"></a>

### Freezes and lists

<a id="token-query-frozen-amount"></a>

#### FrozenAmount

Reads how much of one account's balance the issuer side has frozen. An account with no freeze answers 0. The frozen amount may be larger than the balance. The excess also holds back coins that arrive later.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/FrozenAmount` |
| REST | `GET /urinet/token/v1/accounts/{account}/frozen_amount/{denom}` |
| urid | `urid query token frozen-amount [account] [denom]` |

```proto
message QueryFrozenAmountRequest {
  string denom = 1;
  string account = 2;
}

message QueryFrozenAmountResponse {
  string frozen_amount = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The account to look up. |
| frozen_amount | integer as a string | In the response. The frozen amount, in base units. 0 if there is no freeze. |

<a id="token-query-frozen-accounts"></a>

#### FrozenAccounts

Lists every account that has a freeze on a token, with its frozen amount. It shows freezes only. For an account's balance and what it can still use, ask [UsableBalance](#token-query-usable-balance).

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/FrozenAccounts` |
| REST | `GET /urinet/token/v1/frozen_accounts/{denom}` |
| urid | `urid query token frozen-accounts [denom]` |

```proto
message QueryFrozenAccountsRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message FrozenAccount {
  string account = 1;
  string frozen_amount = 2;
}

message QueryFrozenAccountsResponse {
  repeated FrozenAccount accounts = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| pagination | `PageRequest` | Paged, see [Reading lists](#connect-conventions). |
| accounts | list of `FrozenAccount` | In the response. One entry per frozen account: `account` and its `frozen_amount`, always above 0. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

<a id="token-query-blocklist-entry"></a>

#### BlocklistEntry

Answers "is Alice on this token's blocklist?" and returns the data the issuer stored with her entry. Entries stay in force even after the token's freeze setting is turned off, so they are always returned.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/BlocklistEntry` |
| REST | `GET /urinet/token/v1/accounts/{account}/blocklist_entry/{denom}` |
| urid | `urid query token blocklist-entry [account] [denom]` |

```proto
message QueryBlocklistEntryRequest {
  string denom = 1;
  string account = 2;
}

message QueryBlocklistEntryResponse {
  bool listed = 1;
  bytes data = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The account to look up. |
| listed | boolean | In the response. True if the account has an entry. Read this, not `data`, because an entry with empty data is a normal entry. |
| data | bytes, base64 in JSON | In the response. The issuer's own note for the entry, such as a case number or a reason code. Empty if there is none. |

<a id="token-query-blocklist"></a>

#### Blocklist

Lists the whole blocklist of a token, with the data of each entry.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Blocklist` |
| REST | `GET /urinet/token/v1/blocklist/{denom}` |
| urid | `urid query token blocklist [denom]` |

```proto
message QueryBlocklistRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message BlocklistRecord {
  string account = 1;
  bytes data = 2;
}

message QueryBlocklistResponse {
  repeated BlocklistRecord entries = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| pagination | `PageRequest` | Paged, see [Reading lists](#connect-conventions). |
| entries | list of `BlocklistRecord` | In the response. Each entry is the blocked `account` and its `data`, bytes, base64 in JSON. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

<a id="token-query-allowlist-entry"></a>

#### AllowlistEntry

Answers "is the shop on this token's allowlist?" and returns the data the issuer stored with the entry. For a token whose transfer restriction is allowlist only, a recipient needs an entry. Entries can also exist under the other restrictions and are returned all the same. The issuer can have an entry like any other account.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/AllowlistEntry` |
| REST | `GET /urinet/token/v1/accounts/{account}/allowlist_entry/{denom}` |
| urid | `urid query token allowlist-entry [account] [denom]` |

```proto
message QueryAllowlistEntryRequest {
  string denom = 1;
  string account = 2;
}

message QueryAllowlistEntryResponse {
  bool listed = 1;
  bytes data = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The account to look up. |
| listed | boolean | In the response. True if the account has an entry, even one with empty data. |
| data | bytes, base64 in JSON | In the response. The issuer's own note for the entry. Empty if there is none. |

<a id="token-query-allowlist"></a>

#### Allowlist

Lists the whole allowlist of a token, with the data of each entry.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Allowlist` |
| REST | `GET /urinet/token/v1/allowlist/{denom}` |
| urid | `urid query token allowlist [denom]` |

```proto
message QueryAllowlistRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message AllowlistRecord {
  string account = 1;
  bytes data = 2;
}

message QueryAllowlistResponse {
  repeated AllowlistRecord entries = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| pagination | `PageRequest` | Paged, see [Reading lists](#connect-conventions). |
| entries | list of `AllowlistRecord` | In the response. Each entry is the allowlisted `account` and its `data`, bytes, base64 in JSON. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. |

<a id="token-queries-balances"></a>

### Balances

<a id="token-query-usable-balance"></a>

#### UsableBalance

Shows how much of a token one account can use in the ordinary way right now, and why. A wallet calls it before it offers a send.

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/UsableBalance` |
| REST | `GET /urinet/token/v1/accounts/{account}/usable_balance/{denom}` |
| urid | `urid query token usable-balance [account] [denom]` |

```proto
message QueryUsableBalanceRequest {
  string denom = 1;
  string account = 2;
}

message QueryUsableBalanceResponse {
  string balance = 1;
  string vesting_locked = 2;
  string frozen_amount = 3;
  string token_usable = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The token. |
| account | address | The holder. An address that has a balance but no account record yet is answered normally. |
| balance | integer as a string | In the response. The account's whole balance of the token. |
| vesting_locked | integer as a string | In the response. The part still locked by vesting schedules at the current block time. See [Vesting](#ch-vesting). |
| frozen_amount | integer as a string | In the response. The issuer-side freeze. 0 if there is none. |
| token_usable | integer as a string | In the response. `balance` minus `vesting_locked` minus `frozen_amount`, or 0 if that is negative. |

Say Alice holds 1,000 base units of the shop's token. 600 are still locked and the issuer froze 100, so `token_usable` is 300. The figure counts quantities only. Pauses, the blocklist, the transfer restriction, the recipient's limits, and authorization still decide whether a real transfer goes through.

<a id="token-queries-params"></a>

### Module parameters

<a id="token-query-params"></a>

#### Params

Reads the module's whole parameter set. A proposal to change parameters replaces all of them at once, so read the current set first and change only what you mean to. See [Parameters](#token-parameters).

| | |
|---|---|
| gRPC | `urinet.token.v1.Query/Params` |
| REST | `GET /urinet/token/v1/params` |
| urid | `urid query token params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | `Params` | In the response. The current parameters, as described in [Parameters](#token-parameters). |

<a id="token-parameters"></a>

## Parameters

Parameters are chain-wide limits and gas costs of the token module, not settings of one token. Governance changes them with the module's [MsgUpdateParams](#token-msg-update-params) inside a proposal. It replaces the whole set, so a parameter left out becomes zero, and new values apply only to later operations.

```proto
message Params {
  uint32 subdenom_max_bytes = 1;
  uint32 name_max_bytes = 2;
  uint32 symbol_max_bytes = 3;
  uint32 description_max_bytes = 4;
  uint32 metadata_uri_max_bytes = 5;
  uint32 blocklist_data_max_bytes = 6;
  uint32 allowlist_data_max_bytes = 7;
  uint32 list_batch_max_items = 8;
  uint32 permission_batch_max_accounts = 9;
  repeated string reserved_symbols = 10;
  uint64 storage_entry_create_gas = 11;
  uint64 storage_growth_per_byte_gas = 12;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| subdenom_max_bytes | Longest subdenom, the part you choose in `token/<issuer>/<subdenom>`, in bytes. | 32 |
| name_max_bytes | Longest token name, in UTF-8 bytes. | 128 |
| symbol_max_bytes | Longest token symbol, in ASCII characters. Never below 3. | 16 |
| description_max_bytes | Longest description in the coin metadata, in bytes. | 2,048 |
| metadata_uri_max_bytes | Longest document URI in the coin metadata, in bytes. | 2,048 |
| blocklist_data_max_bytes | Most data the issuer can attach to one blocklist entry, in bytes. It does not limit the number of entries. | 512 |
| allowlist_data_max_bytes | Most data the issuer can attach to one allowlist entry, in bytes. It does not limit the number of entries. | 512 |
| list_batch_max_items | Most accounts one blocklist or allowlist message can handle. | 200 |
| permission_batch_max_accounts | Most accounts one grant or revoke message can handle. | 100 |
| reserved_symbols | Extra symbols, in lower case, that new tokens may not use as symbol or name. The name, symbol, and unit names of URI are always reserved and need no entry. | empty |
| storage_entry_create_gas | Extra gas for each new record the module stores. | 20,000 |
| storage_growth_per_byte_gas | Extra gas for each byte a stored record grows by. Shrinking or deleting costs nothing and refunds nothing. | 100 |

The values are those the network started with. Read the current ones with the [Params](#token-query-params) query.

---

<a id="ch-vesting"></a>

# Vesting: x/vesting

<a id="vesting-overview"></a>

## What it is

Vesting keeps coins in an account locked until set times. The coins are already yours, but you cannot use them yet. A vesting schedule is the plan for one denom: a list of releases, each an unlock time and an amount. When the block time reaches a release's unlock time, that amount unlocks. For example, the issuer of the shop's token gives Bob 1,000 units at creation: 200 free, and 800 in a schedule of four releases of 200, one every three months.

There are no user transactions in this module. Schedules on URI exist only where the network's starting state placed them. A schedule on a user token comes about in one of two ways. A token's initial allocation can carry schedules, see [MsgCreateToken](#token-msg-create-token). A forced transfer can place the coins it moves under a schedule at the recipient, see [MsgForcedTransfer](#token-msg-forced-transfer). In both cases a schedule only locks coins that arrive in that same transaction, so nobody can lock coins you already hold.

Schedules shrink as releases unlock. An unlocked release is removed at the end of the block in which it unlocks, and a schedule with no releases left disappears. Each schedule has a schedule ID, a number counted from 1 within your account and never reused. If you do not want a schedule on a user token, you can remove whole schedules with the token module's [MsgRejectVestingSchedules](#token-msg-reject-vesting-schedules). The coins still locked in them are burned.

Locked coins cannot be sent and cannot pay fees. Locked URI can still be staked. The amount you can use is your balance minus the locked amount, which the queries call `bank_usable`. For a user token, the token module also subtracts any frozen amount, and the result, never below 0, is `token_usable`. Ask [UsableBalance](#token-query-usable-balance) for that figure. The locked amount can never exceed the balance. If it ever did, the chain would report an error rather than show 0.

Every account on UriNet can hold schedules from the moment it exists. The account record that the auth queries return is UriNet's own vesting-aware type, [UriVestingAccount](#vesting-account). See [Accounts](#std-auth).

<a id="vesting-schedules"></a>

### Schedules and releases

A stored schedule covers exactly one denom and lists its releases in order of unlock time.

```proto
message StoredVestingSchedule {
  uint64 schedule_id = 1;
  string denom = 2;
  repeated StoredVestingRelease releases = 3;
}

message StoredVestingRelease {
  int64 unlock_time = 1;
  string amount = 2;
  string delegated_vesting_credit = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| schedule_id | number | The schedule's ID within the account. Starts at 1. A removed ID is never given out again. Other accounts can use the same number. |
| denom | string | The one denom the schedule covers, URI's `kwak` or a user token. |
| releases | list of `StoredVestingRelease` | The releases, unlock times strictly increasing. |
| unlock_time | number | When the release unlocks, in Unix seconds. It counts as unlocked once the block time equals or passes it. |
| amount | integer as a string | How many base units the release unlocks. |
| delegated_vesting_credit | integer as a string | Bookkeeping for staked URI, not coins. The part of this release counted as covered by your stake. Always 0 for a user token and for a new release. |

The locked amount of one denom is the sum of `amount` minus `delegated_vesting_credit` over the releases that have not unlocked yet.

<a id="vesting-schedule-input"></a>

### Schedule input

A message that attaches schedules, such as a token's initial allocation or a forced transfer, describes them with these two types. You give only the denom, the unlock times, and the amounts. The chain assigns the schedule ID and starts the staking bookkeeping at 0.

```proto
message VestingScheduleSpec {
  string denom = 1;
  repeated VestingReleaseSpec releases = 2;
}

message VestingReleaseSpec {
  int64 unlock_time = 1;
  string amount = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom of the schedule. It must be the denom the account receives in the same transaction. |
| releases | list of `VestingReleaseSpec` | At least one release. |
| unlock_time | number | When the release unlocks, in Unix seconds. Later than the current block time, and strictly increasing within the schedule. |
| amount | integer as a string | Above 0. |

**Refused when**

- A schedule has no releases, an amount of 0, a repeated or decreasing unlock time, or an unlock time that is not in the future (`ErrInvalidVestingSchedule`).
- A schedule's denom is not the denom received, or the schedules together lock more than the account receives (`ErrInvalidVestingAttach`).
- The account would end up with more live schedules or releases than the module's ceilings allow (`ErrVestingScheduleLimit`). See [Parameters](#vesting-parameters).

The full list of error codes is in [Errors](#appendix-errors).

<a id="vesting-forced-transfer"></a>

### Vesting in a forced transfer

A forced transfer moves a user token out of an account without the holder's signature. It ignores the holder's freeze but not the holder's vesting. Locked coins can leave only by cutting the releases that lock them. These types say which releases to cut at the source and how to lock the coins at the recipient.

```proto
message VestingReleaseSelection {
  uint64 schedule_id = 1;
  int64 unlock_time = 2;
  string amount = 3;
}

message DestinationVesting {
  repeated VestingScheduleSpec schedules = 1;
}

message ReducedVestingRelease {
  uint64 schedule_id = 1;
  int64 unlock_time = 2;
  string amount = 3;
}

message PreservedScheduleMapping {
  uint64 source_schedule_id = 1;
  uint64 destination_schedule_id = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| schedule_id | number | In `VestingReleaseSelection`. The source schedule of the release to cut. |
| unlock_time | number | In `VestingReleaseSelection`. Picks the release within that schedule, in Unix seconds. |
| amount | integer as a string | In `VestingReleaseSelection`. How much to cut from that release. Above 0 and no more than the release holds. |
| schedules | list of `VestingScheduleSpec` | In `DestinationVesting`. The new schedules to attach at the recipient. See [Schedule input](#vesting-schedule-input). |
| schedule_id | number | In `ReducedVestingRelease`. The source schedule that was actually cut. |
| unlock_time | number | In `ReducedVestingRelease`. The unlock time of the release that was cut. |
| amount | integer as a string | In `ReducedVestingRelease`. How much was cut from it. |
| source_schedule_id | number | In `PreservedScheduleMapping`. A schedule at the source. |
| destination_schedule_id | number | In `PreservedScheduleMapping`. The new schedule at the recipient that carries its conditions. |

At the source, an empty selection lets the chain choose. It uses the source's unlocked coins first, frozen ones included, because a forced transfer ignores the freeze. Only if they fall short does it cut releases, soonest unlock time first and lower schedule ID first on a tie. With a list of `VestingReleaseSelection`, you name the releases yourself. Each must exist, unlock in the future, and be of the transfer's denom, with no release named twice. The selected total cannot exceed the transfer amount, and the rest must fit in the source's unlocked coins.

At the recipient, `DestinationVesting` has three meanings:

- Left out: the conditions are kept. What was cut from locked releases is locked again with the same unlock times, one new schedule for each source schedule. What came from unlocked coins arrives unlocked.
- Present with no schedules: the whole amount is usable at once.
- Present with schedules: those schedules are attached in the order given. Their total cannot exceed the transfer amount, and the rest is usable at once.

The transfer's response reports the outcome with `ReducedVestingRelease` and `PreservedScheduleMapping`.

**Refused when**

- A selected release does not exist, has already unlocked, is of another denom, or is named twice (`ErrInvalidVestingSelection`).
- A selection asks to cut more than the release holds, or the selections leave more than the source's unlocked coins can cover (`ErrInvalidVestingSelection`).
- With automatic selection, the amount is more than the source's whole balance of the denom (insufficient funds).

<a id="vesting-schedule-info"></a>

### Schedule information

The schedule queries return each schedule with two figures worked out at the current block time.

```proto
message VestingScheduleInfo {
  uint64 schedule_id = 1;
  string denom = 2;
  repeated StoredVestingRelease releases = 3;
  string principal = 4;
  string locked_amount = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| schedule_id | number | The schedule's ID within the account. |
| denom | string | The denom of the schedule. |
| releases | list of `StoredVestingRelease` | The releases as stored, see [Schedules and releases](#vesting-schedules). |
| principal | integer as a string | The sum of the release amounts. |
| locked_amount | integer as a string | What is still locked now: `amount` minus `delegated_vesting_credit`, summed over the releases that have not unlocked. |

A release that has unlocked but has not been removed yet still appears. It counts in `principal` and adds 0 to `locked_amount`.

<a id="vesting-account"></a>

### The account record

The auth queries return every ordinary account as a `UriVestingAccount`. In JSON its `@type` is `/urinet.vesting.v1.UriVestingAccount`. Module accounts keep the standard module account type.

```proto
message UriVestingAccount {
  cosmos.auth.v1beta1.BaseAccount base_account = 1;
  repeated StoredVestingSchedule schedules = 2;
  uint64 next_schedule_id = 3;
  repeated cosmos.base.v1beta1.Coin delegated_free = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| base_account | `BaseAccount` | The address, account number, and sequence. Its `pub_key` is always empty, because signing keys live in the SmartAccount. |
| schedules | list of `StoredVestingSchedule` | The schedules the account holds now. See [Schedules and releases](#vesting-schedules). |
| next_schedule_id | number | The ID the next new schedule will get. 1 in a new account. |
| delegated_free | list of coins | Bookkeeping for staked URI that is not tied to any release. Not coins you hold. |

<a id="vesting-transactions"></a>

## Transactions

The vesting module has one message, and only governance can send it.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgUpdateParams](#vesting-msg-update-params) | Replaces the module's parameters. | Governance only |

<a id="vesting-tx-governance"></a>

### Governance

<a id="vesting-msg-update-params"></a>

#### MsgUpdateParams

Replaces the whole parameter set of the vesting module. It is how the community changes the per-account ceilings and the storage gas costs. Read the current set with `urid query vesting params --output json`, change what you need, and send the whole set. The command submits the proposal, and it is signed like any other `urid` transaction, see [Transactions with urid](#urid-tx).

A change applies only from the next schedule attach onward. An account that already holds more than a lowered ceiling stays valid, and it can take a new schedule only once it is back within the ceiling.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values with the [Params](#vesting-query-params) query. Send every field. A field you leave out becomes 0, not the current value. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx vesting update-params-proposal [params]`, where `params` is the whole set as JSON, with `--deposit`, `--title`, `--summary` and `--metadata` |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | `Params` | The whole new parameter set. See [Parameters](#vesting-parameters). |

**Refused when**

- The authority is not the governance account (unauthorized).
- `params` is missing (`ErrInvalidParams`).
- A ceiling is 0 (`ErrInvalidParams`). The two gas values may be 0.

<a id="vesting-queries"></a>

## Queries

The vesting module answers four queries about schedules, locked amounts, and its parameters. Queries change nothing on the chain and work at the current block time.

| Query | What it answers |
|---|---|
| [Schedule](#vesting-query-schedule) | One schedule of an account, with its principal and locked amount. |
| [SchedulesByAccount](#vesting-query-schedules-by-account) | All schedules of an account, page by page. |
| [VestingBalance](#vesting-query-vesting-balance) | An account's balance, locked amount, and usable amount of one denom. |
| [Params](#vesting-query-params) | The module's current parameters. |

<a id="vesting-queries-schedules"></a>

### Schedules

<a id="vesting-query-schedule"></a>

#### Schedule

Reads one schedule of one account by its schedule ID. You need the ID when a forced transfer names releases directly.

| | |
|---|---|
| gRPC | `urinet.vesting.v1.Query/Schedule` |
| REST | `GET /urinet/vesting/v1/accounts/{account}/schedules/{schedule_id}` |
| urid | `urid query vesting schedule [account] [schedule-id]` |

```proto
message QueryScheduleRequest {
  string account = 1;
  uint64 schedule_id = 2;
}

message QueryScheduleResponse {
  VestingScheduleInfo schedule = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account. |
| schedule_id | number | The schedule's ID within the account. 0 is refused with `InvalidArgument`. |
| schedule | [`VestingScheduleInfo`](#vesting-schedule-info) | In the response. The schedule with its principal and locked amount. |

If the account does not exist, is a module account, or has no schedule with that ID, the answer is `NotFound`.

<a id="vesting-query-schedules-by-account"></a>

#### SchedulesByAccount

Lists every schedule an account holds, in ascending schedule ID order, or descending with `reverse`.

| | |
|---|---|
| gRPC | `urinet.vesting.v1.Query/SchedulesByAccount` |
| REST | `GET /urinet/vesting/v1/accounts/{account}/schedules` |
| urid | `urid query vesting schedules [account]` |

```proto
message QuerySchedulesByAccountRequest {
  string account = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QuerySchedulesByAccountResponse {
  repeated VestingScheduleInfo schedules = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account. |
| pagination | `PageRequest` | Which page to read. See below. |
| schedules | list of [`VestingScheduleInfo`](#vesting-schedule-info) | In the response. The schedules on this page. Empty for an address with no account, a module account, or an account with no schedules. |
| pagination | `PageResponse` | In the response. `next_key` reads the next page. It is empty on the last page. |

The query takes `pagination` with `key` and `limit`. The limit is 100 when you leave it out or send 0, and it is capped at 200. `offset` and `count_total` are refused with `InvalidArgument`. To read on, send the `next_key` of the last answer as the next `key`. With `urid`, use `--page-key`, `--page-limit`, and `--page-reverse`. More in [Reading lists](#connect-conventions).

<a id="vesting-queries-balances"></a>

### Balances

<a id="vesting-query-vesting-balance"></a>

#### VestingBalance

Shows an account's balance of one denom, how much of it is locked now, and how much is free. It works for any valid denom: URI as `kwak`, or a user token. The denom goes into the path as it is, slashes included.

| | |
|---|---|
| gRPC | `urinet.vesting.v1.Query/VestingBalance` |
| REST | `GET /urinet/vesting/v1/accounts/{account}/vesting_balance/{denom}` |
| urid | `urid query vesting balance [account] [denom]` |

```proto
message QueryVestingBalanceRequest {
  string account = 1;
  string denom = 2;
}

message QueryVestingBalanceResponse {
  string balance = 1;
  string vesting_locked = 2;
  string bank_usable = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account. An address with a balance but no account record yet is answered normally. |
| denom | string | Any valid denom. |
| balance | integer as a string | In the response. The whole balance of the denom. |
| vesting_locked | integer as a string | In the response. The part still locked at the current block time. 0 if the account has no schedule of this denom. |
| bank_usable | integer as a string | In the response. `balance` minus `vesting_locked`. |

This answer knows nothing about a token's freezes, pauses, or lists. For a user token, [UsableBalance](#token-query-usable-balance) gives the amount you can actually use.

<a id="vesting-queries-params"></a>

### Module parameters

<a id="vesting-query-params"></a>

#### Params

Reads the module's whole parameter set. A proposal replaces all of it at once, so read this first.

| | |
|---|---|
| gRPC | `urinet.vesting.v1.Query/Params` |
| REST | `GET /urinet/vesting/v1/params` |
| urid | `urid query vesting params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | `Params` | In the response. The current parameters, as described in [Parameters](#vesting-parameters). |

<a id="vesting-parameters"></a>

## Parameters

Parameters are the vesting module's chain-wide ceilings and gas costs. Governance changes them with [MsgUpdateParams](#vesting-msg-update-params) inside a proposal, which replaces the whole set and applies only to later operations.

```proto
message Params {
  uint32 max_vesting_schedules_per_account = 1;
  uint32 max_vesting_releases_per_account = 2;
  uint64 storage_entry_create_gas = 3;
  uint64 storage_growth_per_byte_gas = 4;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| max_vesting_schedules_per_account | Most live schedules one account can hold after a new schedule is attached. Unlocked releases are cleared first, so only live schedules count. | 64 |
| max_vesting_releases_per_account | Most releases one account can hold across all its live schedules after a new schedule is attached. | 256 |
| storage_entry_create_gas | Extra gas for each new schedule attached. | 20,000 |
| storage_growth_per_byte_gas | Extra gas for each byte the account record grows by when schedules are attached. Removing or cutting schedules costs nothing and refunds nothing. | 100 |

The values are those the network started with. Read the current ones with the [Params](#vesting-query-params) query.

---

<a id="ch-reputation"></a>

# Reputation: x/reputation

<a id="reputation-overview"></a>

## What it is

The reputation module keeps one current reputation record for each account. A record is a score and an evidence hash. The scores are computed off the chain by a reputation service, which gathers the evidence and runs the scoring. The chain only stores the result, so that wallets, shops, and agents can read it.

Three roles control the module. An updater is an account allowed to write records. The committee is the single account that adds and removes updaters. Governance sets the module's parameters, including which account is the committee. The committee can be a shared account that needs several approvals, and the chain checks those approvals as it does for any account. The committee may also be an updater.

A new value replaces the current record. The chain keeps no history of earlier scores. Past values can be read back from the transactions and events that wrote them. A record is never deleted. To withdraw a score, an updater writes a score of 0. The account then has a record with score 0, which is different from having no record at all.

The chain sets no range for the score and no algorithm for the evidence hash. It does not check that a target account exists. It checks only that the address is a valid `uri1` address. The evidence hash is bytes, and a newly written hash may be at most `max_evidence_hash_bytes` long.

<a id="reputation-type-record"></a>

### The reputation record

```proto
message ReputationRecord {
  uint64 score = 1;
  bytes evidence_hash = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| score | number | The current score, a whole number from the reputation service. If the service needs fractions, it applies a scale off the chain. |
| evidence_hash | bytes, base64 in JSON | A hash made from the account's off-chain evidence. At least 1 byte. |

<a id="reputation-type-update"></a>

### The update item

A `ReputationUpdate` is one account's new record inside a [MsgSetReputations](#reputation-msg-set-reputations) batch.

```proto
message ReputationUpdate {
  string account = 1;
  uint64 score = 2;
  bytes evidence_hash = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account the record is for. Only its format is checked. |
| score | number | The new score. |
| evidence_hash | bytes, base64 in JSON | The new evidence hash, from 1 byte to `max_evidence_hash_bytes`. |

<a id="reputation-transactions"></a>

## Transactions

Four messages write records, manage the updaters, and change the parameters.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgSetReputations](#reputation-msg-set-reputations) | Writes the records of a batch of accounts. | An updater |
| [MsgAddUpdater](#reputation-msg-add-updater) | Adds an account to the updaters. | The committee |
| [MsgRemoveUpdater](#reputation-msg-remove-updater) | Removes an account from the updaters. | The committee |
| [MsgUpdateParams](#reputation-msg-update-params) | Replaces the module's parameters. | Governance only |

Each `urid tx reputation` command takes the signing account from `--from`. It can sign and send directly only for an account with one credential whose address is the key's address, and only with an explicit `--gas`. Otherwise, write the transaction with `--generate-only` and sign it with `urid tx sign`. A committee that needs several approvals always signs with `urid tx sign`. See [Transactions with urid](#urid-tx) and [How transactions work](#how-transactions).

<a id="reputation-tx-records"></a>

### Writing records

<a id="reputation-msg-set-reputations"></a>

#### MsgSetReputations

Writes new records for a batch of accounts. Each item replaces that account's record, or creates it if there is none. The batch succeeds or fails as a whole, so one bad item means no record changes.

The reputation service scores the shop at 850 and Bob at 0. An updater sends both values in one batch. Bob's 0 replaces his earlier score, and his record stays.

Writing a value equal to the current one still succeeds. Each written account gets one `urinet.reputation.v1.EventReputationSet` event. See [Reputation events](#events-reputation).

| | |
|---|---|
| Signer | `updater`, an account in the current updater list |
| Before you send it | The signer is an updater. Each account appears once in the batch. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx reputation set-reputations --updates '{...}'` |

`--updates` takes one JSON item and can be repeated. The evidence hash is base64.

```sh
urid tx reputation set-reputations \
  --updates '{"account": "uri1...", "score": "850", "evidence_hash": "s6Y/hzJ7bz4cGT0sBtbjt6ewqCpZSD1OsyLeqdGo+IY="}' \
  --from uri1... --generate-only > unsigned.json
```

```proto
message MsgSetReputations {
  string updater = 1;
  repeated ReputationUpdate updates = 2;
}

message MsgSetReputationsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| updater | address | The updater. Must sign. |
| updates | list of [`ReputationUpdate`](#reputation-type-update) | The new records, written in the order given. Must not be empty. |

**Refused when**

- The signer is not in the current updater list.
- `updates` is empty.
- The same account appears twice, even if its address is written in a different letter case.
- An account is not a valid `uri1` address.
- An evidence hash is empty or longer than `max_evidence_hash_bytes`.

<a id="reputation-tx-updaters"></a>

### Managing updaters

<a id="reputation-msg-add-updater"></a>

#### MsgAddUpdater

Adds one account to the updater list. The account can then write records. The committee may add itself.

| | |
|---|---|
| Signer | `committee`, the account named as committee in the parameters |
| Before you send it | The account is not already an updater. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx reputation add-updater [updater]` |

```proto
message MsgAddUpdater {
  string committee = 1;
  string updater = 2;
}

message MsgAddUpdaterResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| committee | address | The committee. Must sign. |
| updater | address | The account to add. |

**Refused when**

- The signer is not the committee in the current parameters.
- The updater address is not a valid `uri1` address.
- The account is already an updater.

<a id="reputation-msg-remove-updater"></a>

#### MsgRemoveUpdater

Removes one account from the updater list. The records it wrote stay as they are. Removing the last updater is allowed, and then nobody can write records until the committee adds one.

| | |
|---|---|
| Signer | `committee`, the account named as committee in the parameters |
| Before you send it | The account is an updater. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx reputation remove-updater [updater]` |

```proto
message MsgRemoveUpdater {
  string committee = 1;
  string updater = 2;
}

message MsgRemoveUpdaterResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| committee | address | The committee. Must sign. |
| updater | address | The account to remove. |

**Refused when**

- The signer is not the committee in the current parameters.
- The updater address is not a valid `uri1` address.
- The account is not an updater.

<a id="reputation-tx-governance"></a>

### Governance

<a id="reputation-msg-update-params"></a>

#### MsgUpdateParams

Replaces all the module's parameters at once. This is how the committee is changed. After the change, only the new committee can add or remove updaters. A field you leave out becomes empty or 0, not its current value, so read the current values first with [Params](#reputation-query-params).

A lower `max_evidence_hash_bytes` applies to new writes only. Longer hashes already recorded stay valid.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values and change only the ones you mean to. |
| What comes back | Nothing. The proposal result tells you it went through. |
| urid | `urid tx reputation update-params-proposal [params]`, where `[params]` is the whole parameter set as JSON, with `--title`, `--summary`, `--deposit` and `--metadata`. It submits the proposal. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#reputation-parameters) | The whole new set of parameters. |

**Refused when**

- The signer is not the governance account.
- `params` is missing.
- `committee` is not a valid `uri1` address.
- `max_evidence_hash_bytes` is 0.

<a id="reputation-queries"></a>

## Queries

These queries read one account's record, the updater list, and the parameters.

| Query | What it answers |
|---|---|
| [Reputation](#reputation-query-reputation) | The current record of one account. |
| [Updaters](#reputation-query-updaters) | The accounts that may write records. |
| [Params](#reputation-query-params) | The module's parameters. |

<a id="reputation-query-records"></a>

### Records and updaters

<a id="reputation-query-reputation"></a>

#### Reputation

Returns the current record of one account. An account with no record answers with the gRPC status `NotFound`, which means "no score". A record with score 0 is returned as a record. An empty or invalid address answers `InvalidArgument`.

| | |
|---|---|
| gRPC | `urinet.reputation.v1.Query/Reputation` |
| REST | `GET /urinet/reputation/v1/reputation/{account}` |
| urid | `urid query reputation reputation [account]` |

```proto
message QueryReputationRequest {
  string account = 1;
}

message QueryReputationResponse {
  ReputationRecord record = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| account | address | The account to look up. |
| record | [`ReputationRecord`](#reputation-type-record) | In the response. The current score and evidence hash. |

<a id="reputation-query-updaters"></a>

#### Updaters

Lists the accounts that may write records, one page at a time, in address byte order.

The list is paged. The request takes `pagination` with `key` and `limit`. A limit of 0 means 100, and a limit above 200 is cut to 200. Setting `offset` or `count_total` is refused with `InvalidArgument`. For the next page, pass the `next_key` from the response as `key`. With `urid`, use `--page-key` and `--page-limit`. See [Reading lists](#connect-conventions).

| | |
|---|---|
| gRPC | `urinet.reputation.v1.Query/Updaters` |
| REST | `GET /urinet/reputation/v1/updaters` |
| urid | `urid query reputation updaters` |

```proto
message QueryUpdatersRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryUpdatersResponse {
  repeated string updaters = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | `key` and `limit`. See above. |
| updaters | list of addresses | In the response. One page of updaters. |
| pagination | page response | In the response. Its `next_key` leads to the next page. It is empty on the last page. Do not rely on `total`. |

<a id="reputation-query-parameters"></a>

### Parameters

<a id="reputation-query-params"></a>

#### Params

Returns the module's current parameters, including the committee's address.

| | |
|---|---|
| gRPC | `urinet.reputation.v1.Query/Params` |
| REST | `GET /urinet/reputation/v1/params` |
| urid | `urid query reputation params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#reputation-parameters) | In the response. The current parameters. |

<a id="reputation-parameters"></a>

## Parameters

The parameters name the committee and cap the length of new evidence hashes. Governance changes them with [MsgUpdateParams](#reputation-msg-update-params) inside a proposal.

```proto
message Params {
  string committee = 1;
  uint32 max_evidence_hash_bytes = 2;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| committee | The address of the account that adds and removes updaters. | read it with the Params query |
| max_evidence_hash_bytes | The longest evidence hash, in bytes, that a new write may carry. At least 1. | 64 |

---

<a id="ch-gov"></a>

# Governance

<a id="gov-overview"></a>

## How proposals work

Governance is how the chain changes its own rules. Anyone can submit a proposal, which is a request to run one or more messages. Holders who stake URI vote on it. If it passes, the chain runs its messages at the end of the block in which voting ends. Nobody has to send another transaction.

A proposal may carry only governance-only messages. These are the messages that change a module's parameters, pause a token chain-wide, spend from the community pool, or schedule an upgrade. Each one names the governance account as its `authority`. That is the account the chain uses to act on a passed proposal. You get its address with `urid query auth module-account gov`. A proposal that carries any other message, such as a plain send or a vote, is refused at submission.

Passing a vote does not guarantee success. The chain checks each message again when it runs it. If one message fails, none of the proposal's changes are kept, and the proposal ends as failed.

<a id="gov-deposit-and-voting"></a>

### Deposit and voting

A normal proposal needs a deposit of 100 URI before voting starts. You pay at least 10 percent of it, 10 URI, when you submit. Anyone can add the rest with [MsgDeposit](#gov-msg-deposit) during the deposit period. Voting starts as soon as the total reaches 100 URI. Deposits are paid in kwak.

The voting period of a normal proposal is 2 days. Only stake counts. Validators and the people who stake with them vote with the URI they have staked. If you stake and do not vote, your validator's vote counts for your stake. The options are yes, no, abstain, and no with veto, explained under [VoteOption](#gov-type-vote-option). You can also split your stake across options with a [weighted vote](#gov-msg-vote-weighted).

When voting ends, the chain counts the votes against the quorum, the passing threshold, and the veto threshold. Whether deposits come back or are burned depends on the outcome and on the [parameters](#gov-parameters).

<a id="gov-expedited"></a>

### Expedited proposals

An expedited proposal is a faster proposal with a higher passing threshold. The proposer picks its voting period: any duration from 30 seconds up to the expedited voting period, which is 1 day.

A faster proposal costs more. At the longest period, 1 day, the deposit is 500 URI. At 30 seconds it is 10,000 URI. In between, the deposit rises on a straight line as the period gets shorter, rounded up to the next whole kwak. A 1-hour period, for example, needs 9,607,328,933,658 kwak, about 9,608 URI. You pay the whole deposit when you submit, and voting starts at once.

On UriNet you submit an expedited proposal only with [MsgSubmitExpeditedProposal](#gov-msg-submit-expedited-proposal). The standard [MsgSubmitProposal](#gov-msg-submit-proposal) with its `expedited` flag set to true is refused.

If an expedited proposal does not pass on the faster terms, it becomes a normal proposal. Its voting then ends 2 days after its voting began. Votes from the expedited stage do not carry over, so everyone votes again. The deposit stays locked until the normal vote ends.

<a id="gov-payload-messages"></a>

### Messages a proposal may carry

These 18 messages are the only ones a proposal may carry. None of them can be sent as an ordinary transaction.

| Message | Module | What it changes |
|---|---|---|
| [MsgUpdateParams](#smartaccount-msg-update-params) | smartaccount | The SmartAccount parameters. |
| [MsgUpdateParams](#delegation-msg-update-params) | delegation | The delegation parameters. |
| [MsgUpdateParams](#token-msg-update-params) | token | The token module parameters. |
| [MsgSetGovernanceHardPause](#token-msg-set-governance-hard-pause) | token | Stops or restarts every movement of one user token. |
| [MsgUpdateParams](#vesting-msg-update-params) | vesting | The vesting parameters. |
| [MsgUpdateParams](#reputation-msg-update-params) | reputation | The reputation parameters. |
| [MsgUpdateParams](#bank-msg-update-params) | bank | The bank parameters. |
| [MsgSetSendEnabled](#bank-msg-set-send-enabled) | bank | Turns sending on or off for a denom. |
| [MsgUpdateParams](#auth-msg-update-params) | auth | The account parameters. |
| [MsgUpdateParams](#staking-msg-update-params) | staking | The staking parameters. |
| [MsgUpdateParams](#distribution-msg-update-params) | distribution | The reward distribution parameters. |
| [MsgCommunityPoolSpend](#distribution-msg-community-pool-spend) | distribution | Pays coins out of the community pool. |
| [MsgUpdateParams](#slashing-msg-update-params) | slashing | The slashing parameters. |
| [MsgUpdateParams](#mint-msg-update-params) | mint | The mint parameters. |
| [MsgUpdateParams](#consensus-msg-update-params) | consensus | The consensus parameters. |
| [MsgSoftwareUpgrade](#upgrade-msg-software-upgrade) | upgrade | Schedules a software upgrade. |
| [MsgCancelUpgrade](#upgrade-msg-cancel-upgrade) | upgrade | Cancels a scheduled upgrade. |
| [MsgUpdateParams](#gov-msg-update-params) | gov | The governance parameters. |

<a id="gov-type-proposal"></a>

### Proposal

A proposal record holds the messages, the deposit, the timing, and the result.

```proto
message Proposal {
  uint64 id = 1;
  repeated google.protobuf.Any messages = 2;
  ProposalStatus status = 3;
  TallyResult final_tally_result = 4;
  google.protobuf.Timestamp submit_time = 5;
  google.protobuf.Timestamp deposit_end_time = 6;
  repeated cosmos.base.v1beta1.Coin total_deposit = 7;
  google.protobuf.Timestamp voting_start_time = 8;
  google.protobuf.Timestamp voting_end_time = 9;
  string metadata = 10;
  string title = 11;
  string summary = 12;
  string proposer = 13;
  bool expedited = 14;
  string failed_reason = 15;
}
```
| Field | Type | Meaning |
|---|---|---|
| id | number | The proposal's number, given at submission. |
| messages | list of messages | The messages to run if the proposal passes. |
| status | [`ProposalStatus`](#gov-type-proposal-status) | Where the proposal is in its life. |
| final_tally_result | [`TallyResult`](#gov-type-tally-result) | The final count. Empty until voting ends. |
| submit_time | timestamp | When it was submitted. |
| deposit_end_time | timestamp | When the deposit period ends. |
| total_deposit | list of coins | Everything deposited so far. |
| voting_start_time | timestamp | When voting began. |
| voting_end_time | timestamp | When voting ends. For an expedited proposal this is the start plus the period the proposer chose. |
| metadata | string | Free text the proposer attached, such as a link to a document. |
| title | string | The title. |
| summary | string | A short summary. |
| proposer | address | Who submitted it. |
| expedited | boolean | True while the proposal is on expedited terms. |
| failed_reason | string | Why a passed proposal failed when it ran. |

<a id="gov-type-proposal-status"></a>

### ProposalStatus

```proto
enum ProposalStatus {
  PROPOSAL_STATUS_UNSPECIFIED = 0;
  PROPOSAL_STATUS_DEPOSIT_PERIOD = 1;
  PROPOSAL_STATUS_VOTING_PERIOD = 2;
  PROPOSAL_STATUS_PASSED = 3;
  PROPOSAL_STATUS_REJECTED = 4;
  PROPOSAL_STATUS_FAILED = 5;
}
```
| Value | Meaning |
|---|---|
| PROPOSAL_STATUS_DEPOSIT_PERIOD | Waiting for its full deposit. |
| PROPOSAL_STATUS_VOTING_PERIOD | Open for votes. |
| PROPOSAL_STATUS_PASSED | Passed, and its messages ran. |
| PROPOSAL_STATUS_REJECTED | Voted down, or short of quorum. |
| PROPOSAL_STATUS_FAILED | Passed, but a message failed when it ran. Nothing changed. |
| PROPOSAL_STATUS_UNSPECIFIED | No status. Used only as an empty filter. |

<a id="gov-type-vote-option"></a>

### VoteOption

```proto
enum VoteOption {
  VOTE_OPTION_UNSPECIFIED = 0;
  VOTE_OPTION_YES = 1;
  VOTE_OPTION_ABSTAIN = 2;
  VOTE_OPTION_NO = 3;
  VOTE_OPTION_NO_WITH_VETO = 4;
}
```
| Value | Meaning |
|---|---|
| VOTE_OPTION_YES | For the proposal. |
| VOTE_OPTION_NO | Against the proposal. |
| VOTE_OPTION_ABSTAIN | Counts toward quorum without taking a side. |
| VOTE_OPTION_NO_WITH_VETO | Against, and a signal that the proposal should not have been made. Enough veto votes reject the proposal outright. |
| VOTE_OPTION_UNSPECIFIED | Not a valid vote. |

<a id="gov-type-weighted-vote-option"></a>

### WeightedVoteOption

One part of a split vote.

```proto
message WeightedVoteOption {
  VoteOption option = 1;
  string weight = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| option | [`VoteOption`](#gov-type-vote-option) | The option this part goes to. |
| weight | decimal as a string | The share of your stake for this option, such as `"0.6"`. |

<a id="gov-type-vote"></a>

### Vote

```proto
message Vote {
  uint64 proposal_id = 1;
  string voter = 2;
  reserved 3;
  repeated WeightedVoteOption options = 4;
  string metadata = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| voter | address | Who voted. |
| options | list of [`WeightedVoteOption`](#gov-type-weighted-vote-option) | The vote. A simple vote is one option with weight 1. |
| metadata | string | Free text the voter attached. |

<a id="gov-type-deposit"></a>

### Deposit

```proto
message Deposit {
  uint64 proposal_id = 1;
  string depositor = 2;
  repeated cosmos.base.v1beta1.Coin amount = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| depositor | address | Who deposited. |
| amount | list of coins | How much that account has deposited in total. |

<a id="gov-type-tally-result"></a>

### TallyResult

```proto
message TallyResult {
  string yes_count = 1;
  string abstain_count = 2;
  string no_count = 3;
  string no_with_veto_count = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| yes_count | integer as a string | Staked kwak voting yes. |
| abstain_count | integer as a string | Staked kwak voting abstain. |
| no_count | integer as a string | Staked kwak voting no. |
| no_with_veto_count | integer as a string | Staked kwak voting no with veto. |

<a id="gov-transactions"></a>

## Transactions

These are the messages you send to submit, fund, vote on, and cancel proposals.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgSubmitExpeditedProposal](#gov-msg-submit-expedited-proposal) | Submits an expedited proposal with the period you choose. | The proposer |
| [MsgSubmitProposal](#gov-msg-submit-proposal) | Submits a normal proposal. | The proposer |
| [MsgDeposit](#gov-msg-deposit) | Adds to a proposal's deposit. | The depositor |
| [MsgVote](#gov-msg-vote) | Casts a vote. | The voter |
| [MsgVoteWeighted](#gov-msg-vote-weighted) | Casts a vote split across options. | The voter |
| [MsgCancelProposal](#gov-msg-cancel-proposal) | Withdraws a proposal before voting ends. | The proposer |
| [MsgUpdateParams](#gov-msg-update-params) | Replaces the governance parameters. | The governance account, through a passed proposal |

<a id="gov-tx-submit"></a>

### Submitting proposals

<a id="gov-msg-submit-expedited-proposal"></a>

#### MsgSubmitExpeditedProposal

This is UriNet's own message for an expedited proposal. You choose the voting period, from 30 seconds up to the expedited voting period of 1 day. You pay the whole deposit for that period in the same transaction, and voting starts at once.

Use it when a change cannot wait 2 days. Suppose a user token is being used in an attack. The proposer can ask for a 1-hour vote to pause it, at a deposit of about 9,608 URI. The voting end time is the start plus the chosen period.

| | |
|---|---|
| Signer | `proposer`, the account submitting the proposal |
| Before you send it | Every message in the proposal is one of the [18 governance-only messages](#gov-payload-messages) and names the governance account as its authority. You hold the full expedited deposit for your period. |
| What comes back | `proposal_id`, the new proposal's number |
| urid | `urid tx gov submit-expedited-proposal [path/to/proposal.json] [voting-period]` |

```proto
message MsgSubmitExpeditedProposal {
  reserved 7;
  reserved "expedited";
  repeated google.protobuf.Any messages = 1;
  repeated cosmos.base.v1beta1.Coin initial_deposit = 2;
  string proposer = 3;
  string metadata = 4;
  string title = 5;
  string summary = 6;
  google.protobuf.Duration voting_period = 8;
}

message MsgSubmitExpeditedProposalResponse {
  uint64 proposal_id = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| messages | list of messages | The messages to run if it passes. Each is a JSON object with its `@type`. |
| initial_deposit | list of coins | The deposit. It must cover the full expedited deposit for `voting_period`. |
| proposer | address | Who submits. The proposer gains no authority from it. |
| metadata | string | Free text, such as a link to a document. |
| title | string | The title. |
| summary | string | A short summary. |
| voting_period | duration | The voting period you choose, from 30 seconds to the expedited voting period. |
| proposal_id | number | In the response. The new proposal's number. |

The command takes the proposal as a JSON file and the period as a duration such as `30s`, `90m`, or `24h`. The file has the same fields as for a normal proposal, but no `expedited` field at all. The command refuses a file that has one, even set to false. This file asks for a hard pause of a token:

```json
{
  "messages": [
    {
      "@type": "/urinet.token.v1.MsgSetGovernanceHardPause",
      "authority": "uri1...",
      "denom": "token/uri1.../shop",
      "paused": true
    }
  ],
  "metadata": "",
  "deposit": "9607328933658kwak",
  "title": "Pause the shop token",
  "summary": "Stops every movement of the token while the issuer investigates."
}
```

Compose the transaction with `--generate-only`, then sign and broadcast it as in [Transactions](#urid-tx).

```sh
urid tx gov submit-expedited-proposal proposal.json 1h --from <proposer-address> --generate-only > tx.json
urid tx sign tx.json <proposer-address> --credential laptop --output-document signed.json
urid tx broadcast signed.json
```

**Refused when**

- The voting period is shorter than 30 seconds or longer than the expedited voting period.
- The deposit is short of the full expedited deposit for the chosen period.
- The deposit is in a denom other than kwak.
- A message is not one of the 18 governance-only messages.
- A message does not have the governance account as its only signer.
- The title or summary is empty, or a text field is longer than the chain allows.

<a id="gov-msg-submit-proposal"></a>

#### MsgSubmitProposal

This submits a normal proposal. The deposit period starts, and voting starts once the deposit reaches 100 URI. Voting then lasts 2 days.

You pay at least 10 URI at submission. Others can add to the deposit with [MsgDeposit](#gov-msg-deposit). Alice can submit a proposal to change a module parameter with 10 URI, and Bob can add the remaining 90 URI.

| | |
|---|---|
| Signer | `proposer`, the account submitting the proposal |
| Before you send it | Every message is one of the [18 governance-only messages](#gov-payload-messages) and names the governance account as its authority. `expedited` is false or left out. |
| What comes back | `proposal_id`, the new proposal's number |
| urid | `urid tx gov submit-proposal [path/to/proposal.json]` |

```proto
message MsgSubmitProposal {
  repeated google.protobuf.Any messages = 1;
  repeated cosmos.base.v1beta1.Coin initial_deposit = 2;
  string proposer = 3;
  string metadata = 4;
  string title = 5;
  string summary = 6;
  bool expedited = 7;
}

message MsgSubmitProposalResponse {
  uint64 proposal_id = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| messages | list of messages | The messages to run if it passes. Each is a JSON object with its `@type`. |
| initial_deposit | list of coins | The deposit paid now, at least 10 percent of 100 URI. |
| proposer | address | Who submits. |
| metadata | string | Free text, such as a link to a document. |
| title | string | The title. |
| summary | string | A short summary. |
| expedited | boolean | Must be false. Use [MsgSubmitExpeditedProposal](#gov-msg-submit-expedited-proposal) for an expedited proposal. |
| proposal_id | number | In the response. The new proposal's number. |

The proposal file has the fields `messages`, `metadata`, `deposit`, `title`, `summary`, and `expedited`. `urid tx gov draft-proposal` asks a few questions and writes a skeleton file for you. Its file includes `expedited`, so remove that field before you use the file with `submit-expedited-proposal`. Some modules also offer a shortcut that submits a proposal for one message, such as `urid tx bank update-params-proposal`. Governance has no such shortcut for its own parameters. See [Governance proposals](#urid-gov).

**Refused when**

- `expedited` is true.
- The initial deposit is less than 10 percent of the minimum deposit.
- The deposit is in a denom other than kwak.
- A message is not one of the 18 governance-only messages.
- A message does not have the governance account as its only signer.
- The title or summary is empty, or a text field is longer than the chain allows.

<a id="gov-tx-vote"></a>

### Deposits and votes

<a id="gov-msg-deposit"></a>

#### MsgDeposit

This adds coins to a proposal's deposit. When the total reaches the minimum deposit, voting starts. Any account can deposit, not only the proposer.

| | |
|---|---|
| Signer | `depositor`, the account paying |
| Before you send it | The proposal is still in its deposit period or voting period. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx gov deposit [proposal-id] [deposit]` |

```proto
message MsgDeposit {
  uint64 proposal_id = 1;
  string depositor = 2;
  repeated cosmos.base.v1beta1.Coin amount = 3;
}

message MsgDepositResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| depositor | address | Who pays. |
| amount | list of coins | How much to add, in kwak. |

**Refused when**

- The proposal does not exist, or its deposit and voting periods are over.
- The amount is in a denom other than kwak.
- The amount is below the smallest deposit the chain accepts, a share of the minimum deposit set by `min_deposit_ratio`.
- The depositor does not have the coins.

<a id="gov-msg-vote"></a>

#### MsgVote

This casts your vote on a proposal in its voting period. Your vote carries the weight of your staked URI. Voting again replaces your earlier vote.

| | |
|---|---|
| Signer | `voter` |
| Before you send it | The proposal is in its voting period. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx gov vote [proposal-id] [option]`, where option is `yes`, `no`, `abstain`, or `no_with_veto` |

```proto
message MsgVote {
  uint64 proposal_id = 1;
  string voter = 2;
  VoteOption option = 3;
  string metadata = 4;
}

message MsgVoteResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| voter | address | Who votes. |
| option | [`VoteOption`](#gov-type-vote-option) | YES, NO, ABSTAIN, or NO_WITH_VETO. |
| metadata | string | Free text attached to the vote. |

**Refused when**

- The proposal does not exist or is not in its voting period.
- The option is not one of the four valid options.
- The metadata is too long for the chain.

<a id="gov-msg-vote-weighted"></a>

#### MsgVoteWeighted

This splits your vote across several options. It suits an account that votes for many people with different views. The shop might put 0.7 of its stake on yes and 0.3 on abstain.

| | |
|---|---|
| Signer | `voter` |
| Before you send it | The proposal is in its voting period. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx gov weighted-vote [proposal-id] [weighted-options]`, for example `yes=0.7,abstain=0.3` |

```proto
message MsgVoteWeighted {
  uint64 proposal_id = 1;
  string voter = 2;
  repeated WeightedVoteOption options = 3;
  string metadata = 4;
}

message MsgVoteWeightedResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| voter | address | Who votes. |
| options | list of [`WeightedVoteOption`](#gov-type-weighted-vote-option) | The options and their weights. |
| metadata | string | Free text attached to the vote. |

**Refused when**

- The proposal does not exist or is not in its voting period.
- The weights do not add up to exactly 1.
- An option appears twice, or a weight is zero or negative.
- The metadata is too long for the chain.

<a id="gov-tx-cancel"></a>

### Cancelling a proposal

<a id="gov-msg-cancel-proposal"></a>

#### MsgCancelProposal

This lets the proposer withdraw a proposal before its voting ends. A share of the deposits set by `proposal_cancel_ratio` is not returned. The rest goes back to the depositors. At the starting values, half of the deposits is burned.

| | |
|---|---|
| Signer | `proposer`, the account that submitted the proposal |
| Before you send it | You are the proposer, and voting has not ended. |
| What comes back | `proposal_id`, `canceled_time`, and `canceled_height` |
| urid | `urid tx gov cancel-proposal [proposal-id]` |

```proto
message MsgCancelProposal {
  uint64 proposal_id = 1;
  string proposer = 2;
}

message MsgCancelProposalResponse {
  uint64 proposal_id = 1;
  google.protobuf.Timestamp canceled_time = 2;
  uint64 canceled_height = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal to cancel. |
| proposer | address | The proposer. Must sign. |
| canceled_time | timestamp | In the response. When it was cancelled. |
| canceled_height | number | In the response. The block height of the cancellation. |

**Refused when**

- The proposal does not exist.
- The signer is not the proposer.
- The voting period has already ended.

<a id="gov-tx-params"></a>

### Governance parameters

<a id="gov-msg-update-params"></a>

#### MsgUpdateParams

This replaces the whole set of governance parameters. You must give every parameter, not only the ones you change. A parameter left out of the file becomes zero, not its current value. Read the current set with [Params](#gov-query-params) first and edit it.

The denoms of the two minimum deposits cannot change. Both stay in kwak. Amounts, periods, and thresholds can change.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | the governance account, through a passed proposal |
| Before you send it | The proposal file carries the full new parameter set. |
| What comes back | nothing; the proposal's status tells you it went through |
| urid | none of its own; put the message in the file of `urid tx gov submit-proposal` |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#gov-parameters) | The full new parameter set. |

**Refused when**

- The authority is not the governance account.
- The new `min_deposit` or `expedited_min_deposit` uses a different set of denoms.
- A value is out of range, such as a threshold above 1 or an expedited voting period that is not shorter than the normal one.

<a id="gov-tx-refused"></a>

### Not available

The chain refuses these messages before the fee is taken:

- `cosmos.gov.v1.MsgExecLegacyContent`, the wrapper for old-style proposal content.
- The four older governance messages: `cosmos.gov.v1beta1.MsgSubmitProposal`, `MsgVote`, `MsgVoteWeighted`, and `MsgDeposit`.

`urid tx gov submit-legacy-proposal` builds one of these, so it does not work on UriNet. Use `submit-proposal` or `submit-expedited-proposal`.

<a id="gov-queries"></a>

## Queries

These queries read proposals, votes, deposits, and the governance parameters.

| Query | What it answers |
|---|---|
| [Proposal](#gov-query-proposal) | One proposal. |
| [Proposals](#gov-query-proposals) | Proposals, filtered by status, voter, or depositor. |
| [Vote](#gov-query-vote) | One account's vote on a proposal. |
| [Votes](#gov-query-votes) | All votes on a proposal. |
| [Deposit](#gov-query-deposit) | One account's deposit on a proposal. |
| [Deposits](#gov-query-deposits) | All deposits on a proposal. |
| [TallyResult](#gov-query-tally-result) | The current count for a proposal. |
| [Params](#gov-query-params) | The governance parameters. |
| [Constitution](#gov-query-constitution) | The chain's constitution text. |

List queries take a `pagination` request with the usual `key`, `offset`, `limit`, `count_total` and `reverse` fields. The 200-entry cap, the default page of 100, and the refusal of `offset` and `count_total` apply only to UriNet's own list queries. See [Reading lists](#connect-conventions).

<a id="gov-q-proposals"></a>

### Proposals

<a id="gov-query-proposal"></a>

#### Proposal

Returns one proposal with its messages, status, deposit, and timing. Ask it to follow a proposal you care about.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Proposal` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}` |
| urid | `urid query gov proposal [proposal-id]` |

```proto
message QueryProposalRequest {
  uint64 proposal_id = 1;
}

message QueryProposalResponse {
  Proposal proposal = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| proposal | [`Proposal`](#gov-type-proposal) | In the response. The proposal record. |

<a id="gov-query-proposals"></a>

#### Proposals

Lists proposals. You can filter by status, by an account that voted, or by an account that deposited. Ask it to find open votes.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Proposals` |
| REST | `GET /cosmos/gov/v1/proposals` |
| urid | `urid query gov proposals`, with `--proposal-status`, `--voter`, or `--depositor` |

```proto
message QueryProposalsRequest {
  ProposalStatus proposal_status = 1;
  string voter = 2;
  string depositor = 3;
  cosmos.base.query.v1beta1.PageRequest pagination = 4;
}

message QueryProposalsResponse {
  repeated Proposal proposals = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_status | [`ProposalStatus`](#gov-type-proposal-status) | Only proposals in this status. Leave empty for all. On the command line: `deposit-period`, `voting-period`, `passed`, `rejected`, or `failed`. |
| voter | address | Only proposals this account voted on. |
| depositor | address | Only proposals this account deposited on. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| proposals | list of [`Proposal`](#gov-type-proposal) | In the response. The matching proposals. |

<a id="gov-query-tally-result"></a>

#### TallyResult

Returns the count for a proposal. During voting it shows the count so far. After voting it shows the final result.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/TallyResult` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}/tally` |
| urid | `urid query gov tally [proposal-id]` |

```proto
message QueryTallyResultRequest {
  uint64 proposal_id = 1;
}

message QueryTallyResultResponse {
  TallyResult tally = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| tally | [`TallyResult`](#gov-type-tally-result) | In the response. The count per option. |

<a id="gov-q-votes"></a>

### Votes

<a id="gov-query-vote"></a>

#### Vote

Returns how one account voted on a proposal. Ask it to confirm your vote landed.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Vote` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}/votes/{voter}` |
| urid | `urid query gov vote [proposal-id] [voter-addr]` |

```proto
message QueryVoteRequest {
  uint64 proposal_id = 1;
  string voter = 2;
}

message QueryVoteResponse {
  Vote vote = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| voter | address | The voter. |
| vote | [`Vote`](#gov-type-vote) | In the response. The vote. |

<a id="gov-query-votes"></a>

#### Votes

Lists every vote cast on a proposal.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Votes` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}/votes` |
| urid | `urid query gov votes [proposal-id]` |

```proto
message QueryVotesRequest {
  uint64 proposal_id = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryVotesResponse {
  repeated Vote votes = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| votes | list of [`Vote`](#gov-type-vote) | In the response. The votes. |

<a id="gov-q-deposits"></a>

### Deposits

<a id="gov-query-deposit"></a>

#### Deposit

Returns how much one account has deposited on a proposal.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Deposit` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}/deposits/{depositor}` |
| urid | `urid query gov deposit [proposal-id] [depositor-addr]` |

```proto
message QueryDepositRequest {
  uint64 proposal_id = 1;
  string depositor = 2;
}

message QueryDepositResponse {
  Deposit deposit = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| depositor | address | The depositor. |
| deposit | [`Deposit`](#gov-type-deposit) | In the response. The deposit. |

<a id="gov-query-deposits"></a>

#### Deposits

Lists every deposit on a proposal. Ask it to see how far a proposal is from its minimum deposit.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Deposits` |
| REST | `GET /cosmos/gov/v1/proposals/{proposal_id}/deposits` |
| urid | `urid query gov deposits [proposal-id]` |

```proto
message QueryDepositsRequest {
  uint64 proposal_id = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryDepositsResponse {
  repeated Deposit deposits = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| proposal_id | number | The proposal. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| deposits | list of [`Deposit`](#gov-type-deposit) | In the response. The deposits. |

<a id="gov-q-rules"></a>

### Rules

<a id="gov-query-params"></a>

#### Params

Returns the governance parameters. Ask it before you submit, to see the current deposits and periods.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Params` |
| REST | `GET /cosmos/gov/v1/params/{params_type}` |
| urid | `urid query gov params` |

```proto
message QueryParamsRequest {
  string params_type = 1;
}

message QueryParamsResponse {
  VotingParams voting_params = 1;
  DepositParams deposit_params = 2;
  TallyParams tally_params = 3;
  Params params = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| params_type | string | `voting`, `tallying`, or `deposit`. It goes into the REST path. |
| params | [`Params`](#gov-parameters) | In the response. All the parameters. Read this field. |
| voting_params, deposit_params, tally_params | older groupings | In the response. Kept only for older clients. |

<a id="gov-query-constitution"></a>

#### Constitution

Returns the text the chain holds as its constitution.

| | |
|---|---|
| gRPC | `cosmos.gov.v1.Query/Constitution` |
| REST | `GET /cosmos/gov/v1/constitution` |
| urid | `urid query gov constitution` |

```proto
message QueryConstitutionRequest {}

message QueryConstitutionResponse {
  string constitution = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| constitution | string | In the response. The constitution text. |

<a id="gov-parameters"></a>

## Parameters

Parameters are the chain-wide settings of governance itself. Governance changes them through [MsgUpdateParams](#gov-msg-update-params) inside a proposal. The values below are the ones the network starts with. Read the current ones with the [Params](#gov-query-params) query.

```proto
message Params {
  repeated cosmos.base.v1beta1.Coin min_deposit = 1;
  google.protobuf.Duration max_deposit_period = 2;
  google.protobuf.Duration voting_period = 3;
  string quorum = 4;
  string threshold = 5;
  string veto_threshold = 6;
  string min_initial_deposit_ratio = 7;
  string proposal_cancel_ratio = 8;
  string proposal_cancel_dest = 9;
  google.protobuf.Duration expedited_voting_period = 10;
  string expedited_threshold = 11;
  repeated cosmos.base.v1beta1.Coin expedited_min_deposit = 12;
  bool burn_vote_quorum = 13;
  bool burn_proposal_deposit_prevote = 14;
  bool burn_vote_veto = 15;
  string min_deposit_ratio = 16;
}
```
The 30-second shortest expedited period and the factor of 20 between the longest and shortest expedited deposits are fixed by the chain. They are not parameters.

| Parameter | Meaning | Value |
|---|---|---|
| min_deposit | The total deposit a normal proposal needs before voting starts. | 100 URI, `100000000000kwak` |
| max_deposit_period | How long a proposal waits for its full deposit. | 2 days |
| voting_period | The voting period of a normal proposal. | 2 days |
| quorum | The share of all staked URI that must vote for the result to count. | 0.334 |
| threshold | The share of yes votes, among votes other than abstain, a proposal needs to pass. | 0.50 |
| veto_threshold | The share of no with veto votes that rejects a proposal outright. | 0.334 |
| min_initial_deposit_ratio | The share of `min_deposit` you pay at submission. | 0.10 |
| proposal_cancel_ratio | The share of deposits not returned when a proposal is cancelled. | 0.50 |
| proposal_cancel_dest | Who receives that share. Empty means it is burned. | empty |
| expedited_voting_period | The longest voting period an expedited proposal may choose. | 1 day |
| expedited_threshold | The share of yes votes an expedited proposal needs to pass. | 0.667 |
| expedited_min_deposit | The expedited deposit at the longest expedited period. The deposit at 30 seconds is 20 times this. | 500 URI, `500000000000kwak` |
| burn_vote_quorum | Burn the deposits when a proposal misses quorum. | false |
| burn_proposal_deposit_prevote | Burn the deposits when a proposal never reaches voting. | false |
| burn_vote_veto | Burn the deposits when no with veto votes reject a proposal. | true |
| min_deposit_ratio | The smallest single deposit the chain accepts, as a share of `min_deposit`. | 0.01 |

---

<a id="ch-standard"></a>

# Standard modules

<a id="std-overview"></a>

## What UriNet keeps

UriNet accepts exactly 77 message types. Forty-five are its own. Thirty-two come from the standard modules, the general-purpose modules UriNet shares with other chains of its kind. The chain refuses any other message before the fee is taken.

These are the 32 standard messages. The ones marked governance only travel inside a [governance proposal](#ch-gov) and cannot be sent as ordinary transactions.

| Module | Message | Governance only |
|---|---|---|
| bank | [MsgSend](#bank-msg-send) | |
| bank | [MsgMultiSend](#bank-msg-multi-send) | |
| bank | [MsgSetSendEnabled](#bank-msg-set-send-enabled) | yes |
| bank | [MsgUpdateParams](#bank-msg-update-params) | yes |
| distribution | [MsgSetWithdrawAddress](#distribution-msg-set-withdraw-address) | |
| distribution | [MsgWithdrawDelegatorReward](#distribution-msg-withdraw-delegator-reward) | |
| distribution | [MsgWithdrawValidatorCommission](#distribution-msg-withdraw-validator-commission) | |
| distribution | [MsgCommunityPoolSpend](#distribution-msg-community-pool-spend) | yes |
| distribution | [MsgUpdateParams](#distribution-msg-update-params) | yes |
| staking | [MsgCreateValidator](#staking-msg-create-validator) | |
| staking | [MsgEditValidator](#staking-msg-edit-validator) | |
| staking | [MsgDelegate](#staking-msg-delegate) | |
| staking | [MsgBeginRedelegate](#staking-msg-begin-redelegate) | |
| staking | [MsgUndelegate](#staking-msg-undelegate) | |
| staking | [MsgCancelUnbondingDelegation](#staking-msg-cancel-unbonding-delegation) | |
| staking | [MsgRotateConsPubKey](#staking-msg-rotate-cons-pub-key) | |
| staking | [MsgUpdateParams](#staking-msg-update-params) | yes |
| gov | [MsgSubmitProposal](#gov-msg-submit-proposal) | |
| gov | [MsgVote](#gov-msg-vote) | |
| gov | [MsgVoteWeighted](#gov-msg-vote-weighted) | |
| gov | [MsgDeposit](#gov-msg-deposit) | |
| gov | [MsgCancelProposal](#gov-msg-cancel-proposal) | |
| gov | [MsgUpdateParams](#gov-msg-update-params) | yes |
| mint | [MsgUpdateParams](#mint-msg-update-params) | yes |
| slashing | [MsgUnjail](#slashing-msg-unjail) | |
| slashing | [MsgUpdateParams](#slashing-msg-update-params) | yes |
| upgrade | [MsgSoftwareUpgrade](#upgrade-msg-software-upgrade) | yes |
| upgrade | [MsgCancelUpgrade](#upgrade-msg-cancel-upgrade) | yes |
| consensus | [MsgUpdateParams](#consensus-msg-update-params) | yes |
| auth | [MsgUpdateParams](#auth-msg-update-params) | yes |
| feegrant | [MsgGrantAllowance](#feegrant-msg-grant-allowance) | |
| feegrant | [MsgRevokeAllowance](#feegrant-msg-revoke-allowance) | |

These nine standard messages exist in the standard modules, but the chain refuses each one before the fee is taken:

- `cosmos.distribution.v1beta1.MsgFundCommunityPool`
- `cosmos.distribution.v1beta1.MsgDepositValidatorRewardsPool`
- `cosmos.gov.v1.MsgExecLegacyContent`
- `cosmos.gov.v1beta1.MsgSubmitProposal`
- `cosmos.gov.v1beta1.MsgVote`
- `cosmos.gov.v1beta1.MsgVoteWeighted`
- `cosmos.gov.v1beta1.MsgDeposit`
- `cosmos.evidence.v1beta1.MsgSubmitEvidence`
- `cosmos.feegrant.v1beta1.MsgPruneAllowances`

The queries of the standard modules are all available. Their list queries take `pagination` with the usual `key`, `offset`, `limit`, `count_total` and `reverse` fields. The 200-entry cap, the default page of 100, and the refusal of `offset` and `count_total` apply only to UriNet's own list queries. See [Reading lists](#connect-conventions).

A module command such as `urid tx bank send` signs and sends in one step only for an account with a single credential whose address is the key's address, and only with an explicit `--gas`. Otherwise compose with `--generate-only` and sign with `urid tx sign`. See [Transactions](#urid-tx).

<a id="std-bank"></a>

## Bank

<a id="bank-overview"></a>

### What it is

The bank module holds every balance on the chain and moves coins between accounts. It handles URI, counted in kwak, and every user token, whose denom is `token/<issuer address>/<subdenom>`. You use it to send coins, read balances, and look up supply.

UriNet keeps the standard messages but adds its own rules to every movement:

- A recovery-locked account cannot move coins. Recovery-locked means a recovery of the account is in progress. Ordinary accounts cannot send to it either until the recovery ends. See [SmartAccounts](#ch-smartaccount).
- A user token's rules apply to every movement of that token. These are the governance hard pause, the issuer pause, a holder's frozen amount, the blocklist and allowlist, the transfer restriction, the holding limit per account, and the maximum holder count. See [Tokens](#ch-token).
- Coins still locked by a vesting schedule cannot be spent. See [Vesting](#ch-vesting).
- Fees are paid in kwak only, whatever the transaction moves.

`send_enabled` is a separate, coarse switch. It turns sending on or off for a whole denom, and only governance sets it. It is not a token's pause or transfer restriction.

<a id="bank-type-coin"></a>

#### Coin

A coin is one denom and an amount.

```proto
message Coin {
  string denom = 1;
  string amount = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | `kwak` for URI, or a token denom such as `token/uri1.../shop`. |
| amount | integer as a string | Whole base units. 1 URI is `"1000000000"` kwak. |

A list of coins is sorted by denom, with each denom once and each amount above zero. On the command line you write coins as amount and denom together, such as `5000000000kwak`, separated by commas.

<a id="bank-type-input-output"></a>

#### Input and Output

The two sides of a [MsgMultiSend](#bank-msg-multi-send).

```proto
message Input {
  string address = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}

message Output {
  string address = 1;
  repeated cosmos.base.v1beta1.Coin coins = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | In an Input, the sender. In an Output, a recipient. |
| coins | list of coins | What the sender pays, or what the recipient receives. |

<a id="bank-type-send-enabled"></a>

#### SendEnabled

```proto
message SendEnabled {
  string denom = 1;
  bool enabled = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| enabled | boolean | Whether that denom can be sent. A denom with no entry follows `default_send_enabled`. |

<a id="bank-type-metadata"></a>

#### Metadata and DenomUnit

Display information for a denom, such as its name, symbol, and larger units. Wallets use it to show amounts.

```proto
message Metadata {
  string description = 1;
  repeated DenomUnit denom_units = 2;
  string base = 3;
  string display = 4;
  string name = 5;
  string symbol = 6;
  string uri = 7;
  string uri_hash = 8;
}

message DenomUnit {
  string denom = 1;
  uint32 exponent = 2;
  repeated string aliases = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| description | string | A description of the coin. |
| denom_units | list of `DenomUnit` | The units, from the base unit up. |
| base | string | The base denom, the unit with exponent 0. |
| display | string | The unit a wallet should show. |
| name | string | The name. |
| symbol | string | The ticker symbol. |
| uri | string | A link to a document about the coin. Optional. |
| uri_hash | string | A SHA-256 hash of that document. Optional. |
| denom | string | In a DenomUnit, the unit's name. |
| exponent | number | In a DenomUnit, the power of 10 that one unit holds in base units. |
| aliases | list of strings | In a DenomUnit, other names for the unit. |

<a id="bank-transactions"></a>

### Transactions

You send coins with the first two messages. Governance uses the other two.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgSend](#bank-msg-send) | Sends coins to one account. | The sender |
| [MsgMultiSend](#bank-msg-multi-send) | Sends coins from one account to several. | The sender |
| [MsgSetSendEnabled](#bank-msg-set-send-enabled) | Turns sending on or off for denoms. | The governance account, through a passed proposal |
| [MsgUpdateParams](#bank-msg-update-params) | Replaces the bank parameters. | The governance account, through a passed proposal |

<a id="bank-msg-send"></a>

#### MsgSend

This moves coins from your account to another. It works for URI and for user tokens, and one message can carry several denoms. Alice pays Bob 5 URI by sending `5000000000kwak`.

| | |
|---|---|
| Signer | `from_address`, the sender |
| Before you send it | You hold the coins, outside any vesting lock or frozen amount. For a user token, the token's rules allow this movement. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx bank send [from_key_or_address] [to_address] [amount]` |

```proto
message MsgSend {
  string from_address = 1;
  string to_address = 2;
  repeated cosmos.base.v1beta1.Coin amount = 3;
}

message MsgSendResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| from_address | address | The sender. Must sign. |
| to_address | address | The recipient. |
| amount | list of coins | What to send. |

The command takes the sender in place of `--from`.

**Refused when**

- The sender or the recipient is recovery-locked.
- The amount is more than the sender can spend, counting vesting locks and, for a user token, the frozen amount.
- A user token's rules forbid it: a pause, a blocklist entry, the transfer restriction, the recipient's holding limit, or the maximum holder count.
- A token that needs approval for each transfer is sent this way. Use [MsgApprovedTransfer](#token-msg-approved-transfer).
- The sender and recipient are the same account and the coin is a user token.
- Sending is turned off for the denom, or the recipient is a module account that cannot receive coins.

<a id="bank-msg-multi-send"></a>

#### MsgMultiSend

This sends coins from one account to several recipients in one message. The shop can pay three suppliers at once. The total of the outputs must equal the input, denom by denom.

| | |
|---|---|
| Signer | the address in the one input, the sender |
| Before you send it | Exactly one input. The outputs add up to the input for each denom. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx bank multi-send [from_key_or_address] [to_address_1 to_address_2 ...] [amount]` |

```proto
message MsgMultiSend {
  repeated Input inputs = 1;
  repeated Output outputs = 2;
}

message MsgMultiSendResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| inputs | list of [`Input`](#bank-type-input-output) | Exactly one: the sender and the total it pays. |
| outputs | list of [`Output`](#bank-type-input-output) | The recipients and what each receives. |

The command sends `[amount]` to each address. With `--split` it divides `[amount]` equally among them.

**Refused when**

- There is no input, more than one input, or no output.
- The outputs do not add up to the input for each denom.
- The sender or any recipient is recovery-locked.
- The total is more than the sender can spend, counting vesting locks and frozen amounts.
- A user token's rules forbid any one of the movements, including the holding limit of a recipient that appears more than once.
- Sending is turned off for a denom, or a recipient is a module account that cannot receive coins.

<a id="bank-msg-set-send-enabled"></a>

#### MsgSetSendEnabled

This turns sending on or off for one or more denoms. Entries you leave out stay as they are.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | the governance account, through a passed proposal |
| Before you send it | The proposal carries this message with the governance account as `authority`. |
| What comes back | nothing; the proposal's status tells you it went through |
| urid | `urid tx bank set-send-enabled-proposal [send_enabled]`, which submits a normal proposal carrying it |

```proto
message MsgSetSendEnabled {
  string authority = 1;
  repeated SendEnabled send_enabled = 2;
  repeated string use_default_for = 3;
}

message MsgSetSendEnabledResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| send_enabled | list of [`SendEnabled`](#bank-type-send-enabled) | The entries to add or change. |
| use_default_for | list of strings | Denoms whose entry is removed, so they follow `default_send_enabled` again. |

The command takes the entry as JSON, such as `'{"denom":"kwak","enabled":true}'`. Use `--use-default-for` for removals, and `--deposit`, `--title`, and `--summary` for the proposal.

**Refused when**

- The authority is not the governance account.
- A denom appears twice.
- A denom is not a valid denom.

<a id="bank-msg-update-params"></a>

#### MsgUpdateParams

This replaces the bank parameters. You must give the whole parameter set. A parameter left out becomes zero, so leaving out `default_send_enabled` sets it to false.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | the governance account, through a passed proposal |
| Before you send it | The proposal carries the full new parameter set. |
| What comes back | nothing; the proposal's status tells you it went through |
| urid | `urid tx bank update-params-proposal [params]`, which submits a normal proposal carrying it |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#bank-parameters) | The full new parameter set. |

**Refused when**

- The authority is not the governance account.
- The parameters contain the old `send_enabled` list. Use [MsgSetSendEnabled](#bank-msg-set-send-enabled) instead.

<a id="bank-queries"></a>

### Queries

These queries read balances, supply, denom metadata, and the send switches.

| Query | What it answers |
|---|---|
| [Balance](#bank-query-balance) | One account's balance of one denom. |
| [AllBalances](#bank-query-all-balances) | All of one account's balances. |
| [SpendableBalances](#bank-query-spendable-balances) | What one account can spend, all denoms. |
| [SpendableBalanceByDenom](#bank-query-spendable-balance-by-denom) | What one account can spend of one denom. |
| [TotalSupply](#bank-query-total-supply) | The supply of every denom. |
| [SupplyOf](#bank-query-supply-of) | The supply of one denom. |
| [DenomOwners](#bank-query-denom-owners) | Who holds a denom. |
| [DenomOwnersByQuery](#bank-query-denom-owners-by-query) | Who holds a denom, with the denom in the query string. |
| [DenomMetadata](#bank-query-denom-metadata) | Display information for one denom. |
| [DenomMetadataByQueryString](#bank-query-denom-metadata-by-query-string) | The same, with the denom in the query string. |
| [DenomsMetadata](#bank-query-denoms-metadata) | Display information for every denom that has it. |
| [SendEnabled](#bank-query-send-enabled) | Which denoms have their own send switch. |
| [Params](#bank-query-params) | The bank parameters. |

A token denom contains slashes. Where a REST route puts the denom in the path, you write it as it is, slashes included. Where it goes in the query string as `?denom=`, encode each slash as `%2F`.

<a id="bank-query-balance"></a>

#### Balance

Returns how much of one denom an account holds. This is the full balance, including any part that is locked or frozen.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/Balance` |
| REST | `GET /cosmos/bank/v1beta1/balances/{address}/by_denom?denom=...` |
| urid | `urid query bank balance [address] [denom]` |

```proto
message QueryBalanceRequest {
  string address = 1;
  string denom = 2;
}

message QueryBalanceResponse {
  cosmos.base.v1beta1.Coin balance = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The holder. |
| denom | string | The denom. |
| balance | coin | In the response. The full balance. |

<a id="bank-query-all-balances"></a>

#### AllBalances

Returns every denom an account holds, with full balances.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/AllBalances` |
| REST | `GET /cosmos/bank/v1beta1/balances/{address}` |
| urid | `urid query bank balances [address]` |

```proto
message QueryAllBalancesRequest {
  string address = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
  bool resolve_denom = 3;
}

message QueryAllBalancesResponse {
  repeated cosmos.base.v1beta1.Coin balances = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The holder. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| resolve_denom | boolean | Show each denom by its display name from metadata. `--resolve-denom` on the command line. |
| balances | list of coins | In the response. The balances. |

<a id="bank-query-spendable-balances"></a>

#### SpendableBalances

Returns what an account can spend, denom by denom. It leaves out coins locked by a vesting schedule. It does not know about a user token's frozen amount or rules. For a user token, ask the token module's [UsableBalance](#token-query-usable-balance).

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/SpendableBalances` |
| REST | `GET /cosmos/bank/v1beta1/spendable_balances/{address}` |
| urid | `urid query bank spendable-balances [address]` |

```proto
message QuerySpendableBalancesRequest {
  string address = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QuerySpendableBalancesResponse {
  repeated cosmos.base.v1beta1.Coin balances = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The holder. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| balances | list of coins | In the response. The balance minus vesting locks, per denom. |

<a id="bank-query-spendable-balance-by-denom"></a>

#### SpendableBalanceByDenom

Returns what an account can spend of one denom, leaving out vesting locks. As with [SpendableBalances](#bank-query-spendable-balances), a token's frozen amount is not counted.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/SpendableBalanceByDenom` |
| REST | `GET /cosmos/bank/v1beta1/spendable_balances/{address}/by_denom?denom=...` |
| urid | `urid query bank spendable-balance [address] [denom]` |

```proto
message QuerySpendableBalanceByDenomRequest {
  string address = 1;
  string denom = 2;
}

message QuerySpendableBalanceByDenomResponse {
  cosmos.base.v1beta1.Coin balance = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The holder. |
| denom | string | The denom. |
| balance | coin | In the response. The balance minus vesting locks. |

<a id="bank-query-total-supply"></a>

#### TotalSupply

Returns the total supply of every denom on the chain.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/TotalSupply` |
| REST | `GET /cosmos/bank/v1beta1/supply` |
| urid | `urid query bank total-supply` |

```proto
message QueryTotalSupplyRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryTotalSupplyResponse {
  repeated cosmos.base.v1beta1.Coin supply = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| supply | list of coins | In the response. The supply of each denom. |

<a id="bank-query-supply-of"></a>

#### SupplyOf

Returns the total supply of one denom. The issuer can use it to check a token's supply after minting.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/SupplyOf` |
| REST | `GET /cosmos/bank/v1beta1/supply/by_denom?denom=...` |
| urid | `urid query bank total-supply-of [denom]` |

```proto
message QuerySupplyOfRequest {
  string denom = 1;
}

message QuerySupplyOfResponse {
  cosmos.base.v1beta1.Coin amount = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| amount | coin | In the response. The supply. |

<a id="bank-query-denom-owners"></a>

#### DenomOwners

Lists the accounts that hold a denom, with each balance. The denom goes into the path as it is, slashes included.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/DenomOwners` |
| REST | `GET /cosmos/bank/v1beta1/denom_owners/{denom}` |
| urid | `urid query bank denom-owners [denom]` |

```proto
message QueryDenomOwnersRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message DenomOwner {
  string address = 1;
  cosmos.base.v1beta1.Coin balance = 2;
}

message QueryDenomOwnersResponse {
  repeated DenomOwner denom_owners = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| denom_owners | list of `DenomOwner` | In the response. Each holder's `address` and `balance`. |

<a id="bank-query-denom-owners-by-query"></a>

#### DenomOwnersByQuery

The same as [DenomOwners](#bank-query-denom-owners), with the denom in the query string.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/DenomOwnersByQuery` |
| REST | `GET /cosmos/bank/v1beta1/denom_owners_by_query?denom=...` |
| urid | `urid query bank denom-owners-by-query --denom [denom]` |

```proto
message QueryDenomOwnersByQueryRequest {
  string denom = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryDenomOwnersByQueryResponse {
  repeated DenomOwner denom_owners = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| denom_owners | list of `DenomOwner` | In the response. Each holder's `address` and `balance`. |

<a id="bank-query-denom-metadata"></a>

#### DenomMetadata

Returns the display information of one denom. The denom goes into the path as it is, slashes included.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/DenomMetadata` |
| REST | `GET /cosmos/bank/v1beta1/denoms_metadata/{denom}` |
| urid | `urid query bank denom-metadata [denom]` |

```proto
message QueryDenomMetadataRequest {
  string denom = 1;
}

message QueryDenomMetadataResponse {
  Metadata metadata = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| metadata | [`Metadata`](#bank-type-metadata) | In the response. The display information. |

<a id="bank-query-denom-metadata-by-query-string"></a>

#### DenomMetadataByQueryString

The same as [DenomMetadata](#bank-query-denom-metadata), with the denom in the query string.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/DenomMetadataByQueryString` |
| REST | `GET /cosmos/bank/v1beta1/denoms_metadata_by_query_string?denom=...` |
| urid | `urid query bank denom-metadata-by-query-string --denom [denom]` |

```proto
message QueryDenomMetadataByQueryStringRequest {
  string denom = 1;
}

message QueryDenomMetadataByQueryStringResponse {
  Metadata metadata = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | The denom. |
| metadata | [`Metadata`](#bank-type-metadata) | In the response. The display information. |

<a id="bank-query-denoms-metadata"></a>

#### DenomsMetadata

Lists the display information of every denom that has it.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/DenomsMetadata` |
| REST | `GET /cosmos/bank/v1beta1/denoms_metadata` |
| urid | `urid query bank denoms-metadata` |

```proto
message QueryDenomsMetadataRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryDenomsMetadataResponse {
  repeated Metadata metadatas = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| metadatas | list of [`Metadata`](#bank-type-metadata) | In the response. The display information. |

<a id="bank-query-send-enabled"></a>

#### SendEnabled

Returns the denoms that have their own send switch. A denom with no entry follows `default_send_enabled` and is not listed.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/SendEnabled` |
| REST | `GET /cosmos/bank/v1beta1/send_enabled` |
| urid | `urid query bank send-enabled [denom1 ...]` |

```proto
message QuerySendEnabledRequest {
  repeated string denoms = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 99;
}

message QuerySendEnabledResponse {
  repeated SendEnabled send_enabled = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 99;
}
```
| Field | Type | Meaning |
|---|---|---|
| denoms | list of strings | The denoms to look up. Leave empty for every entry. |
| pagination | page request | Used only when `denoms` is empty. Paged, see [Reading lists](#connect-conventions). |
| send_enabled | list of [`SendEnabled`](#bank-type-send-enabled) | In the response. The entries found. |

<a id="bank-query-params"></a>

#### Params

Returns the bank parameters.

| | |
|---|---|
| gRPC | `cosmos.bank.v1beta1.Query/Params` |
| REST | `GET /cosmos/bank/v1beta1/params` |
| urid | `urid query bank params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#bank-parameters) | In the response. The parameters. |

<a id="bank-parameters"></a>

### Parameters

The bank parameters are chain-wide settings. Governance changes them through [MsgUpdateParams](#bank-msg-update-params) inside a proposal. This manual gives no value for `default_send_enabled`. Read it with the [Params](#bank-query-params) query.

```proto
message Params {
  repeated SendEnabled send_enabled = 1;
  bool default_send_enabled = 2;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| default_send_enabled | Whether a denom with no send entry of its own can be sent. | read it with the Params query |
| send_enabled | An old field kept for compatibility. Always empty. Per-denom switches are set with [MsgSetSendEnabled](#bank-msg-set-send-enabled). | empty |

<a id="std-feegrant"></a>

## Fee grants

<a id="feegrant-overview"></a>

### What it is

A fee grant lets one account pay transaction fees for another. The account that pays is the granter. The account whose fees are covered is the grantee. The shop can grant its customers a fee allowance so they can use its service without holding URI.

The grantee names the granter in the transaction's fee. The fee then comes out of the granter's balance, within the allowance's limits. Fees are paid in kwak only, so give spend limits in kwak.

UriNet adds two rules. A recovery-locked account cannot be a fee granter or a fee payer. If a transaction uses a delegation and names the delegator as fee granter, the delegation pays the fee within its own cap, not a fee grant. See [Delegations](#ch-delegation).

<a id="feegrant-type-basic-allowance"></a>

#### BasicAllowance

A total limit with an optional end time.

```proto
message BasicAllowance {
  repeated cosmos.base.v1beta1.Coin spend_limit = 1;
  google.protobuf.Timestamp expiration = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| spend_limit | list of coins | The most the grantee may spend in fees, reduced as it is used. Empty means no limit. |
| expiration | timestamp | When the allowance ends. Empty means it never expires. |

<a id="feegrant-type-periodic-allowance"></a>

#### PeriodicAllowance

A total limit plus a limit that resets every period.

```proto
message PeriodicAllowance {
  BasicAllowance basic = 1;
  google.protobuf.Duration period = 2;
  repeated cosmos.base.v1beta1.Coin period_spend_limit = 3;
  repeated cosmos.base.v1beta1.Coin period_can_spend = 4;
  google.protobuf.Timestamp period_reset = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| basic | [`BasicAllowance`](#feegrant-type-basic-allowance) | The overall limit and end time. |
| period | duration | The length of each period. |
| period_spend_limit | list of coins | The most the grantee may spend in one period. |
| period_can_spend | list of coins | What is left in the current period. |
| period_reset | timestamp | When the current period ends and a new one starts. |

<a id="feegrant-type-allowed-msg-allowance"></a>

#### AllowedMsgAllowance

Wraps another allowance and limits it to some message types.

```proto
message AllowedMsgAllowance {
  google.protobuf.Any allowance = 1;
  repeated string allowed_messages = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| allowance | allowance | A `BasicAllowance` or a `PeriodicAllowance`, as a JSON object with its `@type`. |
| allowed_messages | list of strings | The message type URLs the grant pays for, such as `/cosmos.bank.v1beta1.MsgSend`. A transaction with any other message cannot use it. |

<a id="feegrant-type-grant"></a>

#### Grant

A stored grant, as the queries return it.

```proto
message Grant {
  string granter = 1;
  string grantee = 2;
  google.protobuf.Any allowance = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| granter | address | Who pays. |
| grantee | address | Whose fees are covered. |
| allowance | allowance | One of the three allowance types, with its `@type`. |

<a id="feegrant-transactions"></a>

### Transactions

The granter signs both messages.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgGrantAllowance](#feegrant-msg-grant-allowance) | Gives an account a fee allowance. | The granter |
| [MsgRevokeAllowance](#feegrant-msg-revoke-allowance) | Takes an allowance away. | The granter |

<a id="feegrant-msg-grant-allowance"></a>

#### MsgGrantAllowance

This gives the grantee an allowance on your balance for fees. A granter can have only one allowance per grantee. To change one, revoke it and grant again.

| | |
|---|---|
| Signer | `granter`, the account that will pay |
| Before you send it | You have no allowance to this grantee yet. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx feegrant grant [granter_key_or_address] [grantee]` |

```proto
message MsgGrantAllowance {
  string granter = 1;
  string grantee = 2;
  google.protobuf.Any allowance = 3;
}

message MsgGrantAllowanceResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| granter | address | Who pays. Must sign. |
| grantee | address | Whose fees are covered. |
| allowance | allowance | One of [`BasicAllowance`](#feegrant-type-basic-allowance), [`PeriodicAllowance`](#feegrant-type-periodic-allowance), or [`AllowedMsgAllowance`](#feegrant-type-allowed-msg-allowance), with its `@type`. |

The command builds the allowance from flags: `--spend-limit`, `--expiration` as an RFC 3339 time, `--period` in seconds with `--period-limit`, and `--allowed-messages`. The shop grants a customer up to 1 URI in fees until 31 January 2027. It composes the grant like this, then signs the file with `urid tx sign`.

```sh
urid tx feegrant grant <shop-address> <customer-address> --spend-limit 1000000000kwak \
  --expiration 2027-01-31T00:00:00Z --generate-only > grant.json
```

**Refused when**

- The granter and the grantee are the same account.
- An allowance from this granter to this grantee already exists.
- The expiration is already past.
- A spend limit holds a zero or invalid amount.
- A periodic allowance's period limit uses a denom that its total limit does not have.
- An allowed-message allowance lists no messages.
- The granter is recovery-locked.

<a id="feegrant-msg-revoke-allowance"></a>

#### MsgRevokeAllowance

This removes your allowance to a grantee. The grantee can no longer use your balance for fees.

| | |
|---|---|
| Signer | `granter` |
| Before you send it | An allowance from you to this grantee exists. |
| What comes back | nothing; the transaction result tells you it went through |
| urid | `urid tx feegrant revoke [granter] [grantee]` |

```proto
message MsgRevokeAllowance {
  string granter = 1;
  string grantee = 2;
}

message MsgRevokeAllowanceResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| granter | address | Who pays. Must sign. |
| grantee | address | The account losing the allowance. |

**Refused when**

- No allowance from this granter to this grantee exists.
- The granter and the grantee are the same account.
- The granter is recovery-locked.

`MsgPruneAllowances`, the standard message that clears expired allowances, is refused on UriNet before the fee is taken. `urid tx feegrant prune` therefore does not work.

<a id="feegrant-queries"></a>

### Queries

These queries read allowances by pair, by grantee, or by granter.

| Query | What it answers |
|---|---|
| [Allowance](#feegrant-query-allowance) | The allowance from one granter to one grantee. |
| [Allowances](#feegrant-query-allowances) | Every allowance a grantee has. |
| [AllowancesByGranter](#feegrant-query-allowances-by-granter) | Every allowance a granter has given. |

<a id="feegrant-query-allowance"></a>

#### Allowance

Returns the allowance between one granter and one grantee, with what is left of it.

| | |
|---|---|
| gRPC | `cosmos.feegrant.v1beta1.Query/Allowance` |
| REST | `GET /cosmos/feegrant/v1beta1/allowance/{granter}/{grantee}` |
| urid | `urid query feegrant grant [granter] [grantee]` |

```proto
message QueryAllowanceRequest {
  string granter = 1;
  string grantee = 2;
}

message QueryAllowanceResponse {
  cosmos.feegrant.v1beta1.Grant allowance = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| granter | address | Who pays. |
| grantee | address | Whose fees are covered. |
| allowance | [`Grant`](#feegrant-type-grant) | In the response. The grant. |

<a id="feegrant-query-allowances"></a>

#### Allowances

Lists every allowance a grantee holds. A wallet can ask it to offer the user a fee granter.

| | |
|---|---|
| gRPC | `cosmos.feegrant.v1beta1.Query/Allowances` |
| REST | `GET /cosmos/feegrant/v1beta1/allowances/{grantee}` |
| urid | `urid query feegrant grants-by-grantee [grantee]` |

```proto
message QueryAllowancesRequest {
  string grantee = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryAllowancesResponse {
  repeated cosmos.feegrant.v1beta1.Grant allowances = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| grantee | address | The grantee. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| allowances | list of [`Grant`](#feegrant-type-grant) | In the response. The grants. |

<a id="feegrant-query-allowances-by-granter"></a>

#### AllowancesByGranter

Lists every allowance a granter has given. The shop can ask it to review what it pays for.

| | |
|---|---|
| gRPC | `cosmos.feegrant.v1beta1.Query/AllowancesByGranter` |
| REST | `GET /cosmos/feegrant/v1beta1/issued/{granter}` |
| urid | `urid query feegrant grants-by-granter [granter]` |

```proto
message QueryAllowancesByGranterRequest {
  string granter = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryAllowancesByGranterResponse {
  repeated cosmos.feegrant.v1beta1.Grant allowances = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| granter | address | The granter. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| allowances | list of [`Grant`](#feegrant-type-grant) | In the response. The grants. |

<a id="std-auth"></a>

## Accounts and auth

<a id="auth-overview"></a>

### What it is

The auth module keeps the basic record of every account: its address, its account number, and its sequence. The sequence counts the transactions the account has sent and goes into each signature. Wallets read it before signing.

On UriNet the account record these queries return is UriNet's own vesting-aware account type, `/urinet.vesting.v1.UriVestingAccount`. It holds the base record and any vesting schedules. See [Vesting](#ch-vesting). Module accounts, which belong to the chain's modules, come back as `/cosmos.auth.v1beta1.ModuleAccount`.

The base record's `pub_key` is always empty. An account's keys are the credentials of its SmartAccount, which you read from the SmartAccount module. See [SmartAccounts](#ch-smartaccount).

<a id="auth-transactions"></a>

### Transactions

The auth module has one message, and only governance uses it.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgUpdateParams](#auth-msg-update-params) | Replaces the auth parameters. | The governance account, through a passed proposal |

<a id="auth-msg-update-params"></a>

#### MsgUpdateParams

This replaces the auth parameters. You must give the whole parameter set. A parameter left out becomes zero, and the chain refuses a zero limit or cost.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | the governance account, through a passed proposal |
| Before you send it | The proposal carries the full new parameter set. |
| What comes back | nothing; the proposal's status tells you it went through |
| urid | `urid tx auth update-params-proposal [params]`, which submits a normal proposal carrying it |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#auth-parameters) | The full new parameter set. |

**Refused when**

- The authority is not the governance account.
- A parameter is zero. Every auth parameter must be above zero.

<a id="auth-queries"></a>

### Queries

These queries read account records and convert addresses.

| Query | What it answers |
|---|---|
| [Account](#auth-query-account) | One account's full record. |
| [AccountInfo](#auth-query-account-info) | One account's base record. |
| [Accounts](#auth-query-accounts) | Every account. |
| [AccountAddressByID](#auth-query-account-address-by-id) | The address for an account number. |
| [ModuleAccounts](#auth-query-module-accounts) | Every module account. |
| [ModuleAccountByName](#auth-query-module-account-by-name) | One module account by name. |
| [Bech32Prefix](#auth-query-bech32-prefix) | The address prefix. |
| [AddressBytesToString](#auth-query-address-bytes-to-string) | An address from raw bytes. |
| [AddressStringToBytes](#auth-query-address-string-to-bytes) | Raw bytes from an address. |
| [Params](#auth-query-params) | The auth parameters. |

<a id="auth-query-account"></a>

#### Account

Returns an account's full record, as a `UriVestingAccount` or, for a module, a `ModuleAccount`. Ask it for the account number and sequence before you sign offline.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/Account` |
| REST | `GET /cosmos/auth/v1beta1/accounts/{address}` |
| urid | `urid query auth account [address]` |

```proto
message QueryAccountRequest {
  string address = 1;
}

message QueryAccountResponse {
  google.protobuf.Any account = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account. |
| account | account record | In the response. The record, as a JSON object with its `@type`. |

<a id="auth-query-account-info"></a>

#### AccountInfo

Returns only the base record: address, account number, and sequence. It is the same shape for every account type.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/AccountInfo` |
| REST | `GET /cosmos/auth/v1beta1/account_info/{address}` |
| urid | `urid query auth account-info [address]` |

```proto
message QueryAccountInfoRequest {
  string address = 1;
}

message QueryAccountInfoResponse {
  BaseAccount info = 1;
}
```
```proto
message BaseAccount {
  string address = 1;
  google.protobuf.Any pub_key = 2;
  uint64 account_number = 3;
  uint64 sequence = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The account. |
| info | `BaseAccount` | In the response. The base record. |
| pub_key | key | In `info`. Always empty on UriNet. |
| account_number | number | In `info`. The account's fixed number, which goes into each signature. |
| sequence | number | In `info`. The account's next transaction number. |

<a id="auth-query-accounts"></a>

#### Accounts

Lists every account on the chain.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/Accounts` |
| REST | `GET /cosmos/auth/v1beta1/accounts` |
| urid | `urid query auth accounts` |

```proto
message QueryAccountsRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryAccountsResponse {
  repeated google.protobuf.Any accounts = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| accounts | list of account records | In the response. Each with its `@type`. |

<a id="auth-query-account-address-by-id"></a>

#### AccountAddressByID

Returns the address that has a given account number.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/AccountAddressByID` |
| REST | `GET /cosmos/auth/v1beta1/address_by_id/{account_id}` |
| urid | `urid query auth address-by-acc-num [acc-num]` |

```proto
message QueryAccountAddressByIDRequest {
  int64 id = 1;
  uint64 account_id = 2;
}

message QueryAccountAddressByIDResponse {
  string account_address = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| account_id | number | The account number. |
| id | number | An older form of `account_id`. Use `account_id`. |
| account_address | address | In the response. The address. |

<a id="auth-query-module-accounts"></a>

#### ModuleAccounts

Lists the accounts that belong to the chain's modules, such as the governance account.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/ModuleAccounts` |
| REST | `GET /cosmos/auth/v1beta1/module_accounts` |
| urid | `urid query auth module-accounts` |

```proto
message QueryModuleAccountsRequest {
}

message QueryModuleAccountsResponse {
  repeated google.protobuf.Any accounts = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| accounts | list of account records | In the response. Each module account, with its name and permissions. |

<a id="auth-query-module-account-by-name"></a>

#### ModuleAccountByName

Returns one module account by its module name. `urid query auth module-account gov` gives the governance account's address, which every proposal message names as its authority.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/ModuleAccountByName` |
| REST | `GET /cosmos/auth/v1beta1/module_accounts/{name}` |
| urid | `urid query auth module-account [module-name]` |

```proto
message QueryModuleAccountByNameRequest {
  string name = 1;
}

message QueryModuleAccountByNameResponse {
  google.protobuf.Any account = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| name | string | The module name, such as `gov`. |
| account | account record | In the response. The module account. |

<a id="auth-query-bech32-prefix"></a>

#### Bech32Prefix

Returns the prefix of account addresses, `uri`.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/Bech32Prefix` |
| REST | `GET /cosmos/auth/v1beta1/bech32` |
| urid | `urid query auth bech32-prefix` |

```proto
message Bech32PrefixRequest {
}

message Bech32PrefixResponse {
  string bech32_prefix = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| bech32_prefix | string | In the response. The prefix. |

<a id="auth-query-address-bytes-to-string"></a>

#### AddressBytesToString

Turns raw address bytes into a `uri1` address.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/AddressBytesToString` |
| REST | `GET /cosmos/auth/v1beta1/bech32/{address_bytes}` |
| urid | `urid query auth address-bytes-to-string [address-bytes]` |

```proto
message AddressBytesToStringRequest {
  bytes address_bytes = 1;
}

message AddressBytesToStringResponse {
  string address_string = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address_bytes | bytes, base64 in JSON | The raw address. |
| address_string | string | In the response. The `uri1` address. |

<a id="auth-query-address-string-to-bytes"></a>

#### AddressStringToBytes

Turns a `uri1` address into its raw bytes.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/AddressStringToBytes` |
| REST | `GET /cosmos/auth/v1beta1/bech32/{address_string}` |
| urid | `urid query auth address-string-to-bytes [address-string]` |

```proto
message AddressStringToBytesRequest {
  string address_string = 1;
}

message AddressStringToBytesResponse {
  bytes address_bytes = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address_string | string | The `uri1` address. |
| address_bytes | bytes, base64 in JSON | In the response. The raw address. |

<a id="auth-query-params"></a>

#### Params

Returns the auth parameters.

| | |
|---|---|
| gRPC | `cosmos.auth.v1beta1.Query/Params` |
| REST | `GET /cosmos/auth/v1beta1/params` |
| urid | `urid query auth params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#auth-parameters) | In the response. The parameters. |

<a id="auth-parameters"></a>

### Parameters

The auth parameters set transaction limits and gas costs. Governance changes them through [MsgUpdateParams](#auth-msg-update-params) inside a proposal. This manual gives no values for them. Read them with the [Params](#auth-query-params) query.

```proto
message Params {
  uint64 max_memo_characters = 1;
  uint64 tx_sig_limit = 2;
  uint64 tx_size_cost_per_byte = 3;
  uint64 sig_verify_cost_ed25519 = 4;
  uint64 sig_verify_cost_secp256k1 = 5;
  uint64 sig_verify_cost_mldsa65 = 6;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| max_memo_characters | The longest memo a transaction may carry. | read it with the Params query |
| tx_sig_limit | A signature limit UriNet does not use. The cap on proofs per transaction is the SmartAccount parameter `max_auth_proofs_per_tx`. See [SmartAccount parameters](#smartaccount-parameters). | read it with the Params query |
| tx_size_cost_per_byte | The gas charged for each byte of a transaction. | read it with the Params query |
| sig_verify_cost_secp256k1 | The gas charged to check one secp256k1 credential proof. A P-256 or WebAuthn proof costs half of it. | read it with the Params query |
| sig_verify_cost_ed25519 | Gas for an Ed25519 signature. UriNet credentials use secp256k1 or P-256 only, so account proofs never use it. | read it with the Params query |
| sig_verify_cost_mldsa65 | Gas for an ML-DSA-65 signature. Account proofs never use it either. | read it with the Params query |

---

<a id="std-staking"></a>

## Staking

<a id="staking-overview"></a>

### What it is

Staking is how URI secures the chain. Validators are the nodes that sign blocks. You stake URI by bonding it to a validator. Your stake adds to that validator's voting power and earns you a share of its rewards.

> **Note.** In this module, "delegation" means staking your URI with a validator. It has nothing to do with UriNet's delegation of authority, where one account lets another act on its assets. That feature is described in [Delegation](#ch-delegation). A delegation of authority can never stake, unstake, or run a validator for you.

Only one coin can be staked: URI, counted in its base unit kwak. This is the bond denom, and it can never change. User tokens cannot be staked. URI still locked by a vesting schedule can be staked. Staking it does not unlock it any earlier.

The words to know:

- **Validator.** A node that signs blocks. Its operator is an ordinary UriNet account. The operator address is that same account written with the `urivaloper1` prefix. `urid debug addr [address]` prints both forms.
- **Delegator.** An account that has staked with a validator. The operator usually stakes with its own validator too, which is called self-delegation.
- **Shares.** What a delegator holds in a validator. Shares are worth URI at the validator's current rate. The rate falls if the validator is slashed, which means penalized for misbehaviour.
- **Unbonding.** Taking stake back. The coins return after the unbonding period of 21 days. During that time they earn nothing, and they can still be cut for an offence the validator committed while they were staked.
- **Redelegation.** Moving stake from one validator to another at once, without waiting out the unbonding period.
- **Commission.** The fraction of a validator's rewards that its operator keeps. The rest goes to its delegators.
- **Jailed.** Removed from block signing for misbehaviour. See [Slashing](#std-slashing).

Only the validators with the most stake sign blocks, up to the `max_validators` parameter. They are bonded. A validator that drops out of that set is unbonding for 21 days and then unbonded. Rewards are paid through [Distribution](#std-distribution). Staked URI also counts as voting power in [Governance](#ch-gov).

Every message here is signed by a SmartAccount with a signature bundle, like any other UriNet transaction. You compose it with the module command and `--generate-only`, then sign it with `urid tx sign`, as [Sending transactions with urid](#urid-tx) shows. Validator operations are signed by the operator's account. While an account is locked for a recovery, it cannot sign any of them.

<a id="staking-type-validator"></a>

#### Validator

A validator as the queries return it.

```proto
message Validator {
  string operator_address = 1;
  google.protobuf.Any consensus_pubkey = 2;
  bool jailed = 3;
  BondStatus status = 4;
  string tokens = 5;
  string delegator_shares = 6;
  Description description = 7;
  int64 unbonding_height = 8;
  google.protobuf.Timestamp unbonding_time = 9;
  Commission commission = 10;
  string min_self_delegation = 11;
  int64 unbonding_on_hold_ref_count = 12;
  repeated uint64 unbonding_ids = 13;
}

enum BondStatus {
  BOND_STATUS_UNSPECIFIED = 0;
  BOND_STATUS_UNBONDED = 1;
  BOND_STATUS_UNBONDING = 2;
  BOND_STATUS_BONDED = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| operator_address | address | The operator address, `urivaloper1...`. |
| consensus_pubkey | public key, `@type` JSON | The key the validator's node signs blocks with. It is not the operator's account key. |
| jailed | boolean | True while the validator is jailed. |
| status | `BondStatus` | `BOND_STATUS_BONDED`: in the active set and signing blocks. `BOND_STATUS_UNBONDING`: left the set and waiting out the unbonding period. `BOND_STATUS_UNBONDED`: fully out. `BOND_STATUS_UNSPECIFIED` is never a real status. |
| tokens | integer as a string | All kwak staked with the validator, self-delegation included. |
| delegator_shares | decimal as a string | The total shares issued to its delegators. |
| description | [`Description`](#staking-type-description) | Its name and contact details. |
| unbonding_height | number | If unbonding, the block height where it began. |
| unbonding_time | timestamp | If unbonding, the earliest time it completes. |
| commission | [`Commission`](#staking-type-commission) | Its commission rates and the time of the last change. |
| min_self_delegation | integer as a string | The least kwak the operator promises to keep staked. If the operator unbonds below it, the validator is jailed. |
| unbonding_on_hold_ref_count | number | Counts holds that other modules place on its unbonding. Nothing on UriNet places such holds, so it reads 0. |
| unbonding_ids | list of numbers | Ids of its unbonding operations. |

<a id="staking-type-description"></a>

#### Description

The public face of a validator. Delegators read it to choose whom to stake with.

```proto
message Description {
  string moniker = 1;
  string identity = 2;
  string website = 3;
  string security_contact = 4;
  string details = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| moniker | string | The validator's display name. |
| identity | string | Optional. An identity signature from an outside identity service. |
| website | string | Optional. A website link. |
| security_contact | string | Optional. An email address for security reports. |
| details | string | Optional. Free text. |

<a id="staking-type-commission"></a>

#### Commission and CommissionRates

The commission a validator charges. `max_rate` and `max_change_rate` are set when the validator is created and never change. Only `rate` can be changed later.

```proto
message Commission {
  CommissionRates commission_rates = 1;
  google.protobuf.Timestamp update_time = 2;
}

message CommissionRates {
  string rate = 1;
  string max_rate = 2;
  string max_change_rate = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| commission_rates | `CommissionRates` | The three rates below. |
| update_time | timestamp | When the rate last changed. |
| rate | decimal as a string | The current commission, as a fraction. `0.05` means 5 percent. |
| max_rate | decimal as a string | The highest rate this validator can ever charge. |
| max_change_rate | decimal as a string | The most the rate can rise in one day. |

<a id="staking-type-delegation"></a>

#### Delegation and DelegationResponse

One delegator's stake with one validator. The queries return it as a `DelegationResponse`, which adds what the shares are worth.

```proto
message Delegation {
  string delegator_address = 1;
  string validator_address = 2;
  string shares = 3;
}

message DelegationResponse {
  Delegation delegation = 1;
  cosmos.base.v1beta1.Coin balance = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator, `uri1...`. |
| validator_address | address | The validator's operator address, `urivaloper1...`. |
| shares | decimal as a string | The shares the delegator holds. |
| delegation | `Delegation` | The record above. |
| balance | coin | What the shares are worth now, in kwak. |

<a id="staking-type-unbonding-delegation"></a>

#### UnbondingDelegation and UnbondingDelegationEntry

All of one delegator's unbondings from one validator. Each MsgUndelegate adds an entry, and each entry completes on its own.

```proto
message UnbondingDelegation {
  string delegator_address = 1;
  string validator_address = 2;
  repeated UnbondingDelegationEntry entries = 3;
}

message UnbondingDelegationEntry {
  int64 creation_height = 1;
  google.protobuf.Timestamp completion_time = 2;
  string initial_balance = 3;
  string balance = 4;
  uint64 unbonding_id = 5;
  int64 unbonding_on_hold_ref_count = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| validator_address | address | The validator's operator address. |
| entries | list of `UnbondingDelegationEntry` | One per unbonding, oldest first. |
| creation_height | number | The block height where the unbonding started. MsgCancelUnbondingDelegation names an entry by it. |
| completion_time | timestamp | When the coins return to the delegator. |
| initial_balance | integer as a string | The kwak scheduled when the unbonding started. |
| balance | integer as a string | The kwak that will return. It is lower than `initial_balance` if the validator was slashed meanwhile. |
| unbonding_id | number | An id for this entry. |
| unbonding_on_hold_ref_count | number | Holds from other modules. Nothing on UriNet places them, so it reads 0. |

<a id="staking-type-redelegation"></a>

#### Redelegation and RedelegationEntry

All of one delegator's redelegations from one validator to another. Each MsgBeginRedelegate adds an entry.

```proto
message Redelegation {
  string delegator_address = 1;
  string validator_src_address = 2;
  string validator_dst_address = 3;
  repeated RedelegationEntry entries = 4;
}

message RedelegationEntry {
  int64 creation_height = 1;
  google.protobuf.Timestamp completion_time = 2;
  string initial_balance = 3;
  string shares_dst = 4;
  uint64 unbonding_id = 5;
  int64 unbonding_on_hold_ref_count = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| validator_src_address | address | The validator the stake moved from. |
| validator_dst_address | address | The validator the stake moved to. |
| entries | list of `RedelegationEntry` | One per redelegation. |
| creation_height | number | The block height where the redelegation happened. |
| completion_time | timestamp | When the entry completes. Until then, the stake can still be cut for an earlier offence of the source validator. |
| initial_balance | integer as a string | The kwak moved. |
| shares_dst | decimal as a string | The shares created at the destination. |
| unbonding_id | number | An id for this entry. |
| unbonding_on_hold_ref_count | number | Holds from other modules. Nothing on UriNet places them, so it reads 0. |

<a id="staking-transactions"></a>

### Transactions

Operators run validators with the first three messages, and delegators manage their stake with the next four.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgCreateValidator](#staking-msg-create-validator) | Registers a validator with its first self-delegation | The operator |
| [MsgEditValidator](#staking-msg-edit-validator) | Changes the description, commission rate, or minimum self-delegation | The operator |
| [MsgRotateConsPubKey](#staking-msg-rotate-cons-pub-key) | Replaces the key the validator signs blocks with | The operator |
| [MsgDelegate](#staking-msg-delegate) | Stakes URI with a validator | The delegator |
| [MsgBeginRedelegate](#staking-msg-begin-redelegate) | Moves stake to another validator at once | The delegator |
| [MsgUndelegate](#staking-msg-undelegate) | Starts taking stake back | The delegator |
| [MsgCancelUnbondingDelegation](#staking-msg-cancel-unbonding-delegation) | Stops an unbonding and stakes the coins again | The delegator |
| [MsgUpdateParams](#staking-msg-update-params) | Changes the staking parameters | Governance |

<a id="staking-msg-create-validator"></a>

#### MsgCreateValidator

Registers a new validator and stakes its first self-delegation in the same step. The operator sends it from the account that will run the validator. The validator joins the active set if its stake ranks within `max_validators`.

| | |
|---|---|
| Signer | `validator_address`, the operator's account in its `urivaloper1` form |
| Before you send it | The node runs and you have its consensus public key, which `urid comet show-validator` prints on the node. The operator's account holds the self-delegation and the fee, in kwak. The commission rate is at least the `min_commission_rate` parameter. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx staking create-validator [path/to/validator.json]` |

The JSON file holds the validator's details. The amounts below are examples. `amount` becomes `value` in the message, and the commission keys fill `commission`.

```json
{
  "pubkey": {"@type": "/cosmos.crypto.ed25519.PubKey", "key": "..."},
  "amount": "1000000000000kwak",
  "moniker": "shop-validator",
  "identity": "",
  "website": "https://example.com",
  "security": "security@example.com",
  "details": "",
  "commission-rate": "0.10",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

```proto
message MsgCreateValidator {
  Description description = 1;
  CommissionRates commission = 2;
  string min_self_delegation = 3;
  string delegator_address = 4;
  string validator_address = 5;
  google.protobuf.Any pubkey = 6;
  cosmos.base.v1beta1.Coin value = 7;
}

message MsgCreateValidatorResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| description | [`Description`](#staking-type-description) | The name and contact details. It must not be empty. |
| commission | [`CommissionRates`](#staking-type-commission) | The starting rate, the maximum rate, and the maximum daily rise. |
| min_self_delegation | integer as a string | The least kwak the operator promises to keep staked. Must be positive. |
| delegator_address | address | No longer used. Leave it empty. |
| validator_address | address | The operator address. |
| pubkey | public key, `@type` JSON | The node's consensus public key. |
| value | coin | The first self-delegation, in kwak. |

**Refused when**

- The operator already runs a validator, or another validator already uses this consensus key.
- The consensus key is of a type the chain does not accept.
- The rate is above `max_rate`, `max_change_rate` is above `max_rate`, or the rate is below `min_commission_rate`.
- `value` is below `min_self_delegation`, or it is not in kwak.
- The description is empty, or a field is too long.
- The account does not hold enough URI for the self-delegation.

<a id="staking-msg-edit-validator"></a>

#### MsgEditValidator

Changes a validator's description, its commission rate, or its minimum self-delegation. Fields you leave out keep their current value. In `urid`, each description flag defaults to `[do-not-modify]`.

| | |
|---|---|
| Signer | `validator_address`, the operator's account in its `urivaloper1` form |
| Before you send it | The validator exists. The commission rate has not changed in the last 24 hours. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx staking edit-validator` with `--new-moniker`, `--identity`, `--website`, `--security-contact`, `--details`, `--commission-rate`, `--min-self-delegation` |

The help text calls `--commission-rate` a percentage, but it is a fraction: `0.05` means 5 percent.

```proto
message MsgEditValidator {
  Description description = 1;
  string validator_address = 2;
  string commission_rate = 3;
  string min_self_delegation = 4;
}

message MsgEditValidatorResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| description | [`Description`](#staking-type-description) | The new details. Fields set to `[do-not-modify]` stay as they are. |
| validator_address | address | The operator address. |
| commission_rate | decimal as a string | Optional. The new rate, as a fraction. |
| min_self_delegation | integer as a string | Optional. The new minimum self-delegation. |

**Refused when**

- No validator exists for this operator.
- The rate already changed in the last 24 hours.
- The new rate is above `max_rate`, below `min_commission_rate`, or rises by more than `max_change_rate`.
- The new minimum self-delegation is not higher than the current one. It can only rise.
- The new minimum self-delegation is higher than all the kwak staked with the validator.

<a id="staking-msg-rotate-cons-pub-key"></a>

#### MsgRotateConsPubKey

Replaces the key the validator's node signs blocks with. An operator does this when the old key may be exposed, or when the node moves to new hardware. The operator account and the stake stay as they are. The chain charges the `key_rotation_fee` parameter, in kwak, and burns it. A validator can rotate its key once per unbonding period.

| | |
|---|---|
| Signer | `validator_address`, the operator's account in its `urivaloper1` form |
| Before you send it | The new key is ready on the node. The validator is not jailed. The operator's account can pay the rotation fee and the transaction fee. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx staking rotate-cons-pub-key [new-pubkey]`, with the key as `@type` JSON |

```proto
message MsgRotateConsPubKey {
  string validator_address = 1;
  google.protobuf.Any new_pubkey = 2;
}

message MsgRotateConsPubKeyResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| new_pubkey | public key, `@type` JSON | The new consensus public key. |

**Refused when**

- No validator exists for this operator, or the validator is jailed.
- Another validator already uses the new key, or a pending rotation already targets it.
- The key was rotated away from within the last unbonding period.
- The validator already rotated its key within the last unbonding period.
- The key is of a type the chain does not accept.
- The account cannot pay the rotation fee.

<a id="staking-msg-delegate"></a>

#### MsgDelegate

Stakes URI with a validator. Alice stakes 100 URI with a validator she trusts, and from then on she earns a share of its rewards.

| | |
|---|---|
| Signer | `delegator_address`, the account that stakes |
| Before you send it | The validator exists. You hold the amount, in kwak. URI locked by vesting counts. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx staking delegate [validator-addr] [amount]`, for example `100000000000kwak` for 100 URI |

```proto
message MsgDelegate {
  string delegator_address = 1;
  string validator_address = 2;
  cosmos.base.v1beta1.Coin amount = 3;
}

message MsgDelegateResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account that stakes. |
| validator_address | address | The validator's operator address. |
| amount | coin | How much to stake, in kwak. |

If you already stake with this validator, the rewards earned so far are paid out to your withdraw address first. See [Distribution](#std-distribution).

**Refused when**

- No validator has this operator address.
- The amount is zero or not in kwak.
- The account does not hold that much URI.
- The validator's shares have lost all value after a slash, so no new stake can be priced.

<a id="staking-msg-begin-redelegate"></a>

#### MsgBeginRedelegate

Moves stake from one validator to another without waiting. The coins stay bonded the whole time. The move is recorded as a redelegation entry that completes after the unbonding period. Until then, the moved stake can still be cut for an earlier offence of the source validator.

| | |
|---|---|
| Signer | `delegator_address`, the account that owns the stake |
| Before you send it | You stake at least the amount with the source validator. None of that stake arrived there by a redelegation that has not completed yet. |
| What comes back | `completion_time`, when the redelegation entry completes |
| urid | `urid tx staking redelegate [src-validator-addr] [dst-validator-addr] [amount]` |

```proto
message MsgBeginRedelegate {
  string delegator_address = 1;
  string validator_src_address = 2;
  string validator_dst_address = 3;
  cosmos.base.v1beta1.Coin amount = 4;
}

message MsgBeginRedelegateResponse {
  google.protobuf.Timestamp completion_time = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account that owns the stake. |
| validator_src_address | address | The validator to move from. |
| validator_dst_address | address | The validator to move to. |
| amount | coin | How much to move, in kwak. |
| completion_time | timestamp | In the response. When the entry completes. |

**Refused when**

- The source and the destination are the same validator.
- Either validator does not exist.
- You do not stake that much with the source.
- Stake at the source arrived by a redelegation that has not completed. Wait for it to complete first.
- You already have the most entries allowed for this pair of validators, the `max_entries` parameter.
- The amount is too small to be worth a single share.

<a id="staking-msg-undelegate"></a>

#### MsgUndelegate

Starts taking stake back. The coins leave the validator at once and return to your account when the unbonding period of 21 days ends. During those days they earn no rewards and you cannot spend them. Bob unbonds 50 URI today, and the 50 URI arrive in his account 21 days later.

If the operator unbonds its own stake below `min_self_delegation`, the validator is jailed.

| | |
|---|---|
| Signer | `delegator_address`, the account that owns the stake |
| Before you send it | You stake at least the amount with this validator. |
| What comes back | `completion_time` and `amount`, when the coins return and how many |
| urid | `urid tx staking unbond [validator-addr] [amount]` |

```proto
message MsgUndelegate {
  string delegator_address = 1;
  string validator_address = 2;
  cosmos.base.v1beta1.Coin amount = 3;
}

message MsgUndelegateResponse {
  google.protobuf.Timestamp completion_time = 1;
  cosmos.base.v1beta1.Coin amount = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account that owns the stake. |
| validator_address | address | The validator to unbond from. |
| amount | coin | How much to unbond, in kwak. |
| completion_time | timestamp | In the response. When the coins return. |
| amount (response) | coin | In the response. The kwak that is unbonding. |

**Refused when**

- You have no stake with this validator, or less than the amount.
- The amount is zero or not in kwak.
- You already have the most unbonding entries allowed with this validator, the `max_entries` parameter.

<a id="staking-msg-cancel-unbonding-delegation"></a>

#### MsgCancelUnbondingDelegation

Stops an unbonding before it ends and stakes the coins with the same validator again. You name the entry by the block height where it started, shown as `creation_height` by the [UnbondingDelegation](#staking-query-unbonding-delegation) query. You can cancel all of the entry or part of it.

| | |
|---|---|
| Signer | `delegator_address`, the account that owns the unbonding |
| Before you send it | The entry has not completed. The validator is not jailed. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx staking cancel-unbond [validator-addr] [amount] [creation-height]` |

```proto
message MsgCancelUnbondingDelegation {
  string delegator_address = 1;
  string validator_address = 2;
  cosmos.base.v1beta1.Coin amount = 3;
  int64 creation_height = 4;
}

message MsgCancelUnbondingDelegationResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account that owns the unbonding. |
| validator_address | address | The validator it is unbonding from. |
| amount | coin | How much to stake again. At most the entry's `balance`. |
| creation_height | number | The block height where the unbonding started. |

**Refused when**

- No unbonding entry started at that height with this validator.
- The amount is zero, not in kwak, or more than the entry's balance.
- The validator is jailed.
- The entry has already completed.

<a id="staking-msg-update-params"></a>

#### MsgUpdateParams

Replaces the staking parameters. The message replaces all of them, so start from the current values. A parameter left out of the file becomes zero. `bond_denom` and the denom of `key_rotation_fee` are kwak and can never change.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values with the [Params](#staking-query-params) query and change only what you mean to. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | Put the message in a JSON proposal file for `urid tx gov submit-proposal [path/to/proposal.json]`. Staking has no `update-params-proposal` command. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#staking-parameters) | The complete new parameters. |

**Refused when**

- The signer is not the governance account.
- `bond_denom` or the `key_rotation_fee` denom differs from the current one. The error is "immutable protocol parameter".
- A value fails the parameter checks, for example a zero unbonding time.

<a id="staking-queries"></a>

### Queries

The staking queries read validators, stakes, unbondings, and redelegations.

Lists in the standard modules are paged in the usual way. `pagination` takes `key`, `offset`, `limit`, `count_total` and `reverse`, and UriNet adds no cap of its own. The cap of 200 entries per page and the refusal of `offset` and `count_total` belong only to UriNet's own list queries. See [Reading lists](#connect-conventions).

| Query | What it answers |
|---|---|
| [Validators](#staking-query-validators) | All validators, optionally of one status |
| [Validator](#staking-query-validator) | One validator |
| [ValidatorDelegations](#staking-query-validator-delegations) | Everyone who stakes with a validator |
| [ValidatorUnbondingDelegations](#staking-query-validator-unbonding-delegations) | Everyone unbonding from a validator |
| [Delegation](#staking-query-delegation) | One account's stake with one validator |
| [UnbondingDelegation](#staking-query-unbonding-delegation) | One account's unbondings from one validator |
| [DelegatorDelegations](#staking-query-delegator-delegations) | All of one account's stakes |
| [DelegatorUnbondingDelegations](#staking-query-delegator-unbonding-delegations) | All of one account's unbondings |
| [Redelegations](#staking-query-redelegations) | One account's redelegations |
| [DelegatorValidators](#staking-query-delegator-validators) | The validators one account stakes with |
| [DelegatorValidator](#staking-query-delegator-validator) | One validator, if the account stakes with it |
| [HistoricalInfo](#staking-query-historical-info) | The block header and validator set at a recent height |
| [Pool](#staking-query-pool) | How much URI is bonded and not bonded |
| [Params](#staking-query-params) | The staking parameters |

<a id="staking-query-validators"></a>

#### Validators

Lists validators. Delegators use it to choose whom to stake with.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Validators` |
| REST | `GET /cosmos/staking/v1beta1/validators` |
| urid | `urid query staking validators`, with `--status` to filter |

```proto
message QueryValidatorsRequest {
  string status = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryValidatorsResponse {
  repeated Validator validators = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| status | string | Optional. One of `BOND_STATUS_BONDED`, `BOND_STATUS_UNBONDING`, `BOND_STATUS_UNBONDED`. On REST, pass it as `?status=BOND_STATUS_BONDED`. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| validators | list of [`Validator`](#staking-type-validator) | The validators on this page. |
| pagination (response) | page response | In the response. `next_key` fetches the next page. |

<a id="staking-query-validator"></a>

#### Validator

Returns one validator by its operator address.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Validator` |
| REST | `GET /cosmos/staking/v1beta1/validators/{validator_addr}` |
| urid | `urid query staking validator [validator-addr]` |

```proto
message QueryValidatorRequest {
  string validator_addr = 1;
}

message QueryValidatorResponse {
  Validator validator = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_addr | address | The operator address, `urivaloper1...`. |
| validator | [`Validator`](#staking-type-validator) | The validator. |

<a id="staking-query-validator-delegations"></a>

#### ValidatorDelegations

Lists every stake with one validator, self-delegation included.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/ValidatorDelegations` |
| REST | `GET /cosmos/staking/v1beta1/validators/{validator_addr}/delegations` |
| urid | `urid query staking delegations-to [validator-addr]` |

```proto
message QueryValidatorDelegationsRequest {
  string validator_addr = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryValidatorDelegationsResponse {
  repeated DelegationResponse delegation_responses = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_addr | address | The operator address. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| delegation_responses | list of [`DelegationResponse`](#staking-type-delegation) | Each stake and its worth in kwak. |

<a id="staking-query-validator-unbonding-delegations"></a>

#### ValidatorUnbondingDelegations

Lists every unbonding from one validator that has not completed.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/ValidatorUnbondingDelegations` |
| REST | `GET /cosmos/staking/v1beta1/validators/{validator_addr}/unbonding_delegations` |
| urid | `urid query staking unbonding-delegations-from [validator-addr]` |

```proto
message QueryValidatorUnbondingDelegationsRequest {
  string validator_addr = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryValidatorUnbondingDelegationsResponse {
  repeated UnbondingDelegation unbonding_responses = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_addr | address | The operator address. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| unbonding_responses | list of [`UnbondingDelegation`](#staking-type-unbonding-delegation) | One record per delegator, each with its entries. |

<a id="staking-query-delegation"></a>

#### Delegation

Returns one account's stake with one validator. It answers "how much does Alice stake with this validator right now".

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Delegation` |
| REST | `GET /cosmos/staking/v1beta1/validators/{validator_addr}/delegations/{delegator_addr}` |
| urid | `urid query staking delegation [delegator-addr] [validator-addr]` |

```proto
message QueryDelegationRequest {
  string delegator_addr = 1;
  string validator_addr = 2;
}

message QueryDelegationResponse {
  DelegationResponse delegation_response = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator, `uri1...`. |
| validator_addr | address | The operator address. |
| delegation_response | [`DelegationResponse`](#staking-type-delegation) | The stake and its worth in kwak. |

<a id="staking-query-unbonding-delegation"></a>

#### UnbondingDelegation

Returns one account's unbondings from one validator. Use it to find the `creation_height` that MsgCancelUnbondingDelegation needs.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/UnbondingDelegation` |
| REST | `GET /cosmos/staking/v1beta1/validators/{validator_addr}/delegations/{delegator_addr}/unbonding_delegation` |
| urid | `urid query staking unbonding-delegation [delegator-addr] [validator-addr]` |

```proto
message QueryUnbondingDelegationRequest {
  string delegator_addr = 1;
  string validator_addr = 2;
}

message QueryUnbondingDelegationResponse {
  UnbondingDelegation unbond = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| validator_addr | address | The operator address. |
| unbond | [`UnbondingDelegation`](#staking-type-unbonding-delegation) | The unbonding record and its entries. |

<a id="staking-query-delegator-delegations"></a>

#### DelegatorDelegations

Lists all of one account's stakes, one per validator.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/DelegatorDelegations` |
| REST | `GET /cosmos/staking/v1beta1/delegations/{delegator_addr}` |
| urid | `urid query staking delegations [delegator-addr]` |

```proto
message QueryDelegatorDelegationsRequest {
  string delegator_addr = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryDelegatorDelegationsResponse {
  repeated DelegationResponse delegation_responses = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| delegation_responses | list of [`DelegationResponse`](#staking-type-delegation) | Each stake and its worth in kwak. |

<a id="staking-query-delegator-unbonding-delegations"></a>

#### DelegatorUnbondingDelegations

Lists all of one account's unbondings that have not completed, across validators.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/DelegatorUnbondingDelegations` |
| REST | `GET /cosmos/staking/v1beta1/delegators/{delegator_addr}/unbonding_delegations` |
| urid | `urid query staking unbonding-delegations [delegator-addr]` |

```proto
message QueryDelegatorUnbondingDelegationsRequest {
  string delegator_addr = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryDelegatorUnbondingDelegationsResponse {
  repeated UnbondingDelegation unbonding_responses = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| unbonding_responses | list of [`UnbondingDelegation`](#staking-type-unbonding-delegation) | One record per validator, each with its entries. |

<a id="staking-query-redelegations"></a>

#### Redelegations

Lists one account's redelegations that have not completed. You can narrow it to a source validator, a destination validator, or both.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Redelegations` |
| REST | `GET /cosmos/staking/v1beta1/delegators/{delegator_addr}/redelegations`, with optional `?src_validator_addr=` and `&dst_validator_addr=` |
| urid | `urid query staking redelegation [delegator-addr] [src-validator-addr] [dst-validator-addr]` |

```proto
message QueryRedelegationsRequest {
  string delegator_addr = 1;
  string src_validator_addr = 2;
  string dst_validator_addr = 3;
  cosmos.base.query.v1beta1.PageRequest pagination = 4;
}

message QueryRedelegationsResponse {
  repeated RedelegationResponse redelegation_responses = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
```proto
message RedelegationResponse {
  Redelegation redelegation = 1;
  repeated RedelegationEntryResponse entries = 2;
}

message RedelegationEntryResponse {
  RedelegationEntry redelegation_entry = 1;
  string balance = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| src_validator_addr | address | Optional. Only redelegations from this validator. |
| dst_validator_addr | address | Optional. Only redelegations to this validator. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| redelegation_responses | list of `RedelegationResponse` | One per pair of validators. |
| redelegation | [`Redelegation`](#staking-type-redelegation) | The redelegation record. |
| entries | list of `RedelegationEntryResponse` | Each entry with its current worth. |
| redelegation_entry | [`RedelegationEntry`](#staking-type-redelegation) | The entry. |
| balance | integer as a string | What the entry is worth now, in kwak. |

<a id="staking-query-delegator-validators"></a>

#### DelegatorValidators

Lists the validators one account stakes with, in full.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/DelegatorValidators` |
| REST | `GET /cosmos/staking/v1beta1/delegators/{delegator_addr}/validators` |
| urid | `urid query staking delegator-validators [delegator-addr]` |

```proto
message QueryDelegatorValidatorsRequest {
  string delegator_addr = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message QueryDelegatorValidatorsResponse {
  repeated Validator validators = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| validators | list of [`Validator`](#staking-type-validator) | The validators. |

<a id="staking-query-delegator-validator"></a>

#### DelegatorValidator

Returns one validator, but only if the account stakes with it. Otherwise the query fails.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/DelegatorValidator` |
| REST | `GET /cosmos/staking/v1beta1/delegators/{delegator_addr}/validators/{validator_addr}` |
| urid | `urid query staking delegator-validator [delegator-addr] [validator-addr]` |

```proto
message QueryDelegatorValidatorRequest {
  string delegator_addr = 1;
  string validator_addr = 2;
}

message QueryDelegatorValidatorResponse {
  Validator validator = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_addr | address | The delegator. |
| validator_addr | address | The operator address. |
| validator | [`Validator`](#staking-type-validator) | The validator. |

<a id="staking-query-historical-info"></a>

#### HistoricalInfo

Returns the block header and the validator set at a recent height. The chain keeps only as many recent heights as the `historical_entries` parameter says. Older heights return an error.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/HistoricalInfo` |
| REST | `GET /cosmos/staking/v1beta1/historical_info/{height}` |
| urid | `urid query staking historical-info [height]` |

```proto
message QueryHistoricalInfoRequest {
  int64 height = 1;
}

message QueryHistoricalInfoResponse {
  HistoricalInfo hist = 1;
}
```
```proto
message HistoricalInfo {
  tendermint.types.Header header = 1;
  repeated Validator valset = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height. |
| hist | `HistoricalInfo` | The record at that height. |
| header | block header | The block's header. |
| valset | list of [`Validator`](#staking-type-validator) | The validator set at that height. |

<a id="staking-query-pool"></a>

#### Pool

Returns how much URI is staked and how much is on its way in or out of staking. Mint uses the bonded share to steer inflation.

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Pool` |
| REST | `GET /cosmos/staking/v1beta1/pool` |
| urid | `urid query staking pool` |

```proto
message QueryPoolRequest {}

message QueryPoolResponse {
  Pool pool = 1;
}
```
```proto
message Pool {
  string not_bonded_tokens = 1;
  string bonded_tokens = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pool | `Pool` | The two totals below. |
| not_bonded_tokens | integer as a string | Kwak staked with validators outside the active set, plus kwak that is unbonding. |
| bonded_tokens | integer as a string | Kwak staked with bonded validators. |

<a id="staking-query-params"></a>

#### Params

Returns the staking parameters, described under [Parameters](#staking-parameters).

| | |
|---|---|
| gRPC | `cosmos.staking.v1beta1.Query/Params` |
| REST | `GET /cosmos/staking/v1beta1/params` |
| urid | `urid query staking params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#staking-parameters) | The current parameters. |

<a id="staking-parameters"></a>

### Parameters

Parameters are the module's settings. Governance changes them with [MsgUpdateParams](#staking-msg-update-params) inside a proposal.

```proto
message Params {
  google.protobuf.Duration unbonding_time = 1;
  uint32 max_validators = 2;
  uint32 max_entries = 3;
  uint32 historical_entries = 4;
  string bond_denom = 5;
  string min_commission_rate = 6;
  cosmos.base.v1beta1.Coin key_rotation_fee = 7;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| unbonding_time | How long unbonding takes, for delegators and for validators that leave the active set. Also how long a redelegation entry lasts. | 21 days, `1814400s` in JSON |
| max_validators | The most validators that can be bonded at once. | read it with the Params query |
| max_entries | The most unbonding entries per delegator and validator, and the most redelegation entries per delegator and pair of validators. | read it with the Params query |
| historical_entries | How many recent heights HistoricalInfo keeps. | read it with the Params query |
| bond_denom | The coin that can be staked. It can never change. | `kwak` |
| min_commission_rate | The lowest commission any validator may charge, as a fraction. | read it with the Params query |
| key_rotation_fee | The fee for MsgRotateConsPubKey. Its denom can never change. | In kwak. Read the amount with the Params query |

<a id="std-distribution"></a>

## Distribution

<a id="distribution-overview"></a>

### What it is

Distribution pays out staking rewards. Each block, the fees collected and the newly minted URI are shared among the bonded validators by stake. A part set by the `community_tax` parameter goes to the community pool first. Each validator keeps its commission, and the rest is shared among its delegators by stake.

Rewards wait on the chain until you withdraw them. They go to your withdraw address, which is your own account unless you set another. When your stake with a validator changes, the rewards from that validator so far are paid out on their own. Delegating more, unbonding, and redelegating all count as a change.

The words to know:

- **Rewards.** What a delegator has earned. The chain keeps them with 18 decimal places, but only whole kwak are paid out. The fraction left over at a withdrawal goes to the community pool.
- **Commission.** The validator's part of its rewards. The operator withdraws it.
- **Community pool.** URI held by the chain for shared purposes. Only governance can spend it. On UriNet nobody can pay into it directly.
- **Withdraw address.** Where your rewards go.
- **Period.** The chain numbers each validator's reward history in periods. A new period starts when a stake with the validator changes or the validator is slashed. Only the low-level queries use it.

These messages are signed by a SmartAccount with a signature bundle. The commission is withdrawn by the operator's account.

<a id="distribution-type-dec-coin"></a>

#### DecCoin

A coin amount with decimal places. Rewards, commission, and the community pool are all reported this way.

```proto
message DecCoin {
  string denom = 1;
  string amount = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| denom | string | Always `kwak` here. |
| amount | decimal as a string | The amount, with 18 decimal places. |

<a id="distribution-type-validator-rewards"></a>

#### Validator reward records

What the chain holds for a validator. `ValidatorAccumulatedCommission` is the commission not yet withdrawn. `ValidatorOutstandingRewards` is everything not yet withdrawn, by the validator and all its delegators.

```proto
message ValidatorAccumulatedCommission {
  repeated cosmos.base.v1beta1.DecCoin commission = 1;
}

message ValidatorOutstandingRewards {
  repeated cosmos.base.v1beta1.DecCoin rewards = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| commission | list of [`DecCoin`](#distribution-type-dec-coin) | The commission waiting to be withdrawn. |
| rewards | list of [`DecCoin`](#distribution-type-dec-coin) | All rewards waiting to be withdrawn. |

<a id="distribution-type-validator-slash-event"></a>

#### ValidatorSlashEvent

One slash of a validator, kept so that later withdrawals pay the right amount.

```proto
message ValidatorSlashEvent {
  uint64 validator_period = 1;
  string fraction = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_period | number | The validator's reward period when the slash happened. |
| fraction | decimal as a string | The share of the stake that was cut. |

<a id="distribution-type-delegation-delegator-reward"></a>

#### DelegationDelegatorReward

A delegator's rewards from one validator.

```proto
message DelegationDelegatorReward {
  string validator_address = 1;
  repeated cosmos.base.v1beta1.DecCoin reward = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The validator's operator address. |
| reward | list of [`DecCoin`](#distribution-type-dec-coin) | The rewards earned from it so far. |

<a id="distribution-transactions"></a>

### Transactions

Delegators and operators withdraw with these messages, and governance spends the community pool.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgSetWithdrawAddress](#distribution-msg-set-withdraw-address) | Chooses where your rewards go | The delegator |
| [MsgWithdrawDelegatorReward](#distribution-msg-withdraw-delegator-reward) | Withdraws your rewards from one validator | The delegator |
| [MsgWithdrawValidatorCommission](#distribution-msg-withdraw-validator-commission) | Withdraws a validator's commission | The operator |
| [MsgCommunityPoolSpend](#distribution-msg-community-pool-spend) | Pays URI out of the community pool | Governance |
| [MsgUpdateParams](#distribution-msg-update-params) | Changes the distribution parameters | Governance |

<a id="distribution-msg-set-withdraw-address"></a>

#### MsgSetWithdrawAddress

Chooses the account that receives your rewards from now on. The shop stakes from its main account and sends the rewards to its accounting account.

| | |
|---|---|
| Signer | `delegator_address`, the account whose rewards are redirected |
| Before you send it | The `withdraw_addr_enabled` parameter is true. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx distribution set-withdraw-addr [withdraw-addr]` |

```proto
message MsgSetWithdrawAddress {
  string delegator_address = 1;
  string withdraw_address = 2;
}

message MsgSetWithdrawAddressResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account whose rewards are redirected. |
| withdraw_address | address | The account that receives them. |

**Refused when**

- `withdraw_addr_enabled` is false.
- The withdraw address is one the chain reserves for its own modules, which cannot receive funds.
- An address is not a valid `uri1` address.

<a id="distribution-msg-withdraw-delegator-reward"></a>

#### MsgWithdrawDelegatorReward

Withdraws your rewards from one validator to your withdraw address. To collect from every validator at once, `urid tx distribution withdraw-all-rewards` sends one of these per validator. Its `--max-msgs` flag splits them across several transactions.

| | |
|---|---|
| Signer | `delegator_address`, the account that stakes |
| Before you send it | You stake with this validator. |
| What comes back | `amount`, the coins paid out |
| urid | `urid tx distribution withdraw-rewards [validator-addr]`. An operator adds `--commission` to withdraw the commission in the same transaction. |

```proto
message MsgWithdrawDelegatorReward {
  string delegator_address = 1;
  string validator_address = 2;
}

message MsgWithdrawDelegatorRewardResponse {
  repeated cosmos.base.v1beta1.Coin amount = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The account that stakes. |
| validator_address | address | The validator's operator address. |
| amount | list of coins | In the response. The whole kwak paid out. |

**Refused when**

- The validator does not exist.
- You have no stake with this validator.

<a id="distribution-msg-withdraw-validator-commission"></a>

#### MsgWithdrawValidatorCommission

Withdraws all of a validator's commission to the operator's withdraw address.

| | |
|---|---|
| Signer | `validator_address`, the operator's account in its `urivaloper1` form |
| Before you send it | The validator has earned commission since the last withdrawal. |
| What comes back | `amount`, the coins paid out |
| urid | `urid tx distribution withdraw-validator-commission [validator-addr]` |

```proto
message MsgWithdrawValidatorCommission {
  string validator_address = 1;
}

message MsgWithdrawValidatorCommissionResponse {
  repeated cosmos.base.v1beta1.Coin amount = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| amount | list of coins | In the response. The whole kwak paid out. |

**Refused when**

- The validator does not exist.
- There is no commission to withdraw.

<a id="distribution-msg-community-pool-spend"></a>

#### MsgCommunityPoolSpend

Pays URI from the community pool to an account. A proposal might fund a public service that the community has voted for.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | The pool holds the amount. Check it with the [CommunityPool](#distribution-query-community-pool) query. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx distribution community-pool-spend-proposal [recipient] [amount]`, with `--title`, `--summary`, `--deposit`, and `--metadata` |

```proto
message MsgCommunityPoolSpend {
  string authority = 1;
  string recipient = 2;
  repeated cosmos.base.v1beta1.Coin amount = 3;
}

message MsgCommunityPoolSpendResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| recipient | address | The account that receives the coins. |
| amount | list of coins | How much to pay, in kwak. |

**Refused when**

- The signer is not the governance account.
- The pool holds less than the amount.
- The recipient is one the chain reserves for its own modules, which cannot receive funds.
- The amount is empty or invalid.

<a id="distribution-msg-update-params"></a>

#### MsgUpdateParams

Replaces the distribution parameters. The message replaces all of them, so start from the current values. A parameter left out of the file becomes zero.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values with the [Params](#distribution-query-params) query and change only what you mean to. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx distribution update-params-proposal [params]`, with the parameters as JSON. It submits a normal proposal carrying the message. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#distribution-parameters) | The complete new parameters. |

**Refused when**

- The signer is not the governance account.
- A value fails the parameter checks, for example a community tax above 1.

<a id="distribution-not-available"></a>

#### Not available

Two distribution messages are refused on UriNet. `urid` still lists commands for them, but the chain refuses what they send.

| Message | urid | Why |
|---|---|---|
| MsgFundCommunityPool | `urid tx distribution fund-community-pool` | The chain refuses it before the fee is taken. |
| MsgDepositValidatorRewardsPool | `urid tx distribution fund-validator-rewards-pool` | The chain refuses it before the fee is taken. |

The error is "message not allowed". See [Errors](#appendix-errors).

<a id="distribution-queries"></a>

### Queries

The distribution queries read rewards, commission, the community pool, and the reward records behind them.

| Query | What it answers |
|---|---|
| [Params](#distribution-query-params) | The distribution parameters |
| [ValidatorDistributionInfo](#distribution-query-validator-distribution-info) | A validator's commission and its own staking rewards |
| [ValidatorOutstandingRewards](#distribution-query-validator-outstanding-rewards) | Everything not yet withdrawn at a validator |
| [ValidatorCommission](#distribution-query-validator-commission) | A validator's commission not yet withdrawn |
| [ValidatorSlashes](#distribution-query-validator-slashes) | A validator's slashes in a height range |
| [DelegationRewards](#distribution-query-delegation-rewards) | One account's rewards from one validator |
| [DelegationTotalRewards](#distribution-query-delegation-total-rewards) | One account's rewards from every validator |
| [DelegatorValidators](#distribution-query-delegator-validators) | The validators one account stakes with |
| [DelegatorWithdrawAddress](#distribution-query-delegator-withdraw-address) | Where one account's rewards go |
| [CommunityPool](#distribution-query-community-pool) | What the community pool holds |
| [ValidatorHistoricalRewards](#distribution-query-validator-historical-rewards) | A validator's reward record for one past period |
| [ValidatorCurrentRewards](#distribution-query-validator-current-rewards) | A validator's reward record for the current period |
| [DelegatorStartingInfo](#distribution-query-delegator-starting-info) | Where a stake's current reward period began |

<a id="distribution-query-params"></a>

#### Params

Returns the distribution parameters, described under [Parameters](#distribution-parameters).

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/Params` |
| REST | `GET /cosmos/distribution/v1beta1/params` |
| urid | `urid query distribution params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#distribution-parameters) | The current parameters. |

<a id="distribution-query-validator-distribution-info"></a>

#### ValidatorDistributionInfo

Returns what an operator can withdraw: its commission and the rewards on its own self-delegation.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorDistributionInfo` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}` |
| urid | `urid query distribution validator-distribution-info [validator]` |

```proto
message QueryValidatorDistributionInfoRequest {
  string validator_address = 1;
}

message QueryValidatorDistributionInfoResponse {
  string operator_address = 1;
  repeated cosmos.base.v1beta1.DecCoin self_bond_rewards = 2;
  repeated cosmos.base.v1beta1.DecCoin commission = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| operator_address | address | In the response. The operator address. |
| self_bond_rewards | list of [`DecCoin`](#distribution-type-dec-coin) | The rewards on the self-delegation. |
| commission | list of [`DecCoin`](#distribution-type-dec-coin) | The commission not yet withdrawn. |

<a id="distribution-query-validator-outstanding-rewards"></a>

#### ValidatorOutstandingRewards

Returns everything not yet withdrawn at a validator, for the validator and all its delegators together.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorOutstandingRewards` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}/outstanding_rewards` |
| urid | `urid query distribution validator-outstanding-rewards [validator]` |

```proto
message QueryValidatorOutstandingRewardsRequest {
  string validator_address = 1;
}

message QueryValidatorOutstandingRewardsResponse {
  ValidatorOutstandingRewards rewards = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| rewards | [`ValidatorOutstandingRewards`](#distribution-type-validator-rewards) | The total waiting to be withdrawn. |

<a id="distribution-query-validator-commission"></a>

#### ValidatorCommission

Returns the commission a validator has not withdrawn yet.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorCommission` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}/commission` |
| urid | `urid query distribution commission [validator]` |

```proto
message QueryValidatorCommissionRequest {
  string validator_address = 1;
}

message QueryValidatorCommissionResponse {
  ValidatorAccumulatedCommission commission = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| commission | [`ValidatorAccumulatedCommission`](#distribution-type-validator-rewards) | The commission waiting to be withdrawn. |

<a id="distribution-query-validator-slashes"></a>

#### ValidatorSlashes

Lists a validator's slashes between two block heights. Delegators use it to judge a validator's record.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorSlashes` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}/slashes`, with `?starting_height=` and `&ending_height=` |
| urid | `urid query distribution slashes [validator] [start-height] [end-height]` |

```proto
message QueryValidatorSlashesRequest {
  string validator_address = 1;
  uint64 starting_height = 2;
  uint64 ending_height = 3;
  cosmos.base.query.v1beta1.PageRequest pagination = 4;
}

message QueryValidatorSlashesResponse {
  repeated ValidatorSlashEvent slashes = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| starting_height | number | The first block height to search. |
| ending_height | number | The last block height to search. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| slashes | list of [`ValidatorSlashEvent`](#distribution-type-validator-slash-event) | The slashes in that range. |

<a id="distribution-query-delegation-rewards"></a>

#### DelegationRewards

Returns one account's rewards from one validator, not yet withdrawn.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/DelegationRewards` |
| REST | `GET /cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards/{validator_address}` |
| urid | `urid query distribution rewards-by-validator [delegator-addr] [validator-addr]` |

```proto
message QueryDelegationRewardsRequest {
  string delegator_address = 1;
  string validator_address = 2;
}

message QueryDelegationRewardsResponse {
  repeated cosmos.base.v1beta1.DecCoin rewards = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| validator_address | address | The operator address. |
| rewards | list of [`DecCoin`](#distribution-type-dec-coin) | The rewards waiting to be withdrawn. |

<a id="distribution-query-delegation-total-rewards"></a>

#### DelegationTotalRewards

Returns one account's rewards from every validator it stakes with, and their sum. A wallet shows this as "rewards to claim".

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/DelegationTotalRewards` |
| REST | `GET /cosmos/distribution/v1beta1/delegators/{delegator_address}/rewards` |
| urid | `urid query distribution rewards [delegator-addr]` |

```proto
message QueryDelegationTotalRewardsRequest {
  string delegator_address = 1;
}

message QueryDelegationTotalRewardsResponse {
  repeated DelegationDelegatorReward rewards = 1;
  repeated cosmos.base.v1beta1.DecCoin total = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| rewards | list of [`DelegationDelegatorReward`](#distribution-type-delegation-delegator-reward) | The rewards from each validator. |
| total | list of [`DecCoin`](#distribution-type-dec-coin) | The sum over all validators. |

<a id="distribution-query-delegator-validators"></a>

#### DelegatorValidators

Lists the operator addresses of the validators one account stakes with. It is not paged.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/DelegatorValidators` |
| REST | `GET /cosmos/distribution/v1beta1/delegators/{delegator_address}/validators` |
| urid | `urid query distribution delegator-validators --delegator-address [address]` |

```proto
message QueryDelegatorValidatorsRequest {
  string delegator_address = 1;
}

message QueryDelegatorValidatorsResponse {
  repeated string validators = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| validators | list of addresses | The operator addresses, `urivaloper1...`. |

<a id="distribution-query-delegator-withdraw-address"></a>

#### DelegatorWithdrawAddress

Returns where one account's rewards go. It is the account itself unless MsgSetWithdrawAddress chose another.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/DelegatorWithdrawAddress` |
| REST | `GET /cosmos/distribution/v1beta1/delegators/{delegator_address}/withdraw_address` |
| urid | `urid query distribution delegator-withdraw-address --delegator-address [address]` |

```proto
message QueryDelegatorWithdrawAddressRequest {
  string delegator_address = 1;
}

message QueryDelegatorWithdrawAddressResponse {
  string withdraw_address = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| withdraw_address | address | In the response. The account that receives the rewards. |

<a id="distribution-query-community-pool"></a>

#### CommunityPool

Returns what the community pool holds. Read it before you propose a MsgCommunityPoolSpend.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/CommunityPool` |
| REST | `GET /cosmos/distribution/v1beta1/community_pool` |
| urid | `urid query distribution community-pool` |

```proto
message QueryCommunityPoolRequest {}

message QueryCommunityPoolResponse {
  repeated cosmos.base.v1beta1.DecCoin pool = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| pool | list of [`DecCoin`](#distribution-type-dec-coin) | The pool's balance. |

<a id="distribution-query-validator-historical-rewards"></a>

#### ValidatorHistoricalRewards

Returns a validator's reward record for one past period. The chain uses these records to work out each delegator's rewards. Most readers never need it.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorHistoricalRewards` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}/historical_rewards/{period}` |
| urid | `urid query distribution validator-historical-rewards [validator] [period]` |

```proto
message QueryValidatorHistoricalRewardsRequest {
  string validator_address = 1;
  uint64 period = 2;
}

message QueryValidatorHistoricalRewardsResponse {
  ValidatorHistoricalRewards rewards = 1;
}
```
```proto
message ValidatorHistoricalRewards {
  repeated cosmos.base.v1beta1.DecCoin cumulative_reward_ratio = 1;
  uint32 reference_count = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| period | number | The period to read. |
| rewards | `ValidatorHistoricalRewards` | The record. |
| cumulative_reward_ratio | list of [`DecCoin`](#distribution-type-dec-coin) | The rewards per staked kwak, added up from the first period to this one. |
| reference_count | number | How many stakes and slashes still refer to this record. |

<a id="distribution-query-validator-current-rewards"></a>

#### ValidatorCurrentRewards

Returns a validator's reward record for the period that is still running.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/ValidatorCurrentRewards` |
| REST | `GET /cosmos/distribution/v1beta1/validators/{validator_address}/current_rewards` |
| urid | `urid query distribution validator-current-rewards [validator]` |

```proto
message QueryValidatorCurrentRewardsRequest {
  string validator_address = 1;
}

message QueryValidatorCurrentRewardsResponse {
  ValidatorCurrentRewards rewards = 1;
}
```
```proto
message ValidatorCurrentRewards {
  repeated cosmos.base.v1beta1.DecCoin rewards = 1;
  uint64 period = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| validator_address | address | The operator address. |
| rewards | `ValidatorCurrentRewards` | The record. |
| rewards (inner) | list of [`DecCoin`](#distribution-type-dec-coin) | The rewards gathered in the current period. |
| period | number | The number of the current period. |

<a id="distribution-query-delegator-starting-info"></a>

#### DelegatorStartingInfo

Returns where a stake's current reward period began. It is the starting point the chain counts that stake's rewards from.

| | |
|---|---|
| gRPC | `cosmos.distribution.v1beta1.Query/DelegatorStartingInfo` |
| REST | `GET /cosmos/distribution/v1beta1/delegators/{delegator_address}/starting_info/{validator_address}` |
| urid | `urid query distribution delegator-starting-info [delegator-address] [validator-address]` |

```proto
message QueryDelegatorStartingInfoRequest {
  string delegator_address = 1;
  string validator_address = 2;
}

message QueryDelegatorStartingInfoResponse {
  DelegatorStartingInfo starting_info = 1;
}
```
```proto
message DelegatorStartingInfo {
  uint64 previous_period = 1;
  string stake = 2;
  uint64 height = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| delegator_address | address | The delegator. |
| validator_address | address | The operator address. |
| starting_info | `DelegatorStartingInfo` | The record. |
| previous_period | number | The validator's period just before this stake's rewards started counting. |
| stake | decimal as a string | The kwak the stake was worth at that point. |
| height | number | The block height where it started. |

<a id="distribution-parameters"></a>

### Parameters

Parameters are the module's settings. Governance changes them with [MsgUpdateParams](#distribution-msg-update-params) inside a proposal.

```proto
message Params {
  string community_tax = 1;
  string base_proposer_reward = 2;
  string bonus_proposer_reward = 3;
  bool withdraw_addr_enabled = 4;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| community_tax | The share of each block's rewards that goes to the community pool, as a fraction. | read it with the Params query |
| base_proposer_reward | No longer used. Keep the current value when you change parameters. | Not used |
| bonus_proposer_reward | No longer used. Keep the current value when you change parameters. | Not used |
| withdraw_addr_enabled | Whether MsgSetWithdrawAddress is allowed. | read it with the Params query |

<a id="std-slashing"></a>

## Slashing

<a id="slashing-overview"></a>

### What it is

Slashing punishes validators that misbehave. It watches for two offences. Downtime means the validator missed too many blocks. Double signing means it signed two different blocks at the same height.

A punished validator is slashed and jailed. Slashing burns a fraction of the stake bonded to it, including its delegators' stake. Unbonding and redelegation entries that began after the offence are cut too. Jailing removes the validator from block signing.

- **Downtime.** Over the last `signed_blocks_window` blocks, a validator must sign at least the fraction `min_signed_per_window`. If it falls short, it is jailed for 10 minutes and slashed by `slash_fraction_downtime`. When the 10 minutes are over, the operator sends [MsgUnjail](#slashing-msg-unjail) to return.
- **Double signing.** The validator is slashed by `slash_fraction_double_sign` and tombstoned, which means jailed for good. A tombstoned validator can never be unjailed.

For a delegator, this is the risk of staking. Choose validators with a good record, which the [ValidatorSlashes](#distribution-query-validator-slashes) query shows.

The chain tracks each validator's signing in a signing info record, found by the validator's consensus address, `urivalcons1...`. That address comes from the node's consensus key, not from the operator's account.

<a id="slashing-type-validator-signing-info"></a>

#### ValidatorSigningInfo

A validator's signing record.

```proto
message ValidatorSigningInfo {
  string address = 1;
  int64 start_height = 2;
  int64 index_offset = 3;
  google.protobuf.Timestamp jailed_until = 4;
  bool tombstoned = 5;
  int64 missed_blocks_counter = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The consensus address, `urivalcons1...`. |
| start_height | number | The height where the validator first became a candidate, or was last unjailed. |
| index_offset | number | A counter of blocks the validator was expected to sign. The chain uses it to track the window. |
| jailed_until | timestamp | The time the jail ends. MsgUnjail works only after it. |
| tombstoned | boolean | True if the validator double signed and can never return. |
| missed_blocks_counter | number | Blocks missed within the current window. |

<a id="slashing-transactions"></a>

### Transactions

An operator returns a jailed validator with MsgUnjail, and governance changes the rules.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgUnjail](#slashing-msg-unjail) | Returns a jailed validator to block signing | The operator |
| [MsgUpdateParams](#slashing-msg-update-params) | Changes the slashing parameters | Governance |

<a id="slashing-msg-unjail"></a>

#### MsgUnjail

Returns a jailed validator to the set of candidates, so it can sign blocks and earn rewards again. It is sent once the jail time is over. Fix whatever made the node miss blocks before you send it, or it may be jailed again.

| | |
|---|---|
| Signer | `validator_addr`, the operator's account in its `urivaloper1` form |
| Before you send it | The time in `jailed_until` has passed. Check it with the [SigningInfo](#slashing-query-signing-info) query. The operator's self-delegation is at least `min_self_delegation`. |
| What comes back | Nothing. The transaction result tells you it went through. |
| urid | `urid tx slashing unjail`. It takes no arguments. The validator is the one run by the signing account. |

```proto
message MsgUnjail {
  string validator_addr = 1;
}

message MsgUnjailResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| validator_addr | address | The operator address. |

**Refused when**

- No validator exists for this operator.
- The validator is not jailed.
- The jail time is not over yet.
- The validator is tombstoned.
- The operator has no self-delegation, or less than `min_self_delegation`.

<a id="slashing-msg-update-params"></a>

#### MsgUpdateParams

Replaces the slashing parameters. The message replaces all of them, so start from the current values. A parameter left out of the file becomes zero.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values with the [Params](#slashing-query-params) query and change only what you mean to. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx slashing update-params-proposal [params]`, with the parameters as JSON. It submits a normal proposal carrying the message. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#slashing-parameters) | The complete new parameters. |

**Refused when**

- The signer is not the governance account.
- A value fails the parameter checks, for example a zero jail time or a fraction above 1.

<a id="slashing-queries"></a>

### Queries

The slashing queries read the rules and each validator's signing record.

| Query | What it answers |
|---|---|
| [Params](#slashing-query-params) | The slashing parameters |
| [SigningInfo](#slashing-query-signing-info) | One validator's signing record |
| [SigningInfos](#slashing-query-signing-infos) | Every validator's signing record |

<a id="slashing-query-params"></a>

#### Params

Returns the slashing parameters, described under [Parameters](#slashing-parameters).

| | |
|---|---|
| gRPC | `cosmos.slashing.v1beta1.Query/Params` |
| REST | `GET /cosmos/slashing/v1beta1/params` |
| urid | `urid query slashing params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#slashing-parameters) | The current parameters. |

<a id="slashing-query-signing-info"></a>

#### SigningInfo

Returns one validator's signing record. An operator checks `jailed_until` here before sending MsgUnjail.

| | |
|---|---|
| gRPC | `cosmos.slashing.v1beta1.Query/SigningInfo` |
| REST | `GET /cosmos/slashing/v1beta1/signing_infos/{cons_address}` |
| urid | `urid query slashing signing-info [validator-conspub/address]` |

`urid` takes either the consensus address or the consensus public key as `@type` JSON. The key is the `consensus_pubkey` of the [Validator](#staking-query-validator) query, or the output of `urid comet show-validator` on the node.

```proto
message QuerySigningInfoRequest {
  string cons_address = 1;
}

message QuerySigningInfoResponse {
  ValidatorSigningInfo val_signing_info = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| cons_address | address | The consensus address, `urivalcons1...`. |
| val_signing_info | [`ValidatorSigningInfo`](#slashing-type-validator-signing-info) | The signing record. |

<a id="slashing-query-signing-infos"></a>

#### SigningInfos

Lists the signing records of all validators.

| | |
|---|---|
| gRPC | `cosmos.slashing.v1beta1.Query/SigningInfos` |
| REST | `GET /cosmos/slashing/v1beta1/signing_infos` |
| urid | `urid query slashing signing-infos` |

```proto
message QuerySigningInfosRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QuerySigningInfosResponse {
  repeated cosmos.slashing.v1beta1.ValidatorSigningInfo info = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| info | list of [`ValidatorSigningInfo`](#slashing-type-validator-signing-info) | The signing records on this page. |

<a id="slashing-parameters"></a>

### Parameters

Parameters are the module's settings. Governance changes them with [MsgUpdateParams](#slashing-msg-update-params) inside a proposal.

```proto
message Params {
  int64 signed_blocks_window = 1;
  bytes min_signed_per_window = 2;
  google.protobuf.Duration downtime_jail_duration = 3;
  bytes slash_fraction_double_sign = 4;
  bytes slash_fraction_downtime = 5;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| signed_blocks_window | The number of recent blocks over which downtime is judged. | read it with the Params query |
| min_signed_per_window | The fraction of the window a validator must sign. | read it with the Params query |
| downtime_jail_duration | How long a validator stays jailed for downtime. | 10 minutes, `600s` in JSON |
| slash_fraction_double_sign | The fraction of stake burned for double signing. | read it with the Params query |
| slash_fraction_downtime | The fraction of stake burned for downtime. | read it with the Params query |

<a id="std-mint"></a>

## Mint

<a id="mint-overview"></a>

### What it is

Mint creates new URI every block. The new coins join the block's fees and are paid to validators and delegators through [Distribution](#std-distribution). The mint denom is kwak, and it can never change.

How much is created depends on the inflation rate, a yearly rate that the chain adjusts every block. When less of the supply is staked than the `goal_bonded` target, the rate rises toward `inflation_max`. When more is staked, it falls toward `inflation_min`. It moves by at most `inflation_rate_change` in a year. The staked share comes from the [Pool](#staking-query-pool) query.

The numbers of the schedule are not fixed in this manual. Read them with the [Params](#mint-query-params) query, and the current rate with the [Inflation](#mint-query-inflation) query.

<a id="mint-type-minter"></a>

#### Minter

The minting state. The [Inflation](#mint-query-inflation) and [AnnualProvisions](#mint-query-annual-provisions) queries each return one of its fields.

```proto
message Minter {
  string inflation = 1;
  string annual_provisions = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| inflation | decimal as a string | The current yearly inflation rate, as a fraction. |
| annual_provisions | decimal as a string | The kwak expected to be created over a year at the current rate. |

<a id="mint-transactions"></a>

### Transactions

Mint has one message, and only governance sends it.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgUpdateParams](#mint-msg-update-params) | Changes the mint parameters | Governance |

<a id="mint-msg-update-params"></a>

#### MsgUpdateParams

Replaces the mint parameters. The message replaces all of them, so start from the current values. A parameter left out of the file becomes zero. `mint_denom` is kwak and can never change.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current values with the [Params](#mint-query-params) query and change only what you mean to. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx mint update-params-proposal [params]`, with the parameters as JSON. It submits a normal proposal carrying the message. |

```proto
message MsgUpdateParams {
  string authority = 1;
  Params params = 2;
}

message MsgUpdateParamsResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| params | [`Params`](#mint-parameters) | The complete new parameters. |

**Refused when**

- The signer is not the governance account.
- `mint_denom` differs from the current one. The error is "immutable protocol parameter".
- A value fails the parameter checks, for example `inflation_min` above `inflation_max`.

<a id="mint-queries"></a>

### Queries

The mint queries read the parameters and the current minting state.

| Query | What it answers |
|---|---|
| [Params](#mint-query-params) | The mint parameters |
| [Inflation](#mint-query-inflation) | The current yearly inflation rate |
| [AnnualProvisions](#mint-query-annual-provisions) | The kwak expected over a year at the current rate |

<a id="mint-query-params"></a>

#### Params

Returns the mint parameters, described under [Parameters](#mint-parameters).

| | |
|---|---|
| gRPC | `cosmos.mint.v1beta1.Query/Params` |
| REST | `GET /cosmos/mint/v1beta1/params` |
| urid | `urid query mint params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  Params params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`Params`](#mint-parameters) | The current parameters. |

<a id="mint-query-inflation"></a>

#### Inflation

Returns the current yearly inflation rate.

| | |
|---|---|
| gRPC | `cosmos.mint.v1beta1.Query/Inflation` |
| REST | `GET /cosmos/mint/v1beta1/inflation` |
| urid | `urid query mint inflation` |

```proto
message QueryInflationRequest {}

message QueryInflationResponse {
  bytes inflation = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| inflation | decimal as a string | The rate, as a fraction. |

<a id="mint-query-annual-provisions"></a>

#### AnnualProvisions

Returns how much kwak the chain expects to create over a year at the current rate.

| | |
|---|---|
| gRPC | `cosmos.mint.v1beta1.Query/AnnualProvisions` |
| REST | `GET /cosmos/mint/v1beta1/annual_provisions` |
| urid | `urid query mint annual-provisions` |

```proto
message QueryAnnualProvisionsRequest {}

message QueryAnnualProvisionsResponse {
  bytes annual_provisions = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| annual_provisions | decimal as a string | The expected kwak per year. |

<a id="mint-parameters"></a>

### Parameters

Parameters are the module's settings. Governance changes them with [MsgUpdateParams](#mint-msg-update-params) inside a proposal.

```proto
message Params {
  string mint_denom = 1;
  string inflation_rate_change = 2;
  string inflation_max = 3;
  string inflation_min = 4;
  string goal_bonded = 5;
  uint64 blocks_per_year = 6;
  string max_supply = 7;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| mint_denom | The coin that is minted. It can never change. | `kwak` |
| inflation_rate_change | The most the inflation rate can move in a year. | read it with the Params query |
| inflation_max | The highest inflation rate. | read it with the Params query |
| inflation_min | The lowest inflation rate. | read it with the Params query |
| goal_bonded | The share of the supply the chain aims to have staked. | read it with the Params query |
| blocks_per_year | The number of blocks the chain expects in a year. It turns the yearly rate into an amount per block. | read it with the Params query |
| max_supply | The most URI that can ever exist, in kwak. `0` means no limit. | read it with the Params query |

---

<a id="std-upgrade"></a>

## Upgrade

<a id="upgrade-overview"></a>

### What it is

The upgrade module schedules a software upgrade of the whole chain. Governance passes a plan with a name and a block height. When the chain reaches that height, it stops. Validators then start the new release of `urid`, which knows the plan by its name. The chain carries on from the same state.

If a node still runs the old release at that height, it stops and does not go on. So node operators watch for plans with the CurrentPlan query and install the new release before the height arrives. Once a plan has been carried out, its name is used up and can never be planned again.

Both messages are governance-only. Nobody sends them in an ordinary transaction. Anyone can read the current plan, past plans, and the version each module is at.

<a id="upgrade-plan"></a>

#### The upgrade plan

```proto
message Plan {
  string name = 1;
  google.protobuf.Timestamp time = 2;
  int64 height = 3;
  string info = 4;
  google.protobuf.Any upgraded_client_state = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| name | string | The name of the upgrade. The new release recognises the upgrade by this name. It must not be empty, and it must not be the name of an upgrade already carried out. |
| time | timestamp | No longer used. It must be left empty, or the plan is refused. |
| height | number | The block height at which the chain stops for the upgrade. |
| info | string | Free text for operators, such as where to download the new release. |
| upgraded_client_state | `Any` | No longer used. It must be left empty, or the plan is refused. |

<a id="upgrade-module-version"></a>

#### Module versions

```proto
message ModuleVersion {
  string name = 1;
  uint64 version = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| name | string | The module's name, such as `bank` or `token`. |
| version | number | The version of the module's stored data. It goes up when an upgrade migrates that module. |

<a id="upgrade-transactions"></a>

### Transactions

Both upgrade messages travel inside a governance proposal.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgSoftwareUpgrade](#upgrade-msg-software-upgrade) | Schedules an upgrade at a block height | Governance |
| [MsgCancelUpgrade](#upgrade-msg-cancel-upgrade) | Removes the plan that is waiting | Governance |

<a id="upgrade-msg-software-upgrade"></a>

#### MsgSoftwareUpgrade

Schedules an upgrade. When the proposal passes, the plan becomes the one waiting plan. If another plan was already waiting, the new one replaces it.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

The `urid` command builds the proposal for you. The account that submits it pays the deposit and signs the proposal, not the upgrade message itself.

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Pick a name that has never been used and a height that is still ahead of the chain. Announce the new release to node operators. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx upgrade software-upgrade [name] --upgrade-height [height] --upgrade-info [info] --title [title] --summary [summary] --deposit [coins]` |

```proto
message MsgSoftwareUpgrade {
  string authority = 1;
  Plan plan = 2;
}

message MsgSoftwareUpgradeResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. Read it with the [Authority](#upgrade-query-authority) query. |
| plan | [`Plan`](#upgrade-plan) | The upgrade to schedule. |

**Refused when**

- The message is not carried by a passed proposal, so `authority` is not the governance account.
- The plan name is empty, or an upgrade with that name was already carried out.
- The height is zero, or it is already behind the chain.
- The `time` or `upgraded_client_state` field is set.

<a id="upgrade-msg-cancel-upgrade"></a>

#### MsgCancelUpgrade

Removes the plan that is waiting, so the chain does not stop at its height. It has to pass before that height is reached. An upgrade that has already been carried out cannot be undone.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | A plan is waiting, and its height is far enough ahead for the proposal to pass first. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx upgrade cancel-software-upgrade --title [title] --summary [summary] --deposit [coins]`. The command `urid tx upgrade cancel-upgrade-proposal` builds the same proposal. |

```proto
message MsgCancelUpgrade {
  string authority = 1;
}

message MsgCancelUpgradeResponse {
}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |

**Refused when**

- The message is not carried by a passed proposal, so `authority` is not the governance account.

<a id="upgrade-queries"></a>

### Queries

These queries show what upgrade is coming and which upgrades have happened.

| Query | What it answers |
|---|---|
| [CurrentPlan](#upgrade-query-current-plan) | The plan that is waiting, if any |
| [AppliedPlan](#upgrade-query-applied-plan) | The height at which a named upgrade was carried out |
| [ModuleVersions](#upgrade-query-module-versions) | The version each module is at |
| [Authority](#upgrade-query-authority) | The account allowed to schedule upgrades |
| [UpgradedConsensusState](#upgrade-query-upgraded-consensus-state) | An old query kept for compatibility |

<a id="upgrade-query-current-plan"></a>

#### CurrentPlan

Returns the plan that is waiting. Node operators ask this to learn the height and the release they need. An empty answer means no upgrade is planned.

| | |
|---|---|
| gRPC | `cosmos.upgrade.v1beta1.Query/CurrentPlan` |
| REST | `GET /cosmos/upgrade/v1beta1/current_plan` |
| urid | `urid query upgrade plan` |

```proto
message QueryCurrentPlanRequest {}

message QueryCurrentPlanResponse {
  Plan plan = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| plan | [`Plan`](#upgrade-plan) | In the response. The waiting plan, or empty. |

<a id="upgrade-query-applied-plan"></a>

#### AppliedPlan

Returns the block height at which a named upgrade was carried out. Use it to tell which release of `urid` was valid for which range of blocks.

| | |
|---|---|
| gRPC | `cosmos.upgrade.v1beta1.Query/AppliedPlan` |
| REST | `GET /cosmos/upgrade/v1beta1/applied_plan/{name}` |
| urid | `urid query upgrade applied [upgrade-name]`, which prints the header of that block |

```proto
message QueryAppliedPlanRequest {
  string name = 1;
}

message QueryAppliedPlanResponse {
  int64 height = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| name | string | The upgrade's name. |
| height | number | In the response. The height where it was carried out, or 0 if no upgrade by that name has happened. |

<a id="upgrade-query-module-versions"></a>

#### ModuleVersions

Lists every module with the version of its stored data. Give a module name to see only that module.

| | |
|---|---|
| gRPC | `cosmos.upgrade.v1beta1.Query/ModuleVersions` |
| REST | `GET /cosmos/upgrade/v1beta1/module_versions` |
| urid | `urid query upgrade module-versions [optional module_name]` |

```proto
message QueryModuleVersionsRequest {
  string module_name = 1;
}

message QueryModuleVersionsResponse {
  repeated ModuleVersion module_versions = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| module_name | string | Optional. One module's name. Leave it empty for the full list. On REST it goes in the query string. |
| module_versions | list of [`ModuleVersion`](#upgrade-module-version) | In the response. The modules and their versions. |

<a id="upgrade-query-authority"></a>

#### Authority

Returns the address of the account allowed to schedule and cancel upgrades. On UriNet this is the governance account. Put it in the `authority` field of an upgrade message.

| | |
|---|---|
| gRPC | `cosmos.upgrade.v1beta1.Query/Authority` |
| REST | `GET /cosmos/upgrade/v1beta1/authority` |
| urid | `urid query upgrade authority` |

```proto
message QueryAuthorityRequest {
}

message QueryAuthorityResponse {
  string address = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | In the response. The governance account. |

<a id="upgrade-query-upgraded-consensus-state"></a>

#### UpgradedConsensusState

This query is deprecated and kept only so that older clients do not break. It served chains that connect to other chains, and it has nothing to tell a UriNet user.

| | |
|---|---|
| gRPC | `cosmos.upgrade.v1beta1.Query/UpgradedConsensusState` |
| REST | `GET /cosmos/upgrade/v1beta1/upgraded_consensus_state/{last_height}` |
| urid | none |

```proto
message QueryUpgradedConsensusStateRequest {
  int64 last_height = 1;
}

message QueryUpgradedConsensusStateResponse {
  reserved 1;
  bytes upgraded_consensus_state = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| last_height | number | The last height before an upgrade. |
| upgraded_consensus_state | bytes, base64 in JSON | In the response. Empty on UriNet. |

<a id="std-consensus"></a>

## Consensus

<a id="consensus-overview"></a>

### What it is

The consensus parameters are the rules the consensus engine applies to every block. The consensus engine is the layer under the chain that orders transactions into blocks and gets the validators to agree on them. Its parameters set the largest block and the most gas a block may use. They also set how long evidence of misbehaviour stays valid, and which key types validators may use.

The consensus module holds these parameters and lets governance change them. It has one message, which is governance-only, and one query. The version parameter is not changed here, because the upgrade module tracks it.

<a id="consensus-transactions"></a>

### Transactions

The one consensus message travels inside a governance proposal.

| Transaction | What it does | Who signs |
|---|---|---|
| [MsgUpdateParams](#consensus-msg-update-params) | Replaces the consensus parameters | Governance |

<a id="consensus-msg-update-params"></a>

#### MsgUpdateParams

Replaces the consensus parameters with new values. It takes the block, evidence, and validator parameters as a whole, so you send every value, not only the ones that change. A value left out of the file becomes zero. Start from the current values of the [Params](#consensus-query-params) query and edit them.

> **Governance only.** This message is carried inside a governance proposal. See [Governance](#ch-gov).

| | |
|---|---|
| Signer | `authority`, the governance account, through a passed proposal |
| Before you send it | Read the current parameters. Fill in `block`, `evidence`, and `validator` in full. |
| What comes back | Nothing. The proposal's result tells you it went through. |
| urid | `urid tx consensus update-params-proposal [params] --title [title] --summary [summary] --deposit [coins]`, with the parameters as JSON. It submits a normal proposal carrying the message. |

```proto
message MsgUpdateParams {
  string authority = 1;
  tendermint.types.BlockParams block = 2;
  tendermint.types.EvidenceParams evidence = 3;
  tendermint.types.ValidatorParams validator = 4;
  tendermint.types.ABCIParams abci = 5;
  tendermint.types.AuthorityParams auth = 6;
}

message MsgUpdateParamsResponse {}
```
| Field | Type | Meaning |
|---|---|---|
| authority | address | The governance account. |
| block | [`BlockParams`](#consensus-parameters) | Block size and gas limits. Required. |
| evidence | [`EvidenceParams`](#consensus-parameters) | How long evidence stays valid, and how much fits in a block. Required. |
| validator | [`ValidatorParams`](#consensus-parameters) | The key types validators may use. Required. |
| abci | [`ABCIParams`](#consensus-parameters) | Optional. The height from which vote extensions are turned on. |
| auth | [`AuthorityParams`](#consensus-parameters) | Optional. On UriNet its `authority` must stay empty. |

**Refused when**

- The message is not carried by a passed proposal, so `authority` is not the governance account.
- Any of `block`, `evidence`, or `validator` is missing.
- A value is out of range. Examples are a block size of zero, a block gas limit below -1, or more evidence bytes than block bytes.
- The list of validator key types is empty or names an unknown type.
- The vote extension height is set in the past, or changed after it has been reached.
- `auth.authority` is not empty. UriNet fixes it to empty, and the error is "immutable protocol parameter".

<a id="consensus-queries"></a>

### Queries

The one consensus query returns the parameters in force.

| Query | What it answers |
|---|---|
| [Params](#consensus-query-params) | The current consensus parameters |

<a id="consensus-query-params"></a>

#### Params

Returns the consensus parameters in force now. Ask it before you draft a change, and to learn the largest block and gas a block may carry.

| | |
|---|---|
| gRPC | `cosmos.consensus.v1.Query/Params` |
| REST | `GET /cosmos/consensus/v1/params` |
| urid | `urid query consensus params` |

```proto
message QueryParamsRequest {}

message QueryParamsResponse {
  tendermint.types.ConsensusParams params = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| params | [`ConsensusParams`](#consensus-parameters) | In the response. The parameters in force. Ignore the `version` part. The upgrade module tracks the application version. |

<a id="consensus-parameters"></a>

### Parameters

These are the consensus engine's own parameters. Governance changes them through [MsgUpdateParams](#consensus-msg-update-params) inside a proposal. Read the values in force with the [Params](#consensus-query-params) query.

```proto
message ConsensusParams {
  BlockParams block = 1;
  EvidenceParams evidence = 2;
  ValidatorParams validator = 3;
  VersionParams version = 4;
  ABCIParams abci = 5;
  AuthorityParams authority = 6;
}

message BlockParams {
  int64 max_bytes = 1;
  int64 max_gas = 2;
  reserved 3;
}

message EvidenceParams {
  int64 max_age_num_blocks = 1;
  google.protobuf.Duration max_age_duration = 2;
  int64 max_bytes = 3;
}

message ValidatorParams {
  repeated string pub_key_types = 1;
}

message VersionParams {
  uint64 app = 1;
}

message ABCIParams {
  int64 vote_extensions_enable_height = 1;
}

message AuthorityParams {
  string authority = 1;
}
```
| Parameter | Meaning | Value |
|---|---|---|
| block.max_bytes | The largest block, in bytes. Must be above 0. | read it with the Params query |
| block.max_gas | The most gas all transactions in one block may use together. -1 means no limit. | read it with the Params query |
| evidence.max_age_num_blocks | How many blocks old evidence of misbehaviour may be and still count. | read it with the Params query |
| evidence.max_age_duration | How much time old evidence may be and still count. Evidence must pass both age limits to count. | read it with the Params query |
| evidence.max_bytes | The most bytes of evidence one block may carry. Must stay below the block size. | read it with the Params query |
| validator.pub_key_types | The key types a validator's consensus key may use. | read it with the Params query |
| version.app | The application version. MsgUpdateParams does not change it. The upgrade module tracks it. | read it with the Params query |
| abci.vote_extensions_enable_height | The first height at which validators add vote extensions to their votes. 0 means they are off. | read it with the Params query |
| authority.authority | Reserved. A change to it is refused. | Empty. UriNet fixes it to empty. |

<a id="std-evidence"></a>

## Evidence

<a id="evidence-overview"></a>

### What it is

Evidence is proof that a validator broke the rules of consensus. The case the chain handles is double signing, where one validator signs two different blocks at the same height. The consensus engine detects it and hands it to the chain. The chain records the evidence and punishes the validator under the slashing rules.

The evidence module keeps these records. You can look them up by hash or list them all. Each record is packed with its type, and a double-signing record has the type `/cosmos.evidence.v1beta1.Equivocation`.

<a id="evidence-equivocation"></a>

#### Double signing

```proto
message Equivocation {
  int64 height = 1;
  google.protobuf.Timestamp time = 2;
  int64 power = 3;
  string consensus_address = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height at which the validator signed twice. |
| time | timestamp | The time of that block. |
| power | number | The validator's voting power at that time. |
| consensus_address | address | The validator's consensus address, starting with `urivalcons1`. |

<a id="evidence-not-available"></a>

### Not available

`MsgSubmitEvidence` is refused on UriNet, so nobody can file evidence in a transaction. The chain refuses it before the fee is taken. Misbehaviour evidence still reaches the chain from the consensus engine, and the queries below show what it recorded.

<a id="evidence-queries"></a>

### Queries

These queries read the recorded evidence.

| Query | What it answers |
|---|---|
| [Evidence](#evidence-query-evidence) | One record, by its hash |
| [AllEvidence](#evidence-query-all-evidence) | Every record, page by page |

<a id="evidence-query-evidence"></a>

#### Evidence

Returns one evidence record by its hash. Use it when an event or a block names the evidence and you want the details.

| | |
|---|---|
| gRPC | `cosmos.evidence.v1beta1.Query/Evidence` |
| REST | `GET /cosmos/evidence/v1beta1/evidence/{hash}` |
| urid | `urid query evidence evidence [hash]` |

```proto
message QueryEvidenceRequest {
  bytes evidence_hash = 1;
  string hash = 2;
}

message QueryEvidenceResponse {
  google.protobuf.Any evidence = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| evidence_hash | bytes, base64 in JSON | No longer used. Use `hash`. |
| hash | string | The evidence hash, in hex. |
| evidence | `Any` | In the response. The record, with its type. For double signing it is an [`Equivocation`](#evidence-equivocation). |

<a id="evidence-query-all-evidence"></a>

#### AllEvidence

Lists every evidence record the chain holds. An empty list means no validator has been caught misbehaving.

| | |
|---|---|
| gRPC | `cosmos.evidence.v1beta1.Query/AllEvidence` |
| REST | `GET /cosmos/evidence/v1beta1/evidence` |
| urid | `urid query evidence list` |

```proto
message QueryAllEvidenceRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message QueryAllEvidenceResponse {
  repeated google.protobuf.Any evidence = 1;
  cosmos.base.query.v1beta1.PageResponse pagination = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| evidence | list of `Any` | In the response. The records, each with its type. |

<a id="std-tx-service"></a>

## Transaction service

<a id="txsvc-overview"></a>

### What it is

The transaction service is how a signed transaction reaches the chain and how you read it back. It also simulates a transaction to estimate its gas, and it converts a transaction between its JSON form and its bytes. Every message in this manual goes through it. See [How transactions work](#how-transactions) for the full path, and [Sending transactions with urid](#urid-tx) for the command line.

Sending takes three steps. First you build the transaction: the messages, the fee, and one signer slot for each account that must sign. Then each signer signs it. On UriNet every account is a SmartAccount, an account controlled by one or more registered keys called credentials. A signer signs with a signature bundle, the set of credential proofs that meets the account's approval rule. Last, you broadcast the signed bytes with [BroadcastTx](#txsvc-query-broadcast-tx).

The chain checks the transaction before it takes the fee. Only messages from the chain's list of 77 accepted types get through. Only the sign modes DIRECT and DIRECT_AUX are accepted. In a sponsored transaction, the accounts that act sign DIRECT_AUX and the fee payer signs DIRECT. Legacy JSON signing is refused, and so are multisig mode info, extension options, and unordered transactions. One transaction carries at most 20 credential proofs across all its signers.

<a id="txsvc-broadcast-mode"></a>

#### Broadcast modes

BroadcastTx takes a mode that says how long the call waits.

| Mode | What happens |
|---|---|
| `BROADCAST_MODE_SYNC` | The node checks the transaction and answers with the result of that check. This is the usual choice and the `urid` default. |
| `BROADCAST_MODE_ASYNC` | The node answers at once, before it checks anything. |
| `BROADCAST_MODE_BLOCK` | Not supported. Use SYNC and then wait for the transaction. |

Either way you get back the transaction hash. The hash is the SHA-256 of the transaction bytes, written in hex. A sync answer with code 0 means the node accepted the transaction for a block. It does not yet mean the messages ran. Wait until it is in a block, then read it with [GetTx](#txsvc-query-get-tx), `urid query tx [hash]`, or `urid query wait-tx [hash]`. There, a code of 0 means success, and any other code is the reason it failed. See [Reading results](#urid-results) and [Errors](#appendix-errors).

<a id="txsvc-tx"></a>

#### The transaction

A transaction has a body, which says what to do, and an authentication part, which says who signs and who pays. The signatures come last, one per signer slot.

```proto
message Tx {
  TxBody body = 1;
  AuthInfo auth_info = 2;
  repeated bytes signatures = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| body | [`TxBody`](#txsvc-tx-body) | The messages and the options every signer signs. |
| auth_info | [`AuthInfo`](#txsvc-auth-info) | The signer slots and the fee. |
| signatures | list of bytes, base64 in JSON | One entry per signer slot, in the same order. For a SmartAccount the entry is its encoded signature bundle. A new account that signs its own creation puts a plain signature of its starting key here. |

<a id="txsvc-tx-body"></a>

#### The body

```proto
message TxBody {
  repeated google.protobuf.Any messages = 1;
  string memo = 2;
  uint64 timeout_height = 3;
  bool unordered = 4;
  google.protobuf.Timestamp timeout_timestamp = 5;
  repeated google.protobuf.Any extension_options = 1023;
  repeated google.protobuf.Any non_critical_extension_options = 2047;
}
```
| Field | Type | Meaning |
|---|---|---|
| messages | list of `Any` | The messages to run, in order. Each carries its type, such as `/cosmos.bank.v1beta1.MsgSend`. If one fails, none of them take effect. |
| memo | string | A free note stored with the transaction. The auth module's parameters cap its length. |
| timeout_height | number | Optional. The chain will not take the transaction after this height. |
| unordered | boolean | Must be false. Unordered transactions are refused. |
| timeout_timestamp | timestamp | Optional. The chain will not take the transaction after this block time. |
| extension_options | list of `Any` | Must be empty. The chain refuses a transaction that carries one. |
| non_critical_extension_options | list of `Any` | Ignored. |

<a id="txsvc-auth-info"></a>

#### Signers and fee

```proto
message AuthInfo {
  repeated SignerInfo signer_infos = 1;
  Fee fee = 2;
  Tip tip = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| signer_infos | list of [`SignerInfo`](#txsvc-signer-info) | One slot per account that must sign, in the order the messages name them. |
| fee | [`Fee`](#txsvc-fee) | The fee and the gas limit. |
| tip | `Tip` | No longer used. Leave it empty. |

<a id="txsvc-signer-info"></a>

#### A signer slot

```proto
message SignerInfo {
  google.protobuf.Any public_key = 1;
  ModeInfo mode_info = 2;
  uint64 sequence = 3;
}

message ModeInfo {
  oneof sum {
    Single single = 1;
    Multi multi = 2;
  }
  message Single {
    cosmos.tx.signing.v1beta1.SignMode mode = 1;
  }
  message Multi {
    cosmos.crypto.multisig.v1beta1.CompactBitArray bitarray = 1;
    repeated ModeInfo mode_infos = 2;
  }
}
```
| Field | Type | Meaning |
|---|---|---|
| public_key | `Any` | Left empty for a SmartAccount, which has no single account key. It is set only when a new account signs its own creation, and then it holds that account's starting key. |
| mode_info | `ModeInfo` | The sign mode of this slot. Only `single` is accepted, with `SIGN_MODE_DIRECT` or `SIGN_MODE_DIRECT_AUX`. In a sponsored transaction, the accounts that act use DIRECT_AUX and the fee payer uses DIRECT. `multi`, the multisig form, is refused. |
| sequence | number | The account's current sequence, the count of its transactions so far. It must match the chain, which raises it by one when the transaction is taken. |

<a id="txsvc-fee"></a>

#### The fee

```proto
message Fee {
  repeated cosmos.base.v1beta1.Coin amount = 1;
  uint64 gas_limit = 2;
  string payer = 3;
  string granter = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| amount | list of coins | The fee, in kwak only. It must be at least the gas limit times the minimum gas price, which is 1 kwak per unit of gas by default. |
| gas_limit | number | The most gas the transaction may use. `urid tx sign --gas auto` and the SDK estimate it by simulation and add 30 percent. |
| payer | address | Optional. A sponsor who pays the fee and signs as fee payer. Empty means the first signer pays. |
| granter | address | Optional. The account whose fee grant pays, or the delegator when the transaction uses a delegation. A delegator-paid transaction must use exactly one delegation and nothing else, and the delegation's `max_fee_total` caps the fee. |

<a id="txsvc-tx-response"></a>

#### The result

GetTx, GetTxsEvent, and BroadcastTx return the result of a transaction in this shape.

```proto
message TxResponse {
  int64 height = 1;
  string txhash = 2;
  string codespace = 3;
  uint32 code = 4;
  string data = 5;
  string raw_log = 6;
  repeated ABCIMessageLog logs = 7;
  string info = 8;
  int64 gas_wanted = 9;
  int64 gas_used = 10;
  google.protobuf.Any tx = 11;
  string timestamp = 12;
  repeated tendermint.abci.Event events = 13;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block that holds the transaction. 0 in a broadcast answer, before it is in a block. |
| txhash | string | The transaction hash, in hex. |
| codespace | string | Where the error code comes from, such as `urinet` or `token`. Empty on success. |
| code | number | 0 on success. Any other value is an error, read together with `codespace`. |
| data | string | The message responses, encoded, in hex. |
| raw_log | string | The error text when the transaction failed. |
| logs | list | Older per-message logs. Use `events`. |
| info | string | Extra information, if any. |
| gas_wanted | number | The gas limit the transaction asked for. |
| gas_used | number | The gas it used. |
| tx | `Any` | The transaction itself. |
| timestamp | string | The time of the block. |
| events | list of events | Everything the transaction emitted, fee and messages included. |

<a id="txsvc-urid"></a>

#### From the command line

These `urid` commands call the service.

- `urid tx broadcast [file_path]` sends a signed transaction file. `--broadcast-mode` takes `sync` or `async`.
- `urid tx encode [file]` turns a transaction file into base64 bytes.
- `urid tx decode [protobuf-byte-string]` turns base64 bytes back into JSON. Add `--hex` for hex input.
- `urid query tx [hash]` and `urid query txs --query [query]` read transactions from the chain.

`urid tx simulate` is not available on UriNet. It builds the transaction with an empty signature, which the chain refuses. Use `urid tx sign --gas auto` instead. It simulates with stand-in proofs for the credentials that will sign, and sets the gas limit from the result.

<a id="txsvc-queries"></a>

### Queries

The service has nine calls. Two of them simulate or send a transaction. The rest read or convert.

| Query | What it answers |
|---|---|
| [Simulate](#txsvc-query-simulate) | How much gas a transaction would use |
| [BroadcastTx](#txsvc-query-broadcast-tx) | Sends a signed transaction to the chain |
| [GetTx](#txsvc-query-get-tx) | One transaction and its result, by hash |
| [GetTxsEvent](#txsvc-query-get-txs-event) | The transactions whose events match a search |
| [GetBlockWithTxs](#txsvc-query-get-block-with-txs) | A block with its transactions decoded |
| [TxDecode](#txsvc-query-tx-decode) | Transaction bytes as JSON |
| [TxEncode](#txsvc-query-tx-encode) | A JSON transaction as bytes |
| [TxEncodeAmino](#txsvc-query-tx-encode-amino) | Legacy JSON as legacy bytes |
| [TxDecodeAmino](#txsvc-query-tx-decode-amino) | Legacy bytes as legacy JSON |

<a id="txsvc-query-simulate"></a>

#### Simulate

Runs a transaction without keeping its result and reports the gas it used. Use it to set the gas limit before signing. Nothing changes on the chain.

The gas depends on who signs and with which credentials. So the simulated transaction must name the same signers and credentials as the real one. The proof bytes themselves can be stand-ins. A transaction with empty signatures is refused. The real transaction may use a little more gas than the estimate, which is why tools add 30 percent.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/Simulate` |
| REST | `POST /cosmos/tx/v1beta1/simulate` |
| urid | `urid tx sign --gas auto`, which simulates for you |

```proto
message SimulateRequest {
  cosmos.tx.v1beta1.Tx tx = 1;
  bytes tx_bytes = 2;
}

message SimulateResponse {
  cosmos.base.abci.v1beta1.GasInfo gas_info = 1;
  cosmos.base.abci.v1beta1.Result result = 2;
}
```
```proto
message GasInfo {
  uint64 gas_wanted = 1;
  uint64 gas_used = 2;
}

message Result {
  bytes data = 1;
  string log = 2;
  repeated tendermint.abci.Event events = 3;
  repeated google.protobuf.Any msg_responses = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| tx | [`Tx`](#txsvc-tx) | No longer used. Send `tx_bytes`. |
| tx_bytes | bytes, base64 in JSON | The encoded transaction to simulate. |
| gas_info | `GasInfo` | In the response. `gas_used` is the estimate. `gas_wanted` is the limit the transaction asked for. |
| result | `Result` | In the response. What the messages would return: `log`, `events`, and `msg_responses`. `data` is no longer used. |

<a id="txsvc-query-broadcast-tx"></a>

#### BroadcastTx

Sends a signed transaction to the chain. The node checks it and, if it passes, shares it with the network to go into a block.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/BroadcastTx` |
| REST | `POST /cosmos/tx/v1beta1/txs` |
| urid | `urid tx broadcast [file_path]` |

```proto
message BroadcastTxRequest {
  bytes tx_bytes = 1;
  BroadcastMode mode = 2;
}

enum BroadcastMode {
  BROADCAST_MODE_UNSPECIFIED = 0;
  BROADCAST_MODE_BLOCK = 1;
  BROADCAST_MODE_SYNC = 2;
  BROADCAST_MODE_ASYNC = 3;
}

message BroadcastTxResponse {
  cosmos.base.abci.v1beta1.TxResponse tx_response = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| tx_bytes | bytes, base64 in JSON | The signed transaction. |
| mode | enum | `BROADCAST_MODE_SYNC` or `BROADCAST_MODE_ASYNC`. See [Broadcast modes](#txsvc-broadcast-mode). |
| tx_response | [`TxResponse`](#txsvc-tx-response) | In the response. The hash, and in sync mode the code of the node's check. |

A REST request looks like this:

```json
{
  "tx_bytes": "<the signed transaction, base64>",
  "mode": "BROADCAST_MODE_SYNC"
}
```

<a id="txsvc-query-get-tx"></a>

#### GetTx

Returns one transaction and its result, by hash. Ask it after a broadcast to learn whether the messages ran. If the transaction is not in a block yet, the call returns a not-found error.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/GetTx` |
| REST | `GET /cosmos/tx/v1beta1/txs/{hash}` |
| urid | `urid query tx [hash]` |

```proto
message GetTxRequest {
  string hash = 1;
}

message GetTxResponse {
  cosmos.tx.v1beta1.Tx tx = 1;
  cosmos.base.abci.v1beta1.TxResponse tx_response = 2;
}
```
| Field | Type | Meaning |
|---|---|---|
| hash | string | The transaction hash, in hex. |
| tx | [`Tx`](#txsvc-tx) | In the response. The transaction. |
| tx_response | [`TxResponse`](#txsvc-tx-response) | In the response. Its result. `code` 0 means it succeeded. |

<a id="txsvc-query-get-txs-event"></a>

#### GetTxsEvent

Finds transactions by the events they emitted. For example, `message.sender='uri1...'` finds what one account sent, and `tx.height=100` finds the transactions of one block. Conditions are joined with `AND`.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/GetTxsEvent` |
| REST | `GET /cosmos/tx/v1beta1/txs?query=...` |
| urid | `urid query txs --query [query]` |

```proto
message GetTxsEventRequest {
  repeated string events = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
  OrderBy order_by = 3;
  uint64 page = 4;
  uint64 limit = 5;
  string query = 6;
}

enum OrderBy {
  ORDER_BY_UNSPECIFIED = 0;
  ORDER_BY_ASC = 1;
  ORDER_BY_DESC = 2;
}

message GetTxsEventResponse {
  repeated cosmos.tx.v1beta1.Tx txs = 1;
  repeated cosmos.base.abci.v1beta1.TxResponse tx_responses = 2;
  cosmos.base.query.v1beta1.PageResponse pagination = 3;
  uint64 total = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| events | list of strings | No longer used. Use `query`. |
| pagination | page request | No longer used. Use `page` and `limit`. |
| order_by | enum | `ORDER_BY_ASC` (the default) or `ORDER_BY_DESC`, by height. |
| page | number | The page to return, starting at 1. |
| limit | number | How many results per page. |
| query | string | The event search. It must be a valid search, or the call fails. |
| txs | list of [`Tx`](#txsvc-tx) | In the response. The transactions found. |
| tx_responses | list of [`TxResponse`](#txsvc-tx-response) | In the response. Their results, in the same order. |
| total | number | In the response. How many transactions match in all. |

<a id="txsvc-query-get-block-with-txs"></a>

#### GetBlockWithTxs

Returns one block with its transactions decoded, not as raw bytes. Use it to walk the chain block by block. The transactions are paged.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/GetBlockWithTxs` |
| REST | `GET /cosmos/tx/v1beta1/txs/block/{height}` |
| urid | none. `urid query block --type=height [height]` shows the block. |

```proto
message GetBlockWithTxsRequest {
  int64 height = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message GetBlockWithTxsResponse {
  repeated cosmos.tx.v1beta1.Tx txs = 1;
  .tendermint.types.BlockID block_id = 2;
  .tendermint.types.Block block = 3;
  cosmos.base.query.v1beta1.PageResponse pagination = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| txs | list of [`Tx`](#txsvc-tx) | In the response. The block's transactions. |
| block_id | `BlockID` | In the response. The block's hash. |
| block | block | In the response. The block itself, in the consensus engine's own form. |

<a id="txsvc-query-tx-decode"></a>

#### TxDecode

Turns transaction bytes into the JSON form. Use it to check what a signed transaction contains before you broadcast it.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/TxDecode` |
| REST | `POST /cosmos/tx/v1beta1/decode` |
| urid | `urid tx decode [protobuf-byte-string]` |

```proto
message TxDecodeRequest {
  bytes tx_bytes = 1;
}

message TxDecodeResponse {
  cosmos.tx.v1beta1.Tx tx = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| tx_bytes | bytes, base64 in JSON | The encoded transaction. |
| tx | [`Tx`](#txsvc-tx) | In the response. The transaction as JSON. |

<a id="txsvc-query-tx-encode"></a>

#### TxEncode

Turns a transaction in JSON form into the bytes that BroadcastTx takes. Signatures are made over these bytes, so a client that builds JSON can use this to get them.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/TxEncode` |
| REST | `POST /cosmos/tx/v1beta1/encode` |
| urid | `urid tx encode [file]` |

```proto
message TxEncodeRequest {
  cosmos.tx.v1beta1.Tx tx = 1;
}

message TxEncodeResponse {
  bytes tx_bytes = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| tx | [`Tx`](#txsvc-tx) | The transaction as JSON. |
| tx_bytes | bytes, base64 in JSON | In the response. The encoded transaction. |

<a id="txsvc-query-tx-encode-amino"></a>

#### TxEncodeAmino

Turns a transaction in the legacy JSON format into legacy bytes. The call exists, but it is of no use on UriNet. Legacy JSON signing is refused, so a transaction in this format cannot be sent.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/TxEncodeAmino` |
| REST | `POST /cosmos/tx/v1beta1/encode/amino` |
| urid | none |

```proto
message TxEncodeAminoRequest {
  string amino_json = 1;
}

message TxEncodeAminoResponse {
  bytes amino_binary = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| amino_json | string | The transaction in legacy JSON. |
| amino_binary | bytes, base64 in JSON | In the response. The legacy bytes. |

<a id="txsvc-query-tx-decode-amino"></a>

#### TxDecodeAmino

Turns legacy bytes back into legacy JSON. Like [TxEncodeAmino](#txsvc-query-tx-encode-amino), it exists but is of no use on UriNet.

| | |
|---|---|
| gRPC | `cosmos.tx.v1beta1.Service/TxDecodeAmino` |
| REST | `POST /cosmos/tx/v1beta1/decode/amino` |
| urid | none |

```proto
message TxDecodeAminoRequest {
  bytes amino_binary = 1;
}

message TxDecodeAminoResponse {
  string amino_json = 1;
}
```
| Field | Type | Meaning |
|---|---|---|
| amino_binary | bytes, base64 in JSON | The legacy bytes. |
| amino_json | string | In the response. The transaction in legacy JSON. |

<a id="std-node-service"></a>

## Node and chain information

<a id="nodeinfo-overview"></a>

### What it is

Two services tell you about the node you are talking to and the chain it follows. The node service reports the node's settings and the height it has reached. The chain information service reports blocks, validator sets, and whether the node is still catching up. Both answer on REST and gRPC, so you need no access to the node's RPC.

These are the first calls to make against a new node. [GetNodeInfo](#nodeinfo-query-get-node-info) gives the chain ID, which you need to sign. [Config](#nodeinfo-query-config) gives the node's minimum gas price, which sets your fee. [GetSyncing](#nodeinfo-query-get-syncing) tells you whether its answers are current. `urid status` gives the same facts from the node's RPC.

Answers reflect the latest committed block. A node may also drop old blocks to save space. The earliest height it still holds is in [Status](#nodeinfo-query-status) and GetSyncing.

<a id="nodeinfo-block"></a>

#### A block

```proto
message Block {
  Header header = 1;
  .tendermint.types.Data data = 2;
  .tendermint.types.EvidenceList evidence = 3;
  .tendermint.types.Commit last_commit = 4;
}

message Header {
  .tendermint.version.Consensus version = 1;
  string chain_id = 2;
  int64 height = 3;
  google.protobuf.Timestamp time = 4;
  .tendermint.types.BlockID last_block_id = 5;
  bytes last_commit_hash = 6;
  bytes data_hash = 7;
  bytes validators_hash = 8;
  bytes next_validators_hash = 9;
  bytes consensus_hash = 10;
  bytes app_hash = 11;
  bytes last_results_hash = 12;
  bytes evidence_hash = 13;
  string proposer_address = 14;
}
```
| Field | Type | Meaning |
|---|---|---|
| header | `Header` | The block's header, below. |
| data | block data | The block's transactions, as raw bytes in `txs`. Decode them with [TxDecode](#txsvc-query-tx-decode), or use [GetBlockWithTxs](#txsvc-query-get-block-with-txs). |
| evidence | evidence list | Evidence of misbehaviour included in this block. |
| last_commit | commit | The validators' signatures on the previous block. |
| header.version | versions | The block protocol and application versions the block was made with. |
| header.chain_id | string | The chain ID. |
| header.height | number | The block's height. |
| header.time | timestamp | The block's time. |
| header.last_block_id | `BlockID` | The hash of the previous block. |
| header.app_hash | bytes, base64 in JSON | A fingerprint of the chain's state after the previous block. |
| header.proposer_address | string | The consensus address of the validator who proposed the block, starting with `urivalcons1`. |
| other `*_hash` fields | bytes, base64 in JSON | Fingerprints of the block's parts, such as its transactions, validators, and evidence. |

<a id="nodeinfo-validator"></a>

#### A validator in the set

```proto
message Validator {
  string address = 1;
  google.protobuf.Any pub_key = 2;
  int64 voting_power = 3;
  int64 proposer_priority = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| address | address | The validator's consensus address, starting with `urivalcons1`. |
| pub_key | `Any` | The validator's consensus key. |
| voting_power | number | Its weight in agreeing on blocks. |
| proposer_priority | number | Its place in the turn order for proposing blocks. |

<a id="nodeinfo-queries"></a>

### Queries

The node service has two calls and the chain information service has nine.

| Query | What it answers |
|---|---|
| [Config](#nodeinfo-query-config) | The node's minimum gas price and storage settings |
| [Status](#nodeinfo-query-status) | The node's current height and state fingerprint |
| [GetNodeInfo](#nodeinfo-query-get-node-info) | The chain ID and the software versions |
| [GetSyncing](#nodeinfo-query-get-syncing) | Whether the node is still catching up |
| [GetLatestBlock](#nodeinfo-query-get-latest-block) | The latest block |
| [GetBlockByHeight](#nodeinfo-query-get-block-by-height) | The block at a height |
| [GetLatestValidatorSet](#nodeinfo-query-get-latest-validator-set) | The validators signing blocks now |
| [GetValidatorSetByHeight](#nodeinfo-query-get-validator-set-by-height) | The validators at a height |
| [ABCIQuery](#nodeinfo-query-abci-query) | A raw query passed straight to the chain |
| [GetLatestBlockResults](#nodeinfo-query-get-latest-block-results) | The outcome of the latest block |
| [GetBlockResults](#nodeinfo-query-get-block-results) | The outcome of the block at a height |

<a id="nodeinfo-query-config"></a>

#### Config

Returns settings of the node you asked. The minimum gas price is the one that matters most. A fee below gas limit times this price is refused by this node.

| | |
|---|---|
| gRPC | `cosmos.base.node.v1beta1.Service/Config` |
| REST | `GET /cosmos/base/node/v1beta1/config` |
| urid | `urid query node config` |

```proto
message ConfigRequest {}

message ConfigResponse {
  string minimum_gas_price = 1;
  string pruning_keep_recent = 2;
  string pruning_interval = 3;
  uint64 halt_height = 4;
}
```
| Field | Type | Meaning |
|---|---|---|
| minimum_gas_price | string | The lowest gas price this node accepts, as an amount and the denom `kwak`. |
| pruning_keep_recent | string | How many recent states the node keeps. |
| pruning_interval | string | How often the node drops old states. |
| halt_height | number | A height at which the operator set the node to stop. 0 means none. |

<a id="nodeinfo-query-status"></a>

#### Status

Returns the node's latest height and the fingerprints of that block. Use it to check that a node is up to date, or to compare two nodes.

| | |
|---|---|
| gRPC | `cosmos.base.node.v1beta1.Service/Status` |
| REST | `GET /cosmos/base/node/v1beta1/status` |
| urid | `urid query node status` |

```proto
message StatusRequest {}

message StatusResponse {
  uint64 earliest_store_height = 1;
  uint64 height = 2;
  google.protobuf.Timestamp timestamp = 3;
  bytes app_hash = 4;
  bytes validator_hash = 5;
}
```
| Field | Type | Meaning |
|---|---|---|
| earliest_store_height | number | The oldest height whose state this node still holds. |
| height | number | The latest height. |
| timestamp | timestamp | The time of that block. |
| app_hash | bytes, base64 in JSON | The fingerprint of the chain's state. |
| validator_hash | bytes, base64 in JSON | The fingerprint of the validator set. |

<a id="nodeinfo-query-get-node-info"></a>

#### GetNodeInfo

Returns who the node is and what software it runs. The `default_node_info.network` field is the chain ID, which every signature must include.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetNodeInfo` |
| REST | `GET /cosmos/base/tendermint/v1beta1/node_info` |
| urid | `urid query consensus comet node-info`, or `urid status` |

```proto
message GetNodeInfoRequest {}

message GetNodeInfoResponse {
  .tendermint.p2p.DefaultNodeInfo default_node_info = 1;
  VersionInfo application_version = 2;
}

message VersionInfo {
  string name = 1;
  string app_name = 2;
  string version = 3;
  string git_commit = 4;
  string build_tags = 5;
  string go_version = 6;
  repeated Module build_deps = 7;
  string cosmos_sdk_version = 8;
}

message Module {
  string path = 1;
  string version = 2;
  string sum = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| default_node_info | node info | The node's ID, its network address, its name (`moniker`), the consensus engine's version, and `network`, the chain ID. |
| application_version | `VersionInfo` | The `urid` build. `name`, `app_name`, `version`, and `git_commit` identify it. `build_tags`, `go_version`, and `cosmos_sdk_version` say how it was built. `build_deps` lists the libraries, each a `Module` with its `path`, `version`, and `sum`, a checksum. |

<a id="nodeinfo-query-get-syncing"></a>

#### GetSyncing

Tells you whether the node is still catching up with the network. While it is, its answers lag behind the chain, so use another node or wait.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetSyncing` |
| REST | `GET /cosmos/base/tendermint/v1beta1/syncing` |
| urid | `urid query consensus comet syncing` |

```proto
message GetSyncingRequest {}

message GetSyncingResponse {
  bool syncing = 1;
  int64 earliest_block_height = 2;
  int64 latest_block_height = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| syncing | boolean | True while the node is catching up. |
| earliest_block_height | number | The oldest block this node holds. |
| latest_block_height | number | The newest block this node holds. |

<a id="nodeinfo-query-get-latest-block"></a>

#### GetLatestBlock

Returns the latest committed block. Its header gives the current height and time.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetLatestBlock` |
| REST | `GET /cosmos/base/tendermint/v1beta1/blocks/latest` |
| urid | `urid query consensus comet block-latest` |

```proto
message GetLatestBlockRequest {}

message GetLatestBlockResponse {
  .tendermint.types.BlockID block_id = 1;
  .tendermint.types.Block block = 2;
  Block sdk_block = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| block_id | `BlockID` | The block's hash. |
| block | block | No longer used. Read `sdk_block`. |
| sdk_block | [`Block`](#nodeinfo-block) | The block, with the proposer written as a `urivalcons1` address. |

<a id="nodeinfo-query-get-block-by-height"></a>

#### GetBlockByHeight

Returns the block at one height. A height the node no longer holds returns an error.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetBlockByHeight` |
| REST | `GET /cosmos/base/tendermint/v1beta1/blocks/{height}` |
| urid | `urid query consensus comet block-by-height [height]`, or `urid query block --type=height [height]` |

```proto
message GetBlockByHeightRequest {
  int64 height = 1;
}

message GetBlockByHeightResponse {
  .tendermint.types.BlockID block_id = 1;
  .tendermint.types.Block block = 2;
  Block sdk_block = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height. |
| block_id | `BlockID` | In the response. The block's hash. |
| block | block | No longer used. Read `sdk_block`. |
| sdk_block | [`Block`](#nodeinfo-block) | In the response. The block. |

`urid query block --type=hash [hash]` finds a block by its hash instead, and `urid query blocks --query [query]` searches blocks by their events. Both read from the node's RPC.

<a id="nodeinfo-query-get-latest-validator-set"></a>

#### GetLatestValidatorSet

Lists the validators that sign blocks now, with their voting power. The validator set is the group of validators that take part in agreeing on blocks.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetLatestValidatorSet` |
| REST | `GET /cosmos/base/tendermint/v1beta1/validatorsets/latest` |
| urid | `urid query consensus comet validator-set` |

```proto
message GetLatestValidatorSetRequest {
  cosmos.base.query.v1beta1.PageRequest pagination = 1;
}

message GetLatestValidatorSetResponse {
  int64 block_height = 1;
  repeated Validator validators = 2;
  cosmos.base.query.v1beta1.PageResponse pagination = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| block_height | number | In the response. The height the set belongs to. |
| validators | list of [`Validator`](#nodeinfo-validator) | In the response. The validators. |

<a id="nodeinfo-query-get-validator-set-by-height"></a>

#### GetValidatorSetByHeight

Lists the validators that signed blocks at one height. Use it to see who was in the set when something happened.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetValidatorSetByHeight` |
| REST | `GET /cosmos/base/tendermint/v1beta1/validatorsets/{height}` |
| urid | `urid query consensus comet validator-set-by-height [height]` |

```proto
message GetValidatorSetByHeightRequest {
  int64 height = 1;
  cosmos.base.query.v1beta1.PageRequest pagination = 2;
}

message GetValidatorSetByHeightResponse {
  int64 block_height = 1;
  repeated Validator validators = 2;
  cosmos.base.query.v1beta1.PageResponse pagination = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height. |
| pagination | page request | Paged, see [Reading lists](#connect-conventions). |
| block_height | number | In the response. The height the set belongs to. |
| validators | list of [`Validator`](#nodeinfo-validator) | In the response. The validators. |

<a id="nodeinfo-query-abci-query"></a>

#### ABCIQuery

Passes a raw query straight to the chain, by path, and returns the raw answer. It is for tools that need low-level access or a proof of a stored value. For everyday reading, use the typed queries in this manual.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/ABCIQuery` |
| REST | `GET /cosmos/base/tendermint/v1beta1/abci_query` |
| urid | none |

```proto
message ABCIQueryRequest {
  bytes data = 1;
  string path = 2;
  int64 height = 3;
  bool prove = 4;
}

message ABCIQueryResponse {
  uint32 code = 1;
  reserved 2;
  string log = 3;
  string info = 4;
  int64 index = 5;
  bytes key = 6;
  bytes value = 7;
  ProofOps proof_ops = 8;
  int64 height = 9;
  string codespace = 10;
}

message ProofOps {
  repeated ProofOp ops = 1;
}

message ProofOp {
  string type = 1;
  bytes key = 2;
  bytes data = 3;
}
```
| Field | Type | Meaning |
|---|---|---|
| data | bytes, base64 in JSON | The query's input, encoded. |
| path | string | What to query. It must be a path the chain supports, starting with `app`, `custom`, `p2p`, or `store`. |
| height | number | The height to read at. 0 means the latest. |
| prove | boolean | True to ask for a proof with the answer. |
| code | number | In the response. 0 on success. |
| log, info | string | In the response. Text about the answer. |
| key, value | bytes, base64 in JSON | In the response. The key read and the value found. |
| index | number | In the response. The position of the key, when the answer gives one. |
| proof_ops | `ProofOps` | In the response. The proof, when one was asked for. It holds `ops`, a list of proof steps. Each step has a `type`, a `key`, and its `data`. |
| height | number | In the response. The height the answer comes from. |
| codespace | string | In the response. Where an error code comes from. |

<a id="nodeinfo-query-get-latest-block-results"></a>

#### GetLatestBlockResults

Returns the outcome of the latest block: each transaction's result, and the events the block itself emitted. Block-level events include slashing, jailing, and changes to the validator set, which no transaction carries.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetLatestBlockResults` |
| REST | `GET /cosmos/base/tendermint/v1beta1/block_results/latest` |
| urid | `urid query consensus comet block-results-latest` |

```proto
message GetLatestBlockResultsRequest {}

message GetLatestBlockResultsResponse {
  int64 height = 1;
  repeated .tendermint.abci.ExecTxResult txs_results = 2;
  repeated .tendermint.abci.Event finalize_block_events = 3;
  repeated .tendermint.abci.ValidatorUpdate validator_updates = 4;
  .tendermint.types.ConsensusParams consensus_param_updates = 5;
  bytes app_hash = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block's height. |
| txs_results | list of results | Each transaction's code, log, gas, and events, in block order. |
| finalize_block_events | list of events | The events of the block itself, outside any transaction. |
| validator_updates | list | Changes to validator voting power made by this block. |
| consensus_param_updates | [`ConsensusParams`](#consensus-parameters) | Consensus parameters changed by this block, if any. |
| app_hash | bytes, base64 in JSON | The fingerprint of the chain's state after this block. |

<a id="nodeinfo-query-get-block-results"></a>

#### GetBlockResults

Returns the outcome of the block at one height, in the same shape as [GetLatestBlockResults](#nodeinfo-query-get-latest-block-results). Use it to find what happened at the end of a block, such as a validator being jailed.

| | |
|---|---|
| gRPC | `cosmos.base.tendermint.v1beta1.Service/GetBlockResults` |
| REST | `GET /cosmos/base/tendermint/v1beta1/block_results/{height}` |
| urid | `urid query consensus comet block-results [height]`, or `urid query block-results [height]` |

```proto
message GetBlockResultsRequest {
  int64 height = 1;
}

message GetBlockResultsResponse {
  int64 height = 1;
  repeated .tendermint.abci.ExecTxResult txs_results = 2;
  repeated .tendermint.abci.Event finalize_block_events = 3;
  repeated .tendermint.abci.ValidatorUpdate validator_updates = 4;
  .tendermint.types.ConsensusParams consensus_param_updates = 5;
  bytes app_hash = 6;
}
```
| Field | Type | Meaning |
|---|---|---|
| height | number | The block height. In the response, the same. |
| txs_results | list of results | In the response. Each transaction's result. |
| finalize_block_events | list of events | In the response. The block's own events. |
| validator_updates | list | In the response. Changes to validator voting power. |
| consensus_param_updates | [`ConsensusParams`](#consensus-parameters) | In the response. Changed consensus parameters, if any. |
| app_hash | bytes, base64 in JSON | In the response. The state fingerprint after this block. |

---

<a id="ch-tools"></a>

# Tools around the chain

<a id="tools-admin-wallet"></a>

## The Admin Wallet

The Admin Wallet is a console for UriNet that runs in a web browser. It sits on top of the TypeScript SDK and the wallet core, described at the end of this chapter. It can send every message the chain allows and run the queries a node serves over REST.

Whoever runs it serves it as a web page. You open the page and enter the REST address of a node, such as `https://<node>:1317`, and connect. You then load or create accounts and pick the acting account, the account the window acts as. You can also enter the address of a sponsor service, so the window can offer sponsored fees.

Passkeys work through the browser's own authenticator, such as the phone or laptop you open the page on. One window is one person. Each window keeps its own keys, accounts and acting account for as long as its browser session lasts. To act as two people, such as Alice and her recovery factor Bob, open two windows. Anything one person hands another, like a partial signature or a recovery vote, is text you copy between the windows.

The page has seven areas.

- **Connection and acting account.** The top bar shows the node, the chain ID and the height, and holds the account selector.
- **Keys and identities.** You make and import keys, register passkeys and add accounts. It shows which keys cover the acting account's credentials.
- **Message composer.** You pick a module and a message, and fill a form built from the message's definition.
- **Signing panel.** You choose who signs with which keys, the sign mode and who pays. Then you build, simulate, sign and broadcast.
- **Queries and state diff.** You run any query and compare state before and after a transaction.
- **Chain status.** It shows the acting account's credentials, balances, status and roles. It also shows an inbox of things waiting for you, read from an indexer when one is connected.
- **Log console.** The bottom area records every connection, block, action and result, with its events.

Nine dedicated screens cover the flows that a plain form handles poorly: **Create account**, **Threshold collection**, **Recovery**, **Delegations**, **Tokens**, **Reputation**, **Governance**, **Staking** and **Fee grants**.

With the Admin Wallet you can create an account, add a passkey and collect signatures from several devices. You can run a recovery with each factor in its own window. You can give an agent a delegation, issue and manage a token, or submit and vote on a proposal. When the chain refuses something, the screen shows two lines. The first is the chain's own text exactly as received. The second is a plain line that says what was refused and where.

<a id="tools-sponsor"></a>

## The sponsor service

A sponsor service pays network fees for other people's transactions. An operator runs it with an account of its own that holds URI. Any account can be a sponsor, and the chain gives the service no special rights. The chain does not even know the service exists. It checks the signatures, the fee payer's balance, and the gas and fee rules, as for any transaction.

To have a transaction sponsored, you build it and sign it in DIRECT_AUX, which leaves the fee out of what you sign. You send the service your aux signer data, the signed body together with your account details. The service checks its policy and simulates the transaction to find the gas. It then sets itself as the fee payer, signs in DIRECT, and submits the transaction. It waits for the result and returns it to you.

The service can also relay. Relaying means it submits certain messages for you, signs as their submitter, and pays the fee. It relays account creation and the three recovery messages: starting a recovery and both kinds of vote. These messages carry their own proofs, so the relayer gains no authority. It cannot change the action you approved.

The operator controls the service with a policy of five switches.

- **Mode.** `follow_policy` applies the other switches. `approve_all` accepts every well-formed request and ignores them. `refuse_all` refuses everything.
- **Message types.** Sponsoring is turned on or off per message type. A type not listed is on.
- **Daily limit.** It caps how many transactions, and how much in fees, each account may have sponsored per calendar day in UTC.
- **Maximum fee.** It is the largest fee the service pays for one transaction.
- **Relay allow-list.** It lists the message types the service will relay.

The service speaks HTTP with JSON. Field names are in lowerCamelCase. Amounts, gas values and heights are strings of digits. Every answer allows any origin, so a web page can call it.

| | |
|---|---|
| `POST /v1/sponsor` | Sponsors a transaction you signed in DIRECT_AUX. |
| `POST /v1/relay` | Relays messages that carry their own proofs. |
| `GET /v1/policy` | Returns the current policy. |
| `PUT /v1/policy` | Replaces the policy with a full document and returns it. |
| `POST /v1/policy/reset` | Restores the service's starting policy and returns it. |
| `GET /v1/status` | Returns the sponsor's address, chain ID and balance, the request counts, today's usage per account, and the recent decisions. `?limit=N` keeps the last N decisions. |
| `GET /healthz` | Answers `{"status":"ok"}` while the service runs. |

A sponsoring request holds the aux signer data of every account that signs a message, each in protobuf JSON.

```json
{
  "auxSignerData": [
    {
      "address": "uri1...",
      "signDoc": {"bodyBytes": "CoYB...", "chainId": "<chain-id>", "accountNumber": "11", "sequence": "4"},
      "mode": "SIGN_MODE_DIRECT_AUX",
      "sig": "CmgK..."
    }
  ]
}
```

A relaying request holds only the messages, each with its `@type`. Leave `submitter` empty, or set it to the service's own address.

```json
{"messages": [{"@type": "/urinet.smartaccount.v1.MsgStartRecovery", "account": "uri1...", "recoveryFactorId": "1", "recoveryProof": {"publicKeySignature": "..."}}]}
```

Both answer with status 200 and one of two bodies. An accepted answer has `"status": "accepted"`. It carries `feePayer`, the `fee`, the `txhash`, the gas used, and an `outcome` with the block result: its `status`, `height`, `code`, `codespace`, `log` and `events`. The outcome status is `committed`, `failed`, `rejected` or `pending`. A refused answer has `"status": "refused"` with a `stage`, a `code` and a `reason`. Nothing was submitted, so the wallet can pay the fee itself instead. When the node's simulation failed, `error` holds the chain's refusal: its `name`, `codespace`, `code`, `reason` and `message`.

The policy document looks like this.

```json
{
  "mode": "follow_policy",
  "messageTypes": {"/cosmos.gov.v1.MsgVote": false},
  "dailyLimit": {"count": 100, "fees": {"denom": "kwak", "amount": "1000000000"}},
  "maxFeePerTx": {"denom": "kwak", "amount": "10000000"},
  "relayAllowlist": ["/urinet.smartaccount.v1.MsgCreateSmartAccount", "/urinet.smartaccount.v1.MsgStartRecovery"]
}
```

The numbers are only an example. Read the policy a service actually runs with `GET /v1/policy`.

Most other statuses carry an error body.

```json
{"error": {"code": "invalid_request", "message": "auxSignerData is missing", "submitted": "no"}}
```

`submitted` is `"no"` when nothing was sent to the chain. It is `"unknown"` when the service broadcast the transaction but lost the node's answer. In that case do not pay again blindly. Look the transaction up by its hash first, if the answer gives one.

<a id="tools-indexer"></a>

## The indexer

An indexer keeps the chain's history in a form that is easy to search. It reads every block and its results from the node and records each event. It serves that record over HTTP as JSON. It only reads, and it never sends anything to the chain. Every answer allows any origin.

| | |
|---|---|
| `GET /health` | The service is up, with the chain ID and the last height it has read. |
| `GET /events` | Searches events. Filter by `from_height`, `to_height`, `tx_hash`, `type`, `attr_key` and `attr_value`. |
| `GET /txs/{hash}` | One transaction's result and its events. |
| `GET /accounts/{address}/history` | Every event that names the account, with the role it plays in each. |
| `GET /accounts/{address}/recovery` | The account's recovery sessions and recovery policy changes. |
| `GET /accounts/{address}/credentials` | The account's credential changes. |
| `GET /accounts/{address}/owner` | The account's Owner changes. |
| `GET /accounts/{address}/delegations/received` | The delegations the account received, each with its later suspensions, resumptions, revocation or expiry. |
| `GET /inbox` | Items waiting for an account or a key. Give `?address=` or `?public_key=` with the key in hex. |

The inbox lists four kinds of item. They are a recovery started in which you are a factor, a change of vote in such a recovery, a delegation you received, and a proposal open for voting. `resolved=true` or `resolved=false` keeps only finished or open items.

Lists come in pages. `limit` sets the page size, 100 by default and at most 1000. Each answer carries `next_cursor`, and you pass it back as `cursor` for the next page. An empty `next_cursor` means the last page. The account and inbox routes also take `order`, `desc` for newest first, the default, or `asc`. To follow new entries as they arrive, ask again with `order=asc` and the last entry's id as the cursor.

A bad parameter answers 400 and an unknown transaction 404, each with a body like `{"error": "limit must be a positive integer"}`.

The indexer records what happened. It does not serve current values. Read balances, locked amounts, reputation scores and the current status of a delegation from the chain's own queries.

<a id="tools-sdk"></a>

## The TypeScript SDK and wallet core

Two TypeScript packages help you build a wallet or an app for UriNet. They rely only on pure JavaScript libraries and on `fetch`, so they work in browsers, in Node and in React Native. The Admin Wallet is built on them.

The SDK speaks the protocol. It talks to a node over REST only. It pins the reads of one view to a single block height, so the numbers you show always agree. It builds transactions with their exact bytes and signs as a SmartAccount, with KEYPAIR keys and passkeys alike. It signs both an account's creation and its everyday transactions. It estimates gas by simulating the transaction and adds 30 percent. It submits, waits for the result, and decodes the result, its events and any error. It shows a refusal as two lines: the chain's exact text and a plain line.

The wallet core makes the wallet's decisions. It decides which account acts, which keys sign and who pays the fee. It checks that a transaction is ready before anyone signs. It runs the flows: creating an account, changing credentials, recovery, delegation, tokens, staking, governance and fee grants. The app that hosts it supplies five pieces: key storage, the signer for each credential, passkey access, an indexer connection and a sponsor connection.

The wallet core chooses the fee path with one rule. When a sponsor is set, it asks the sponsor first. If the sponsor refuses, it signs the transaction again and pays the fee itself. If the sponsor's answer is unknown, it stops, because it cannot tell whether the transaction went through. Without a sponsor, the account pays its own fee. A delegator pays only when you choose that path for a delegation.

---

<a id="ch-appendix"></a>

# Appendices

<a id="appendix-glossary"></a>

## Glossary

**Account.** The holder of coins and tokens on the chain, known by an address that starts with `uri1`. Every account that signs is a SmartAccount, explained in [Accounts](#ch-smartaccount).

**Account type.** A label on a SmartAccount: `HUMAN`, `AGENT`, `ORGANIZATION`, or `SERVICE`. All four work the same way. The label is only for display, regulation, and search.

**Address.** The permanent identity of an account. It never changes, even when every credential on the account is replaced.

**Admin Wallet.** A browser console in which you compose any UriNet message, sign it, send it, and read its result and events. It also runs any query a node serves over REST. See [Tools](#ch-tools).

**Allowed messages.** The fixed list of 77 message types the chain accepts. Any other message is refused before the fee is taken, and 18 of the 77 run only inside governance proposals.

**Allowlist.** A token's list of accounts that may receive it when its transfer restriction is allowlist only. A SEND rule can carry its own allowlist, a fixed set of recipients the delegate may pay.

**Approval threshold.** The least number of distinct credentials that must sign for an account, stored as `min_approvals` and also called its minimum approvals. An account with three credentials and a threshold of 2 is a 2-of-3 account.

**Approved transfer.** A move of a token whose transfer restriction is approval required. It needs the holder's authorization, or a delegate's, and an approval from the issuer or a holder of the approve transfer permission. See [Tokens](#ch-token).

**Auth Policy version.** A counter, `auth_version`, that rises by one each time a credential is added or removed or the approval threshold changes.

**Blocklist.** A token's list of accounts that may not send or receive that token in the normal way. An account on both the blocklist and the allowlist is treated as blocked.

**Bond denom.** The denom used for staking, which on UriNet is `kwak`. Network fees are accepted only in this denom, and governance cannot change it.

**Bootstrap key.** The key whose public key gives an account its address and which signs the account's creation. It is a secp256k1 or P-256 key, and its special role ends once the account is created.

**Burn.** Destroying tokens from your own balance, which lowers the total supply. Nobody can burn another account's balance directly.

**Cancel vote.** A recovery factor's vote to stop a recovery. When cancel votes reach the recovery threshold, the recovery ends and nothing changes.

**Change rule.** The direction in which a token setting may change later: fixed, disable only, enable only, increase only, decrease only, tighten only, relax only, or free. The issuer picks it when creating the token, and nobody can change it afterwards.

**Cleanup.** The removal of vesting releases that have already unlocked, and of schedules left empty. It happens before an account's vesting changes and at the end of the block in which a release matures.

**Committee.** The single SmartAccount that adds and removes reputation updaters. See [Reputation](#ch-reputation).

**Credential.** A public key, with its identifier, that may sign for an account. A `WEBAUTHN` credential is a passkey on the P-256 curve, and a `KEYPAIR` credential is an ordinary key pair on P-256 or secp256k1.

**Delegate.** The account that acts under a delegation, with its own credentials and its own sequence. Anything it sends under the delegation leaves the delegator's balance.

**Delegated vesting credit.** A bookkeeping entry that keeps locked URI under its lock while it is staked. It is not a coin, and it has nothing to do with a delegation between accounts.

**Delegation.** A record in which one account, the delegator, lets another account, the delegate, use the delegator's assets within set rules. It lasts at most 90 days and cannot be edited, only revoked and replaced. See [Delegation](#ch-delegation).

**Delegation ID.** The number of a delegation, `delegation_id`, given in order from 1 across the whole chain. A number is never reused.

**Delegation reference.** A pointer, `DelegationRef`, to a delegation and a permission inside it. A delegate gives it when it moves the delegator's tokens in an approved transfer.

**Delegator.** The account that grants a delegation and whose assets the delegate uses.

**Delegator-funded fee.** A fee taken from the delegator's balance under a delegation. The transaction names the delegator as fee granter and uses exactly one delegation. The delegation's `max_fee_total` caps these fees, and `fee_spent_total` shows what has been charged.

**Denom.** The machine name of a coin or token, such as `kwak` or `token/<issuer address>/<subdenom>`. People see a display name and a symbol, but the chain always uses the full denom.

**Destination vesting.** The input to a forced transfer that decides how the moved tokens vest at the recipient. Left out, the conditions are kept: what was taken from locked releases stays locked with the same unlock times, and the rest arrives unlocked. Present with no schedules, everything is usable at once. Present with schedules, those are attached as new.

**Event.** A typed record with named attributes that a transaction or a block leaves in its result. See [Events](#appendix-events).

**Evidence hash.** A hash of the evidence behind an account's reputation score. The evidence itself stays outside the chain, and the hash's largest allowed length is a reputation parameter.

**Expedited proposal.** A governance proposal sent with `MsgSubmitExpeditedProposal`, whose submitter picks a voting period from 30 seconds to 1 day. The shorter the period, the larger the deposit, and the whole deposit is paid at submission. See [Governance](#ch-gov).

**Expiry.** The moment, `expires_at`, at which a delegation's authority ends. The record is removed later, but the authority never outlasts the expiry time.

**Fee.** What a transaction pays the network for its gas, always in `kwak`. You pay it yourself, through a fee grant, from a delegator's funds under a delegation, or through a sponsor. See [How transactions work](#how-transactions).

**Fee grant.** An allowance in which one account, the granter, pays another account's fees. The account that uses the allowance is the grantee.

**Fee payer.** The account whose balance pays a transaction's fee.

**Forced transfer.** A move of a holder's tokens without the holder's signature. The issuer or a holder of the forced transfer permission runs it. It ignores the holder's freeze, the issuer pause, and the transfer restriction. It never takes tokens from the issuer, and a governance hard pause stops it.

**Free amount.** Your balance minus the part that vesting still locks. Subtract any frozen amount as well and you get the usable amount.

**Freeze.** An amount of an account's tokens, `frozen_amount`, that the issuer has made unusable. It may exceed the balance, and then tokens that arrive later are frozen too.

**Governance.** The way the chain changes its own rules. Holders of staked URI vote on proposals, and a passed proposal runs the messages it carries. See [Governance](#ch-gov).

**Governance account.** The account that signs a proposal's messages when the proposal passes. Governance-only messages name it as their `authority`.

**Governance hard pause.** A pause on one token, `paused_by_gov`, that only governance turns on and off. While it is on, every movement and change of that token stops, forced transfers included, but queries still work.

**Holder count.** The number of accounts with a positive balance of a token, leaving out accounts the chain itself owns. The max holder count caps it, and the chain checks the cap once, when each transaction ends.

**Holding limit per account.** The most of a token that one account may hold after receiving it.

**Indexer.** A service that copies the chain's events into a database and answers questions about history, such as an account's past changes. For current values, ask the chain. See [The indexer](#tools-indexer).

**Initial allocation.** The amounts handed out when a token is created, each of which may carry vesting schedules. Its `unlocked_amount` is the part that is free from the start.

**Issuer.** The account that created a token. It holds the root authority over the token forever, but it still cannot break the token's settings, change rules, or permanent locks.

**Issuer account protection.** The rule that the issuer can never be frozen, blocklisted, or made the source of a forced transfer. Only the issuer can change its own minter record.

**Issuer pause.** A pause, `paused_by_issuer`, that the issuer or a holder of the pause permission sets on a token. Ordinary movements stop, but forced transfers and management operations still work.

**kwak.** The base unit of URI. 1 URI is 1,000,000,000 kwak, and amounts on the wire are whole numbers of kwak written as strings.

**Locked amount.** The part of a balance that vesting has not yet unlocked at the current block time. Locked coins cannot be sent.

**Max supply.** The largest amount of a token that may exist at one time. Burning makes room to mint the same amount again.

**Message.** One request inside a transaction, such as `MsgSend`. Its full type name starts with a slash, as in `/urinet.token.v1.MsgMint`.

**Meter.** The running record on a SEND rule of how much has been sent, `spent_total`, and how many times, `use_count`. A refund does not lower it, because it measures authority used.

**Mint allowance.** The amount a minter may still create. A minter keeps its minter record even when the allowance is 0.

**Minter.** An account with a minter record for a token, which lets it mint up to its allowance. Even the issuer needs its own minter record to mint.

**Non-retroactive rule.** When governance lowers a limit, what already exists stays valid. The lower limit applies only to what is added or increased afterwards.

**Owner.** One SmartAccount recorded as the owner of another, in `owner_account`. Being the Owner gives no signing power, no delegation, and no recovery vote, and an account may have no Owner at all.

**Parameters.** A module's chain-wide settings, which governance replaces as a whole with the module's `MsgUpdateParams`. Each module's Params query shows the current values.

**Permanent lock.** A switch that fixes a token setting's current value forever. Only the issuer can turn it on, and nobody can turn it off.

**Permission (delegation).** One entry in a delegation. It holds a description, rules in plain language for people and agents, and SEND rules that the chain enforces.

**Permissions (token).** Nine roles that a token's root hands to other accounts: admin, manage minters, burn, pause, freeze and blocklist, forced transfer, update metadata, manage allowlist, and approve transfer. A permission says who may act, and a setting says whether the action is possible now.

**Proposal.** A governance proposal, which holders of staked URI vote on. Do not confuse it with a recovery proposal.

**Recovery.** The emergency path that installs approved new credentials on an account whose keys are lost or stolen. It never brings an old key back, and it cannot move assets or change the Owner. See [Accounts](#ch-smartaccount).

**Recovery factor.** A participant with one vote in an account's recovery, either another SmartAccount or a public key. An account can never be its own recovery factor.

**Recovery lock.** The `RECOVERY_LOCKED` status of an account while a recovery runs. The account cannot sign normally, send assets out, or receive from other users until the recovery ends.

**Recovery policy change.** A change to an account's recovery factors or recovery threshold made in normal times. It waits out a safety delay of 7 days before it applies, and only one can be pending at a time.

**Recovery proposal.** The exact change that a recovery factor proposes, which counts as that factor's vote. The recovery succeeds when the votes for one proposal reach the recovery threshold.

**Recovery session.** The running record of one recovery, opened when a recovery factor starts it. It fixes the factor list, the threshold, and the voting period at the start and counts votes against them.

**Recovery threshold.** The number of votes, `recovery_min_approvals`, that a recovery proposal or a cancel needs. It must be more than half the active recovery factors, and an account with no factors cannot be recovered.

**Relayed creation.** Creating a SmartAccount through another, already initialized SmartAccount that submits `MsgCreateSmartAccount` for it. The new address need not exist on the chain beforehand.

**Relayer.** An account that submits a transaction on another account's behalf. Submitting gives the relayer no authority over what it carries.

**Release.** One entry in a vesting schedule, made of an unlock time and an amount. It unlocks once the block time reaches its unlock time.

**Reputation record.** An account's current reputation entry, a score and an evidence hash written by an updater. An account with no record has no score, while a score of 0 is a real score.

**Revoke.** Ending a delegation for good. Either the delegator or the delegate can do it, and the chain then deletes the delegation.

**Root.** The top authority over a token, which its issuer holds forever. No grant or revoke of permissions can take it away.

**Schedule ID.** The number of a vesting schedule within one account, `schedule_id`, counted from 1. A number is never reused.

**SDK.** The UriNet client library. It connects to a node, reads state, builds and signs transactions as a SmartAccount, sends them, and decodes results, events, and errors. See [Tools](#ch-tools).

**Self-funded creation.** Creating a SmartAccount by submitting `MsgCreateSmartAccount` from the new account itself. The address must already exist on the chain, for example because it has received coins.

**SEND rule.** The chain-enforced rule in a delegation that lets the delegate send the delegator's coins of one denom. It sets the recipients and a required total limit, `max_total`, and may also limit each message, `max_per_tx`, and the number of uses, `max_uses`.

**Sequence.** A counter on each account that orders its transactions and stops one from being run twice. A SmartAccount has exactly one sequence.

**Settings (token).** Nine values that define a token: mint enabled, max supply, pause enabled, freeze enabled, forced transfer enabled, burn enabled, transfer restriction, holding limit per account, and max holder count. When a setting is off, nobody can use that feature, not even the issuer.

**Sign mode.** The way the bytes a signer signs are built. UriNet accepts only `SIGN_MODE_DIRECT` and `SIGN_MODE_DIRECT_AUX`, and refuses legacy JSON signing. DIRECT_AUX is for a signer who does not pay the fee. In a sponsored transaction every message signer uses it.

**Signature bundle.** The set of credential proofs that one SmartAccount gives in one transaction. A repeated credential or a single bad proof fails the whole transaction, and a transaction carries at most 20 proofs.

**SmartAccount.** The record that holds an account's credentials, approval threshold, Owner, recovery policy, and status. An address without one can receive coins but cannot sign, send, delegate, or recover. See [Accounts](#ch-smartaccount).

**Sponsor.** An account that pays the fee of someone else's transaction by signing as its fee payer. Any account can be a sponsor.

**Sponsor service.** A service that pays the fee for transactions signed with `SIGN_MODE_DIRECT_AUX`. It also relays messages that carry their own proofs. The chain treats its transactions like any other. See [Tools](#ch-tools).

**Staking delegation.** URI bonded to a validator, which is unrelated to a delegation between accounts. Unbonding it takes 21 days.

**Storage gas.** Extra gas charged when an operation creates new state or makes stored data larger. Shrinking or deleting data costs nothing and earns no refund.

**Subdenom.** The short name that an issuer picks for a token, the last part of its denom.

**Suspend.** Pausing a delegation so that it cannot be used until the delegator resumes it. A recovery that replaces an account's credentials suspends every active delegation that account gave.

**Total supply.** The amount of a token that exists right now.

**Transaction.** A signed bundle of one or more messages that an account sends to the chain. See [How transactions work](#how-transactions).

**Transfer restriction.** A token setting with three values, from loosest to strictest: `FREE`, `ALLOWLIST_ONLY`, and `APPROVAL_REQUIRED`.

**Updater.** An account that the committee allows to write reputation records. The set of updaters may be empty.

**URI.** The chain's native coin, used for staking and fees. It is not a user token, so it has no token settings, permissions, or freezes.

**Usable amount.** The free amount minus any frozen amount, which is what you can actually move.

**User token.** A token created with the token module, with a denom of the form `token/<issuer address>/<subdenom>`. See [Tokens](#ch-token).

**Vesting.** A lock that keeps part of a balance unusable until set times. See [Vesting](#ch-vesting).

**Vesting schedule.** A list of releases for one denom on one account, with unlock times that strictly increase. When a schedule is attached, every unlock time must lie in the future.

**Vesting-capable account.** The basic account record, of type `UriVestingAccount`, that every user account has from the start. It can hold vesting schedules, and `urid query auth account` shows it apart from the SmartAccount record.

**VOID.** The outcome of a recovery session that reaches its deadline without a decision. Nothing changes, and the account returns to its normal status.

**Wallet core.** The library, built on the SDK, that makes a wallet's decisions: which account acts, which keys sign, who pays, and what a result means. The Admin Wallet runs on it. See [Tools](#ch-tools).

---

<a id="appendix-events"></a>

## Events

An event is a typed record that a transaction leaves in its result. It has a type and named attributes, and it tells you exactly what changed. You see events in the output of `urid query tx [hash]`, in the transaction service's [GetTx](#txsvc-query-get-tx), and through the [indexer](#tools-indexer).

The type of a UriNet event is its full name, such as `urinet.token.v1.EventTokenMinted`. Each attribute is one field of the event, and its value is written as JSON. A few events come from the end of a block rather than from a transaction. At the end of a block, the chain applies recovery policy changes that are due and closes recovery sessions past their deadline. It also removes expired delegations and matured vesting releases, and runs passed proposals. Those events appear in the block's results, which [GetBlockResults](#nodeinfo-query-get-block-results) returns, and the indexer records them as well.

The standard modules emit their usual events next to these, such as `transfer`, `coin_spent`, `coin_received`, and `message`.

<a id="events-smartaccount"></a>

### SmartAccount events

A change carries the values after it, and a removal carries what was removed. An empty address means none.

| Event | When it is emitted | Attributes |
|---|---|---|
| `EventSmartAccountCreated` | A SmartAccount is initialized. | `address` new account, `type` account type, `name` display name, `metadata_uri` profile reference, `credentials` initial credentials, `min_approvals` approval threshold, `auth_version` starting version, `owner_account` initial Owner, `recovery_policy` initial recovery policy, `created_at` creation time |
| `EventAuthPolicyUpdated` | `MsgUpdateAuthPolicy` changes the credentials or the approval threshold. | `account` the account, `added_credentials` credentials added, `removed_credential_ids` credentials removed, `min_approvals` threshold after, `auth_version` version after |
| `EventSmartAccountProfileUpdated` | `MsgUpdateProfile` changes the profile. | `account` the account, `name` name after, `metadata_uri` reference after |
| `EventSmartAccountOwnerUpdated` | An account's Owner changes. | `account` the account, `old_owner` previous Owner, `new_owner` new Owner |
| `EventRecoveryPolicyChangeRequested` | A recovery policy change is requested in normal times and starts its safety delay. | `account` the account, `type` kind of change, `new_factor` factor to add, `recovery_factor_id` factor to remove, `new_recovery_min_approvals` threshold to apply, `requested_at` request time, `activate_at` when it applies, `delay_snapshot` delay applied |
| `EventRecoveryPolicyChangeCanceled` | `MsgCancelPendingRecoveryPolicyChange` deletes the pending change. The policy stays as it was. | `account` the account, `type` kind of change, `pending_change` deleted change, `recovery_policy_version` unchanged version |
| `EventRecoveryPolicyChangeActivated` | A pending change takes effect, at the end of the first block at or after its due time. | `account` the account, `type` kind of change, `added_factor` factor added, `removed_factor_id` factor removed, `recovery_min_approvals` threshold after, `recovery_policy_version` version after, `activate_at` due time |
| `EventRecoveryStarted` | `MsgStartRecovery` succeeds. The account becomes recovery locked, the session takes its snapshot, and any pending policy change is deleted. | `account` the account, `session_id` new session, `recovery_factor_id` starting factor, `started_at` start time, `deadline` session deadline, `recovery_policy_version` policy version, `recovery_factor_snapshot` voting factors, `recovery_min_approvals` recovery threshold, `recovery_voting_period` voting period, `pending_change_deleted` change deleted or not, `deleted_pending_change` the deleted change |
| `EventRecoveryFactorChoiceChanged` | A vote replaces a factor's current choice. A vote that changes nothing emits no event. | `account` the account, `session_id` the session, `recovery_factor_id` voting factor, `old_choice` previous choice, `old_proposal_hash` previous proposal hash, `new_choice` `CANCEL` or `PROPOSAL`, `new_proposal_hash` new proposal hash, `new_proposal` proposed change, `new_vote_nonce` factor's next nonce |
| `EventRecoverySucceeded` | A proposal reaches the recovery threshold and its changes are applied. The account returns to `NORMAL`. | `account` the account, `session_id` ended session, `proposal_hash` approved proposal hash, `proposal` applied change, `auth_version` version after, `recovery_policy_version` version after |
| `EventRecoveryCanceled` | Cancel votes reach the recovery threshold. The account returns to `NORMAL` and nothing else changes. | `account` the account, `session_id` ended session |
| `EventRecoveryVoided` | A session that never reached its threshold closes as VOID, at the end of the first block at or after its deadline. | `account` the account, `session_id` ended session, `deadline` the deadline |

<a id="events-delegation"></a>

### Delegation events

| Event | When it is emitted | Attributes |
|---|---|---|
| `EventDelegationCreated` | `MsgCreateDelegation` creates a delegation. | `delegation` the whole record |
| `EventDelegationSuspended` | An active delegation is suspended, by `MsgSuspendDelegation` or by a recovery that changes the delegator's credentials. | `delegation_id` the delegation, `delegator` its delegator, `delegate` its delegate |
| `EventDelegationResumed` | `MsgResumeDelegation` resumes a delegation. | `delegation_id` the delegation, `delegator` its delegator, `delegate` its delegate |
| `EventDelegationRevoked` | `MsgRevokeDelegation` ends a delegation. | `delegation_id` the delegation, `delegator` its delegator, `delegate` its delegate, `signer` who revoked it |
| `EventDelegationExpired` | An expired delegation is removed at the end of a block. Its authority ended at its expiry, which may be earlier. | `delegation_id` the delegation, `delegator` its delegator, `delegate` its delegate, `expires_at` when authority ended |
| `EventDelegatedSendExecuted` | A `MsgDelegatedSend` succeeds. | `delegation_id` delegation used, `permission_index` permission used, `delegator` source of coins, `delegate` the signer, `to_address` the recipient, `amount` coins sent |
| `EventDelegationUsageConsumed` | A SEND rule's meter is charged, by a delegated send or by an Approved Transfer that uses a delegation. One event per meter. | `delegation_id` delegation used, `permission_index` permission used, `rule_index` SEND rule used, `denom` the rule's denom, `amount` amount used, `spent_total` meter total after, `use_count` uses after, present only when the rule limits uses |
| `EventDelegationFeeCharged` | A fee is taken from the delegator's balance under a delegation, before the messages run. | `delegation_id` delegation used, `delegator` who paid, `delegate` the signer, `fee` fee charged, `fee_spent_total` fee meter after |

<a id="events-token"></a>

### Token events

A token event is emitted only when something actually changes. `MsgUpdateParams` emits none. Text fields such as the description and the data on a list entry are not copied into events.

| Event | When it is emitted | Attributes |
|---|---|---|
| `EventTokenCreated` | `MsgCreateToken` creates a token. | `denom` full denom, `issuer` the issuer, `subdenom` short name, `name` token name, `symbol` token symbol, `decimals` display decimals, `initial_supply` starting supply |
| `EventInitialAllocation` | Token creation pays an initial amount. One per recipient, none when the initial supply is 0. | `denom` the token, `recipient` who received, `total_amount` amount received, `unlocked_amount` part free at once, `vesting_principal` part under vesting, `vesting_schedule_count` schedules created |
| `EventTokenSettingUpdated` | A setting's value changes. One per setting. | `denom` the token, `setting_key` the setting, `old_value` value before, `new_value` value after, `sender` who changed it |
| `EventTokenSettingLocked` | The issuer permanently locks a setting. One per setting. | `denom` the token, `setting_key` the setting, `sender` the issuer |
| `EventTokenDescriptionUpdated` | The token's description changes. | `denom` the token, `sender` who changed it |
| `EventTokenDocumentReferenceUpdated` | The document reference or its hash changes. | `denom` the token, `sender` who changed it, `old_uri` reference before, `new_uri` reference after, `old_uri_hash` hash before, `new_uri_hash` hash after |
| `EventTokenPermissionGranted` | An account gains permissions, at creation or by a grant. One per account. | `denom` the token, `account` who gained them, `changed_permissions` permissions added, `sender` who granted them |
| `EventTokenPermissionRevoked` | An account loses permissions. One per account. | `denom` the token, `account` who lost them, `changed_permissions` permissions removed, `sender` who revoked them |
| `EventTokenMinterSet` | A Minter record is created or its allowance changes, at creation or later. | `denom` the token, `minter` the minter, `old_remaining` allowance before, `new_remaining` allowance after, `created` new record or not, `sender` who set it |
| `EventTokenMinterRemoved` | A Minter record is deleted. | `denom` the token, `minter` the minter, `last_remaining` final allowance, `sender` who removed it |
| `EventTokenIssuerPauseChanged` | The issuer pause turns on or off. | `denom` the token, `old_paused` state before, `new_paused` state after, `sender` who changed it |
| `EventTokenFrozenAmountChanged` | An account's frozen amount changes. | `denom` the token, `account` the holder, `old_frozen_amount` amount before, `new_frozen_amount` amount after, `sender` who changed it |
| `EventTokenBlocklistUpserted` | A blocklist entry is added or its data changes. One per entry. | `denom` the token, `account` listed account, `created` new entry or not, `sender` who listed it |
| `EventTokenBlocklistDeleted` | A blocklist entry is removed. One per entry. | `denom` the token, `account` removed account, `sender` who removed it |
| `EventTokenAllowlistUpserted` | An allowlist entry is added or its data changes, at creation or later. One per entry. | `denom` the token, `account` listed account, `created` new entry or not, `sender` who listed it |
| `EventTokenAllowlistDeleted` | An allowlist entry is removed. One per entry. | `denom` the token, `account` removed account, `sender` who removed it |
| `EventTokenMinted` | `MsgMint` creates new supply. It is the only record of the allowance change. | `denom` the token, `minter` the minter, `recipient` who received, `amount` amount minted, `old_remaining` allowance before, `new_remaining` allowance after |
| `EventTokenBurned` | `MsgBurn` destroys tokens. | `denom` the token, `burner` who burned, `amount` amount burned |
| `EventTokenApprovedTransfer` | `MsgApprovedTransfer` moves tokens. | `denom` the token, `source` tokens left here, `recipient` who received, `amount` amount moved, `asset_authorizer` holder or its delegate, `approver` who approved |
| `EventTokenForcedTransfer` | `MsgForcedTransfer` moves tokens. The lists are audit detail. | `sender` who forced it, `denom` the token, `source` tokens taken here, `recipient` who received, `amount` amount moved, `source_free_amount_used` taken from free amount, `reduced_source_releases` source releases reduced, `preserved_schedule_mappings` source to recipient schedules, `created_destination_schedule_ids` new recipient schedules |
| `EventTokenGovernanceHardPauseChanged` | The Governance Hard Pause turns on or off, when a passed proposal runs at the end of a block. | `denom` the token, `old_paused` state before, `new_paused` state after, `authority` governance account |

<a id="events-vesting"></a>

### Vesting events

| Event | When it is emitted | Attributes |
|---|---|---|
| `EventVestingScheduleAttached` | A new vesting schedule is stored on an account, for example from an initial allocation or a Forced Transfer. One per schedule, in number order. | `account` the account, `schedule_id` new schedule, `denom` its denom, `releases` stored releases |
| `EventVestingCleanup` | Matured releases are removed, before an account's vesting changes or at the end of the block in which they mature. Not emitted when nothing was removed. | `account` the account, `removed_releases` releases removed, `removed_schedule_ids` emptied schedules, `delegated_free_added` credit moved to `delegated_free` |
| `EventVestingReleasesReduced` | A Forced Transfer reduces the source's future releases of one denom. Not emitted when only the free amount was used. | `account` the source, `denom` the denom, `reductions` changed releases, `delegated_free_added` credit moved to `delegated_free`, `removed_schedule_ids` deleted schedules |
| `EventVestingSchedulesRejected` | A holder removes whole vesting schedules of a user token with `MsgRejectVestingSchedules`. Their locked coins are burned in the same transaction. | `account` the holder, `denom` the token, `schedule_ids` removed schedules, `locked_amount` amount burned |

Each entry of `removed_releases` gives `schedule_id`, `denom`, `unlock_time`, `amount`, and `delegated_vesting_credit`. Each entry of `reductions` gives `schedule_id`, `unlock_time`, `reduced_amount`, `remaining_amount`, `remaining_delegated_vesting_credit`, and `release_removed`.

<a id="events-reputation"></a>

### Reputation events

| Event | When it is emitted | Attributes |
|---|---|---|
| `EventReputationSet` | `MsgSetReputations` writes an account's record, even when the value is unchanged. One per account. | `account` the account, `score` new score, `evidence_hash` new evidence hash, `updater` the signer |
| `EventUpdaterAdded` | `MsgAddUpdater` adds an updater. | `updater` account added, `committee` the signer |
| `EventUpdaterRemoved` | `MsgRemoveUpdater` removes an updater. | `updater` account removed, `committee` the signer |
| `EventReputationParamsUpdated` | A passed proposal changes the reputation parameters with `MsgUpdateParams`. | `authority` governance account, `committee` new committee, `max_evidence_hash_bytes` new hash limit |

---

<a id="appendix-errors"></a>

## Error codes

When the chain refuses a transaction, the result carries three things: a codespace, a numeric code, and a message. The codespace names the part of the chain that refused, and the code identifies the error within that codespace. A code of 0 means the transaction succeeded. In `urid` output and in REST and gRPC responses, look at the `code`, `codespace`, and `raw_log` fields.

A sync broadcast that answers code 0 only means the node accepted the transaction. It has not run yet. The final code comes from [GetTx](#txsvc-query-get-tx), `urid query tx [hash]`, or `urid query wait-tx [hash]`.

The message contains the error's meaning from the tables below, usually with detail about your case. Wallets built on the [TypeScript SDK](#tools-sdk) show a refusal as two lines. The first is the chain's text exactly as received. The second is a plain line naming your request, the step where the chain refused it, and the error from these tables.

Codes within a codespace are not always consecutive. A number missing from a table is not used.

<a id="errors-urinet"></a>

### UriNet errors

These codespaces belong to the UriNet modules. The `urinet` codespace holds chain-wide refusals, such as a message type the chain does not accept or an expedited proposal outside its rules.

<a id="errors-cs-delegation"></a>

#### Codespace `delegation`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ERR_DELEGATION_NOT_FOUND` | Delegation not found. |
| 3 | `ERR_UNAUTHORIZED_DELEGATION_ACTOR` | Unauthorized delegation actor. |
| 4 | `ERR_INVALID_DELEGATION_STATE` | Invalid delegation state. |
| 5 | `ERR_INVALID_DELEGATION_CONFIG` | Invalid delegation config. |
| 6 | `ERR_INVALID_DELEGATION_ACTION` | Invalid delegation action. |
| 7 | `ERR_DELEGATION_RULE_VIOLATION` | Delegation rule violation. |
| 8 | `ERR_DELEGATION_FEE_LIMIT` | Delegation fee limit. |

<a id="errors-cs-reputation"></a>

#### Codespace `reputation`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrNotUpdater` | Not a reputation updater. |
| 3 | `ErrNotCommittee` | Not the reputation committee. |
| 4 | `ErrEmptyUpdates` | Empty reputation updates. |
| 5 | `ErrDuplicateReputationAccount` | Duplicate reputation account. |
| 6 | `ErrInvalidEvidenceHash` | Invalid reputation evidence hash. |
| 7 | `ErrUpdaterAlreadyExists` | Reputation updater already exists. |
| 8 | `ErrUpdaterNotFound` | Reputation updater not found. |
| 9 | `ErrInvalidParams` | Invalid reputation params. |

<a id="errors-cs-smartaccount"></a>

#### Codespace `smartaccount`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ERR_SMART_ACCOUNT_NOT_INITIALIZED` | Smart account not initialized. |
| 3 | `ERR_SMART_ACCOUNT_ALREADY_INITIALIZED` | Smart account already initialized. |
| 4 | `ERR_ACCOUNT_RECOVERY_LOCKED` | Account recovery locked. |
| 5 | `ERR_SMART_ACCOUNT_AUTH_FAILED` | Smart account auth failed. |
| 6 | `ERR_INVALID_SMART_ACCOUNT_CONFIG` | Invalid smart account config. |
| 7 | `ERR_INVALID_BOOTSTRAP_PROOF` | Invalid bootstrap proof. |
| 8 | `ERR_INVALID_CREDENTIAL_PROOF` | Invalid credential proof. |
| 9 | `ERR_INVALID_OWNER_TRANSITION` | Invalid owner transition. |
| 10 | `ERR_OWNERSHIP_CYCLE` | Ownership cycle. |
| 11 | `ERR_INVALID_RECOVERY_STATE` | Invalid recovery state. |
| 12 | `ERR_INVALID_RECOVERY_FACTOR` | Invalid recovery factor. |
| 13 | `ERR_INVALID_RECOVERY_PROOF` | Invalid recovery proof. |
| 14 | `ERR_INVALID_RECOVERY_PROPOSAL` | Invalid recovery proposal. |
| 15 | `ERR_INVALID_RECOVERY_POLICY` | Invalid recovery policy. |

<a id="errors-cs-token"></a>

#### Codespace `token`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrTokenNotFound` | Token not found. |
| 3 | `ErrTokenAlreadyExists` | Token already exists. |
| 4 | `ErrInvalidDenom` | Invalid token denom. |
| 5 | `ErrInvalidSubdenom` | Invalid token subdenom. |
| 6 | `ErrDenomStateConflict` | Token denom state conflict. |
| 7 | `ErrModuleAccountNotAllowed` | Module account not allowed for token operation. |
| 8 | `ErrMissingAmount` | Missing token amount. |
| 9 | `ErrInvalidAmount` | Invalid token amount. |
| 10 | `ErrSelfSend` | Token source and recipient are the same. |
| 11 | `ErrGovernancePaused` | Token governance hard paused. |
| 12 | `ErrIssuerPaused` | Token issuer paused. |
| 13 | `ErrInsufficientTokenUsable` | Insufficient token usable amount. |
| 14 | `ErrHoldingLimitExceeded` | Token holding limit exceeded. |
| 15 | `ErrDestinationBlocked` | Token destination blocklisted. |
| 16 | `ErrSourceBlocked` | Token source blocklisted. |
| 17 | `ErrRecipientNotAllowlisted` | Token recipient not allowlisted. |
| 18 | `ErrApprovalRequired` | Token transfer approval required. |
| 19 | `ErrMaxHolderCountExceeded` | Token max holder count exceeded. |
| 20 | `ErrMissingSettings` | Missing token settings. |
| 21 | `ErrInvalidSettings` | Invalid token settings. |
| 22 | `ErrInvalidLimitMode` | Invalid token limit mode. |
| 23 | `ErrInvalidName` | Invalid token name. |
| 24 | `ErrInvalidSymbol` | Invalid token symbol. |
| 25 | `ErrReservedSymbol` | Reserved token symbol. |
| 26 | `ErrInvalidDecimals` | Invalid token decimals. |
| 27 | `ErrInvalidInitialAllocation` | Invalid token initial allocation. |
| 28 | `ErrMaxSupplyExceeded` | Token max supply exceeded. |
| 29 | `ErrNativeMetadataUnavailable` | Native coin metadata unavailable. |
| 30 | `ErrBankMetadataInvalid` | Invalid token bank metadata. |
| 31 | `ErrBankMetadataNotFound` | Token bank metadata not found. |
| 32 | `ErrBankMetadataIdentityMismatch` | Token bank metadata identity mismatch. |
| 33 | `ErrInvalidDescription` | Invalid token description. |
| 34 | `ErrInvalidMetadataURI` | Invalid token metadata uri. |
| 35 | `ErrInvalidMetadataURIHash` | Invalid token metadata uri hash. |
| 36 | `ErrInvalidDocumentReference` | Invalid token document reference. |
| 37 | `ErrEmptyMetadataUpdate` | Empty token metadata update. |
| 38 | `ErrReservedName` | Reserved token name. |
| 39 | `ErrAmountOverflow` | Token amount overflow. |
| 40 | `ErrEmptySettingUpdate` | Empty token setting update. |
| 41 | `ErrInvalidSettingValue` | Invalid token setting value. |
| 42 | `ErrSettingPermanentlyLocked` | Token setting permanently locked. |
| 43 | `ErrSettingChangeNotAllowed` | Token setting change not allowed. |
| 44 | `ErrIssuerStillPaused` | Token issuer pause still active. |
| 45 | `ErrMaxSupplyBelowCurrentSupply` | Token max supply below current supply. |
| 46 | `ErrMaxHolderCountBelowCurrentCount` | Token max holder count below current holder count. |
| 47 | `ErrIssuerRequired` | Token issuer required. |
| 48 | `ErrEmptySettingKeys` | Empty token setting keys. |
| 49 | `ErrInvalidSettingKey` | Invalid token setting key. |
| 50 | `ErrDuplicateSettingKey` | Duplicate token setting key. |
| 51 | `ErrSettingAlreadyLocked` | Token setting already locked. |
| 52 | `ErrInvalidParams` | Invalid token params. |
| 53 | `ErrStorageGasOverflow` | Token storage gas overflow. |
| 59 | `ErrUnauthorized` | Token permission required. |
| 60 | `ErrEmptyPermissionList` | Empty token permission list. |
| 61 | `ErrDuplicatePermissionEntry` | Duplicate token permission entry. |
| 62 | `ErrInvalidPermission` | Invalid token permission. |
| 63 | `ErrPermissionTargetIsIssuer` | Token permission target is the issuer. |
| 64 | `ErrPermissionBatchTooLarge` | Token permission batch too large. |
| 66 | `ErrInvalidAdminSelfResignation` | Invalid token admin self-resignation. |
| 68 | `ErrNotMinter` | Not a token minter. |
| 69 | `ErrMintAllowanceExceeded` | Token mint allowance exceeded. |
| 70 | `ErrDuplicateMinterEntry` | Duplicate token minter entry. |
| 80 | `ErrPauseDisabled` | Token pause disabled. |
| 81 | `ErrFreezeDisabled` | Token freeze disabled. |
| 82 | `ErrRestrictionTargetIsIssuer` | Token restriction target is the issuer. |
| 83 | `ErrEmptyBatch` | Empty token list batch. |
| 84 | `ErrDuplicateBatchAccount` | Duplicate token list batch account. |
| 85 | `ErrBatchTooLarge` | Token list batch too large. |
| 86 | `ErrOpaqueValueTooLarge` | Token list data too large. |
| 100 | `ErrMintDisabled` | Token mint disabled. |
| 101 | `ErrBurnDisabled` | Token burn disabled. |
| 102 | `ErrNotApprovalMode` | Token not in approval mode. |
| 103 | `ErrMissingDelegation` | Token approved transfer missing delegation. |
| 104 | `ErrForcedTransferDisabled` | Token forced transfer disabled. |
| 105 | `ErrUnexpectedDelegation` | Token approved transfer unexpected delegation. |

<a id="errors-cs-urinet"></a>

#### Codespace `urinet`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrMsgNotAllowed` | Message not allowed. |
| 3 | `ErrImmutableProtocolParam` | Immutable protocol parameter. |
| 4 | `ErrExpeditedFlagNotAllowed` | Expedited flag not allowed. |
| 5 | `ErrExpeditedVotingPeriodOutOfRange` | Expedited voting period out of range. |

<a id="errors-cs-vesting"></a>

#### Codespace `vesting`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrInvalidVestingSelection` | Invalid vesting selection. |
| 3 | `ErrInvalidVestingSchedule` | Invalid vesting schedule. |
| 4 | `ErrVestingScheduleLimit` | Vesting schedule limit exceeded. |
| 5 | `ErrIncompatibleAccountType` | Incompatible vesting account type. |
| 6 | `ErrVestingInvariantViolation` | Vesting invariant violation. |
| 7 | `ErrInvalidParams` | Invalid vesting params. |
| 8 | `ErrInvalidVestingAttach` | Vesting schedules do not fit the newly received funds. |
| 9 | `ErrVestingScheduleIDOverflow` | Vesting schedule id overflow. |
| 10 | `ErrVestingStorageGasOverflow` | Vesting storage gas overflow. |

<a id="errors-standard"></a>

### Standard module errors

These codespaces belong to the standard modules. The `sdk` codespace covers refusals of the transaction as a whole, such as a wrong sequence, a fee that is too low, or running out of gas. The `undefined` codespace marks an internal failure on the node.

<a id="errors-cs-std-bank"></a>

#### Codespace `bank`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrNoInputs` | No inputs to send transaction. |
| 3 | `ErrNoOutputs` | No outputs to send transaction. |
| 4 | `ErrInputOutputMismatch` | Sum inputs != sum outputs. |
| 5 | `ErrSendDisabled` | Send transactions are disabled. |
| 6 | `ErrDenomMetadataNotFound` | Client denom metadata not found. |
| 7 | `ErrInvalidKey` | Invalid key. |
| 8 | `ErrDuplicateEntry` | Duplicate entry. |
| 9 | `ErrMultipleSenders` | Multiple senders not allowed. |

<a id="errors-cs-std-distribution"></a>

#### Codespace `distribution`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrEmptyDelegatorAddr` | Delegator address is empty. |
| 3 | `ErrEmptyWithdrawAddr` | Withdraw address is empty. |
| 4 | `ErrEmptyValidatorAddr` | Validator address is empty. |
| 5 | `ErrEmptyDelegationDistInfo` | No delegation distribution info. |
| 6 | `ErrNoValidatorDistInfo` | No validator distribution info. |
| 7 | `ErrNoValidatorCommission` | No validator commission to withdraw. |
| 8 | `ErrSetWithdrawAddrDisabled` | Set withdraw address disabled. |
| 9 | `ErrBadDistribution` | Community pool does not have sufficient coins to distribute. |
| 10 | `ErrInvalidProposalAmount` | Invalid community pool spend proposal amount. |
| 11 | `ErrEmptyProposalRecipient` | Invalid community pool spend proposal recipient. |
| 12 | `ErrNoValidatorExists` | Validator does not exist. |
| 13 | `ErrNoDelegationExists` | Delegation does not exist. |

<a id="errors-cs-std-feegrant"></a>

#### Codespace `feegrant`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrFeeLimitExceeded` | Fee limit exceeded. |
| 3 | `ErrFeeLimitExpired` | Fee allowance expired. |
| 4 | `ErrInvalidDuration` | Invalid duration. |
| 5 | `ErrNoAllowance` | No allowance. |
| 6 | `ErrNoMessages` | Allowed messages are empty. |
| 7 | `ErrMessageNotAllowed` | Message not allowed. |

<a id="errors-cs-std-gov"></a>

#### Codespace `gov`

| Code | Name | Meaning |
|---|---|---|
| 3 | `ErrInactiveProposal` | Inactive proposal. |
| 4 | `ErrAlreadyActiveProposal` | Proposal already active. |
| 5 | `ErrInvalidProposalContent` | Invalid proposal content. |
| 6 | `ErrInvalidProposalType` | Invalid proposal type. |
| 7 | `ErrInvalidVote` | Invalid vote option. |
| 8 | `ErrInvalidGenesis` | Invalid genesis state. |
| 9 | `ErrNoProposalHandlerExists` | No handler exists for proposal type. |
| 10 | `ErrUnroutableProposalMsg` | Proposal message not recognized by router. |
| 11 | `ErrNoProposalMsgs` | No messages proposed. |
| 12 | `ErrInvalidProposalMsg` | Invalid proposal message. |
| 13 | `ErrInvalidSigner` | Expected gov account as only signer for proposal message. |
| 15 | `ErrMetadataTooLong` | Metadata too long. |
| 16 | `ErrMinDepositTooSmall` | Minimum deposit is too small. |
| 18 | `ErrInvalidProposer` | Invalid proposer. |
| 20 | `ErrVotingPeriodEnded` | Voting period already ended. |
| 21 | `ErrInvalidProposal` | Invalid proposal. |
| 22 | `ErrSummaryTooLong` | Summary too long. |
| 23 | `ErrInvalidDepositDenom` | Invalid deposit denom. |

<a id="errors-cs-std-sdk"></a>

#### Codespace `sdk`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrTxDecode` | Tx parse error. |
| 3 | `ErrInvalidSequence` | Invalid sequence. |
| 4 | `ErrUnauthorized` | Unauthorized. |
| 5 | `ErrInsufficientFunds` | Insufficient funds. |
| 6 | `ErrUnknownRequest` | Unknown request. |
| 7 | `ErrInvalidAddress` | Invalid address. |
| 8 | `ErrInvalidPubKey` | Invalid pubkey. |
| 9 | `ErrUnknownAddress` | Unknown address. |
| 10 | `ErrInvalidCoins` | Invalid coins. |
| 11 | `ErrOutOfGas` | Out of gas. |
| 12 | `ErrMemoTooLarge` | Memo too large. |
| 13 | `ErrInsufficientFee` | Insufficient fee. |
| 14 | `ErrTooManySignatures` | Maximum number of signatures exceeded. |
| 15 | `ErrNoSignatures` | No signatures supplied. |
| 16 | `ErrJSONMarshal` | Failed to marshal JSON bytes. |
| 17 | `ErrJSONUnmarshal` | Failed to unmarshal JSON bytes. |
| 18 | `ErrInvalidRequest` | Invalid request. |
| 19 | `ErrTxInMempoolCache` | Tx already in mempool. |
| 20 | `ErrMempoolIsFull` | Mempool is full. |
| 21 | `ErrTxTooLarge` | Tx too large. |
| 22 | `ErrKeyNotFound` | Key not found. |
| 23 | `ErrWrongPassword` | Invalid account password. |
| 24 | `ErrorInvalidSigner` | Tx intended signer does not match the given signer. |
| 25 | `ErrorInvalidGasAdjustment` | Invalid gas adjustment. |
| 26 | `ErrInvalidHeight` | Invalid height. |
| 27 | `ErrInvalidVersion` | Invalid version. |
| 28 | `ErrInvalidChainID` | Invalid chain-id. |
| 29 | `ErrInvalidType` | Invalid type. |
| 30 | `ErrTxTimeoutHeight` | Tx timeout height. |
| 31 | `ErrUnknownExtensionOptions` | Unknown extension options. |
| 32 | `ErrWrongSequence` | Incorrect account sequence. |
| 33 | `ErrPackAny` | Failed packing protobuf message to Any. |
| 34 | `ErrUnpackAny` | Failed unpacking protobuf message from Any. |
| 35 | `ErrLogic` | Internal logic error. |
| 36 | `ErrConflict` | Conflict. |
| 37 | `ErrNotSupported` | Feature not supported. |
| 38 | `ErrNotFound` | Not found. |
| 39 | `ErrIO` | Internal IO error. |
| 40 | `ErrAppConfig` | Error in app.toml. |
| 41 | `ErrInvalidGasLimit` | Invalid gas limit. |
| 42 | `ErrTxTimeout` | Tx timeout. |

<a id="errors-cs-std-slashing"></a>

#### Codespace `slashing`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrNoValidatorForAddress` | Address is not associated with any known validator. |
| 3 | `ErrBadValidatorAddr` | Validator does not exist for that address. |
| 4 | `ErrValidatorJailed` | Validator still jailed; cannot be unjailed. |
| 5 | `ErrValidatorNotJailed` | Validator not jailed; cannot be unjailed. |
| 6 | `ErrMissingSelfDelegation` | Validator has no self-delegation; cannot be unjailed. |
| 7 | `ErrSelfDelegationTooLowToUnjail` | Validator's self delegation less than minimum; cannot be unjailed. |
| 8 | `ErrNoSigningInfoFound` | No validator signing info found. |
| 9 | `ErrValidatorTombstoned` | Validator already tombstoned. |

<a id="errors-cs-std-staking"></a>

#### Codespace `staking`

| Code | Name | Meaning |
|---|---|---|
| 2 | `ErrEmptyValidatorAddr` | Empty validator address. |
| 3 | `ErrNoValidatorFound` | Validator does not exist. |
| 4 | `ErrValidatorOwnerExists` | Validator already exist for this operator address; must use new validator operator address. |
| 5 | `ErrValidatorPubKeyExists` | Validator already exist for this pubkey; must use new validator pubkey. |
| 6 | `ErrValidatorPubKeyTypeNotSupported` | Validator pubkey type is not supported. |
| 7 | `ErrValidatorJailed` | Validator for this address is currently jailed. |
| 8 | `ErrBadRemoveValidator` | Failed to remove validator. |
| 9 | `ErrCommissionNegative` | Commission must be positive. |
| 10 | `ErrCommissionHuge` | Commission cannot be more than 100%. |
| 11 | `ErrCommissionGTMaxRate` | Commission cannot be more than the max rate. |
| 12 | `ErrCommissionUpdateTime` | Commission cannot be changed more than once in 24h. |
| 13 | `ErrCommissionChangeRateNegative` | Commission change rate must be positive. |
| 14 | `ErrCommissionChangeRateGTMaxRate` | Commission change rate cannot be more than the max rate. |
| 15 | `ErrCommissionGTMaxChangeRate` | Commission cannot be changed more than max change rate. |
| 16 | `ErrSelfDelegationBelowMinimum` | Validator's self delegation must be greater than their minimum self delegation. |
| 17 | `ErrMinSelfDelegationDecreased` | Minimum self delegation cannot be decreased. |
| 18 | `ErrEmptyDelegatorAddr` | Empty delegator address. |
| 19 | `ErrNoDelegation` | No delegation for (address, validator) tuple. |
| 20 | `ErrBadDelegatorAddr` | Delegator does not exist with address. |
| 21 | `ErrNoDelegatorForAddress` | Delegator does not contain delegation. |
| 22 | `ErrInsufficientShares` | Insufficient delegation shares. |
| 23 | `ErrDelegationValidatorEmpty` | Cannot delegate to an empty validator. |
| 24 | `ErrNotEnoughDelegationShares` | Not enough delegation shares. |
| 25 | `ErrNotMature` | Entry not mature. |
| 26 | `ErrNoUnbondingDelegation` | No unbonding delegation found. |
| 27 | `ErrMaxUnbondingDelegationEntries` | Too many unbonding delegation entries for (delegator, validator) tuple. |
| 28 | `ErrNoRedelegation` | No redelegation found. |
| 29 | `ErrSelfRedelegation` | Cannot redelegate to the same validator. |
| 30 | `ErrTinyRedelegationAmount` | Too few tokens to redelegate (truncates to zero tokens). |
| 31 | `ErrBadRedelegationDst` | Redelegation destination validator not found. |
| 32 | `ErrTransitiveRedelegation` | Redelegation to this validator already in progress; first redelegation to this validator must complete before next redelegation. |
| 33 | `ErrMaxRedelegationEntries` | Too many redelegation entries for (delegator, src-validator, dst-validator) tuple. |
| 34 | `ErrDelegatorShareExRateInvalid` | Cannot delegate to validators with invalid (zero) ex-rate. |
| 35 | `ErrBothShareMsgsGiven` | Both shares amount and shares percent provided. |
| 36 | `ErrNeitherShareMsgsGiven` | Neither shares amount nor shares percent provided. |
| 37 | `ErrInvalidHistoricalInfo` | Invalid historical info. |
| 38 | `ErrNoHistoricalInfo` | No historical info found. |
| 39 | `ErrEmptyValidatorPubKey` | Empty validator public key. |
| 40 | `ErrCommissionLTMinRate` | Commission cannot be less than min rate. |
| 41 | `ErrUnbondingNotFound` | Unbonding operation not found. |
| 42 | `ErrUnbondingOnHoldRefCountNegative` | Cannot un-hold unbonding operation that is not on hold. |
| 43 | `ErrInvalidSigner` | Expected authority account as only signer for proposal message. |
| 44 | `ErrBadRedelegationSrc` | Redelegation source validator not found. |
| 45 | `ErrNoUnbondingType` | Unbonding type not found. |
| 46 | `ErrConsensusPubKeyAlreadyUsedForValidator` | Consensus pubkey is already in use by another validator. |
| 47 | `ErrConsensusPubKeyInRotationHistory` | Consensus pubkey was previously rotated away from and is still within the unbonding window. |
| 48 | `ErrExceedingMaxConsPubKeyRotations` | Validator has reached the max consensus pubkey rotations within the unbonding period. |

<a id="errors-cs-std-undefined"></a>

#### Codespace `undefined`

| Code | Name | Meaning |
|---|---|---|
| 1 | `ErrInternal` | Internal. |
| 2 | `ErrStopIterating` | Stop iterating. |
| 111222 | `ErrPanic` | Panic. |
