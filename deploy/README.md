# Deploy

Deploying a pangeo / Planetary-Computer-style JupyterHub on Azure.
This is a mostly standard [daskhub](https://github.com/dask/helm-chart/tree/main/daskhub) deployment.

## Prerequisites

* An Azure subscription
* [Helm](https://helm.sh/)
* A `secrets.yaml` file filled in. Use `secrets.yaml.example` as a template.

## Authentication

We used the `dummy` authenticator for the workshop. You'll also probably want use one of JupyterHub's [real authenticators](https://jupyterhub.readthedocs.io/en/stable/reference/authenticators.html).

## Helm configuration

There are a few azure specific things in the configuration

* `jupyterhub.proxy.service.annotations.service.beta.kubernetes.io/azure-dns-label-name`: Set this is you want to use AKS's automatic domain name feature. Otherwise, just delete it.
* `jupyterhub.proxy.hosts`: Set this to your hub URL

## Deployment

```
$ make resource-group
$ make cluster
$ make hub
$ make userpools
$ NODE_COUNT=1 make scale  # your number of users
```

That'll get you a multi-user, Dask enabled hub up and running in 10-15 minutes.

## Capacity notes

We're assuming ~100 users for the tutorial. We're using a `Standard_D8s_v3` for the user pool, and assigning two users per node (4 CPU, 16 GiB of RAM).