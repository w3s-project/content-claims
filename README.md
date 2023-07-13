<div align="center">
  <h1>🦪<br/>Content Claims</h1>
</div>

[![Test](https://github.com/web3-storage/content-claims/actions/workflows/test.yml/badge.svg)](https://github.com/web3-storage/content-claims/actions/workflows/test.yml)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)

Implementation of the Content Claims Protocol.


## Background

Read the [spec](https://hackmd.io/@gozala/content-claims).

### Supported claims

These are the types of claim that we're interested in from the spec:

#### Location claim

Claims that a CID is available at a URL.

Capability: `assert/location`

Input:

```js
{
  content: CID /* CAR CID */, 
  location: ['https://r2.cf/bag...car', 's3://bucket/bag...car'],
  range?: { offset: number, length?: number } /* Optional: Byte Range in URL */
}
```

#### Inclusion claim

Claims that a CID includes the contents claimed in another CID.

Capability: `assert/inclusion`

Input:

```js
{
  content: CID /* CAR CID */,
  includes: CID /* CARv2 Index CID */,
  proof?: CID /* Optional: zero-knowledge proof */
}
```

#### Partition claim

Claims that a CID's graph can be read from the blocks found in parts.

Capability: `assert/partition`

Input:

```js
{
  content: CID /* Content Root CID */,
  blocks?: CID /* CIDs CID */,
  parts: [
    CID /* CAR CID */,
    CID /* CAR CID */,
    ...
  ]
}
```

#### Relation claim 🆕

Claims that a CID links to other CIDs. Like a [partition claim](#partition-claim) crossed with an [inclusion claim](#inclusion-claim), a relation claim asserts that a block of content links to other blocks and, that the block and it's links may be found in the specified parts. Furthermore, for each part it specifies which CIDs are included in which parts via reference to a CARv2 index CID.

Capability: `assert/relation`

Input:

```js
{
  content: CID /* Block CID */,
  children: [
    CID /* Linked block CID */,
    CID /* Linked block CID */,
    ...
  ],
  parts: [
    {
      content: CID /* CAR CID */,
      includes: CID /* CID of CARv2 index that lists the blocks this part includes */
    }
  ]
}
```


## Usage

### Client libraries

Client libraries make reading and writing claims easier.

* [JavaScript client](https://www.npmjs.com/package/@web3-storage/content-claims)

### HTTP API

The production deployment is at https://claims.web3.storage.

#### `GET /claims/:cid`

Fetch a CAR full of content claims for the content CID in the URL path.

Query parameters:

* `?walk=` - a CSV list of properties in claims to walk in order to return additional claims about the related CIDs. Any property that is a CID can be walked. e.g. `?walk=parts,includes`.


## Getting started

The repo contains the infra deployment code and the service implementation.

```
├── packages   - content-claims core and lambda wrapper
└── stacks     - sst and AWS CDK code to deploy all the things
```

To work on this codebase **you need**:

- Node.js >= v18 (prod env is node v18)
- An AWS account with the AWS CLI configured locally
- Copy `.env.tpl` to `.env` and fill in the blanks
- Install the deps with `npm i`

Deploy dev services to your AWS account and start dev console. You may need to provide a `--profile` to pick the aws profile to deploy with.

```sh
npm start
```

See: https://docs.sst.dev for more info on how things get deployed.


### Environment variables

The following should be set in the env when deploying. Copy `.env.tpl` to `.env` to set in dev.

```sh
SENTRY_DSN=<your error reporting key here>

# Region of the DynamoDB to query
DYNAMO_REGION=us-west-2

# Private key for the service
PRIVATE_KEY=MgCblCY...
```

#### `SENTRY_DSN`

Data source name for Sentry application monitoring service.

#### `DYNAMO_REGION`

Region of the DynamoDB to query.

#### `PRIVATE_KEY`

The [`multibase`](https://github.com/multiformats/multibase) encoded ED25519 keypair used as the signing key for content-claims.

Generated by [@ucanto/principal `EdSigner`](https://github.com/web3-storage/ucanto) via [`ucan-key`](https://www.npmjs.com/package/ucan-key)

_Example:_ `MgCZG7EvaA...1pX9as=`


### Secrets

Set production secrets in AWS SSM via [`sst secrets`](https://docs.sst.dev/config#sst-secrets). The region must be set to the one you deploy that stage to:

```sh
# set `PRIVATE_KEY` for prod
$ npx sst secrets set --region us-west-2 --stage prod PRIVATE_KEY "MgCblCY...="
```

To set a fallback value for `staging` or an ephemeral PR build use [`sst secrets set --fallback`](https://docs.sst.dev/config#fallback-values)

```sh
# set `PRIVATE_KEY` for any stage in us-east-2
$ npx sst secrets set --region us-east-2 --fallback PRIVATE_KEY "MgCZG7...="
```

**note** The fallback value can only be inherited by stages deployed in the same AWS account and region.

Confirm the secret value using [`sst secrets list`](https://docs.sst.dev/config#sst-secrets)

```sh
$ npx sst secrets list --region us-east-2
PRIVATE_KEY MgCZG7...= (fallback)

$ npx sst secrets list --region us-west-2 --stage prod
PRIVATE_KEY M...=
```


### DynamoDB tables

```ts
interface ClaimTable {
  /** CID of the UCAN invocation task we received this claim in. */
  claim: string // Note: sort key

  /** Archive of the claim invocation */
  bytes: Uint8Array

  /** The subject of the claim. */
  content: string // Note: partition key

  /** UCAN expiration */
  expiration: number
}
```

## Contributing

Feel free to join in. All welcome. Please [open an issue](https://github.com/web3-storage/content-claims/issues)!

## License

Dual-licensed under [MIT + Apache 2.0](https://github.com/web3-storage/content-claims/blob/main/LICENSE.md)
