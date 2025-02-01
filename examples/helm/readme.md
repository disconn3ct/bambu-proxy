# Helm Installation

This is an example installation using Flux2 and Helm. You need to generate the `users` and `blcert.pem` files using the directions in `../kustomization.yaml` and review all `##` comments.

To enable SSL, update the `relay-cert.yaml` with the correct hostname for the relay. If the relay does not have a proper domain name, you cannot enable SSL. This also assumes you have a working cert-manager installation.

The kustomization file is in `examples/` so that it can consume files from `examples/configs`.
