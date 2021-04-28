# Kibana Chart with TLS

Tailored Kibana Helm chart with TLS enabled

## Get the repo

Because this is TLS enabled, ensure you have the certificates and secrets created

```bash
helm repo add ankh https://ankh-is.github.io/charts/
```

## Create Secrets and Certificates

The included `Makefile` creates the helm release `kibana` in the namespace
`observability`.

Generate secrets and certificates and then copy them over to the `observability`
namespace.

```bash
make secrets
make sync-secrets
```

## Installation and Upgrade

Once the secrets have been satisfied. Deploy the stack as is with the Makefile.

```bash
make install
```
