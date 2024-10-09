# cf-workers-broken-local-https

This repo reproduces a bug where subsequent to local https resources result in `Error: internal error` in local development. It was first reported at https://github.com/cloudflare/workers-sdk/issues/5774.

## Overview

We will spin up a local https serveru using Caddy and certificates are issued by Caddy's built-in PKI. In Workers script, we will always route requests to the Caddy https endpoint.

## Steps

### Setup Caddy

Follow https://caddyserver.com/docs/install to install caddy

```sh
caddy run --config Caddyfile --watch
```

Open a separate terminal to trust the self-signed certificate

```sh
caddy trust
```

### Setup hosts file

```sh
sudo echo "127.0.0.1  sourcegraph.test" >> /etc/hosts
```

## Run the Workers script

> You may need to adjust the path to the certificates https://caddyserver.com/docs/conventions#file-locations

```sh
npx wrangler dev --local --https-key-path "$HOME/Library/Application Support/Caddy/certificates/local/sourcegraph.test/sourcegraph.test.key" --https-cert-path "$HOME/Library/Application Support/Caddy/certificates/local/sourcegraph.test/sourcegraph.test.crt"
```

## Test the endpoint

Verify the Caddy https endpoint is working:

```sh
curl https://sourcegraph.test:4443
```

Try to access the Workers:

```sh
curl https://sourcegraph.test:8787
```

you should get:

```
Error: internal error
    at async jsonError (file:///Users/michael/Code/michaellzc/cf-workers-broken-local-https/node_modules/wrangler/templates/middleware/middleware-miniflare3-json-error.ts:22:10)
    at async drainBody (file:///Users/michael/Code/michaellzc/cf-workers-broken-local-https/node_modules/wrangler/templates/middleware/middleware-ensure-req-body-drained.ts:5:10)
```

However, you we do so without `https` against a `http` origin, it works.
