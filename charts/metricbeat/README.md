# Metricbeat Chart with TLS

Tailored metricbeat Helm chart with TLS enabled

## Get the repo

Because this is TLS enabled, ensure you have the certificates and secrets created

```bash
helm repo add ankh https://ankh-is.github.io/charts/
```

## Secrets and Certificates

If installed with the other charts in mind, TLS is required.  If metricbeat
exists in a different namespace, the secrets should be copied.

## Installation and Upgrade

Once the secrets have been satisfied. Deploy the stack as is with the Makefile.

```bash
make install
```
