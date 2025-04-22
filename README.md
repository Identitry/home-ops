# Home-Ops

# Cozystack opionated Kubernetes cluster on top of Talos

This is the config for my Kubernetes cluster.

This will be a living project with tha apps I will run at home.

After trying to run an RKE2 cluster on Rocky Linux i decided to redo it all on [Talos](https://www.talos.dev) with [CozyStack](https://cozystack.io) that opionates the cluster with a lot of operators that simplifies everything but gives slightly less control but that doesn't worry me too much.

I am/will be using:

- [Flux CD](https://fluxcd.io): Git-Ops way of deplying and managing apps in the cluster.
- [Renovate](https://www.mend.io/mend-renovate/): Dependency management
