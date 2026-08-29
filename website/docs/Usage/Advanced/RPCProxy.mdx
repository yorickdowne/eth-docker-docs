---
title: "RPC Proxy"
sidebar_position: 10
sidebar_label: RPC Proxy
---

## Verified RPC Proxy

When expiring more than pre-merge history and running protocols such as RocketPool, Nodeset or SSV, the receipts
needed for these protocols are missing from the local execution layer. The `eth_getLogs` calls for these receipts
will fail. This issue will become even more pronounced when validators can run without an execution layer at all,
and just consume proofs instead.

The obvious solution is to use third-party RPC for RPC queries. But this requires trusting the third-party RPC.

A "verified RPC proxy" can solve this, by checking against a trusted block root.

### Setup

- Make an account with Alchemy or any other provider that supports `eth_getProof`. Infura does not as of March 2026.
Free account should be fine if used for occasional receipts queries.
- Add `nimbus-vp.yml` to `COMPOSE_FILE` in `.env` via `nano .env`
- While in `.env`, set `RPC_URL` to your RPC provider's `wss://` endpoint, including API key / auth
- Run `./ethd update` and `./ethd up`
- RocketPool, **only** if using hybrid mode with execution layer client in Eth Docker: `rocketpool service config` and set
"Execution Client" `HTTP URL` to `http://rpc-proxy:48545` and `Websocket URL` to `ws://rpc-proxy:48546`
- Nodeset, **only** if using hybrid mode with execution layer client in Eth Docker: `hyperdrive service config` and set
"Execution Client" `HTTP URL` to `http://rpc-proxy:48545` and `Websocket URL` to `ws://rpc-proxy:48546`
- SSV Node, `nano ssv-config/config.yaml` and change the `ETH1Addr` to be `ws://rpc-proxy:48546`, then `./ethd restart ssv-node`
- SSV Anchor, `nano .env` and change `EL_RPC_NODE` to `http://rpc-proxy:48545` and `EL_WS_NODE` to `ws://rpc-proxy:48546`, then
`./ethd restart anchor`
- Lido SimpleDVT with Obol, `nano .env` and change `OBOL_EL_NODE` to `http://rpc-proxy:48545`, then `./ethd restart validator-ejector`

### Adjusting defaults

Nimbus Verified Proxy on startup gets a trusted root from `CL_NODE`, connects to `RPC_URL`, and proxies all
RPC requests while verifying them against the trusted root. This works for `http://` and `ws://` queries.

You can change the ports the proxy listens on with `PROXY_RPC_PORT` and `PROXY_WS_PORT`.

These ports can be exposed to the host via `proxy-shared.yml`, or encrypted to `https://` and `wss://`
via `proxy-traefik.yml`, in which case you also want `DOMAIN`, `PROXY_RPC_HOST`, `PROXY_WS_HOST`, and either
`traefik-cf.yml` or `traefik-aws.yml` with their attendant parameters.

If running multiple Eth Docker stacks on one host, each with an rpc proxy and using the same Docker bridge network via
`ext-network.yml`, you can use `RPC_PROXY_ALIAS` and `WS_PROXY_ALIAS` to give the proxies distinctive names. In
that case, use the alias names when configuring other protocols to connect to the proxy, do **not** use
`rpc-proxy` as it'd round-robin between multiple instances of the proxy on the bridge network.
