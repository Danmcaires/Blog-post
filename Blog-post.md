# Deploying your application to StarlingX

An application can be deployed in many ways to the 
[Kubernetes cluster(s) that StarlingX manages](https://docs.starlingx.io/operations/k8s_cluster.html)
:

- [raw Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/);
- [Helm](https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package);
- [Flux](https://fluxcd.io/); and finally
- StarlingX Application, which benefits from tight integration with the
  [StarlingX system](https://opendev.org/starlingx/config).

In this article we will continue our deployment demonstrations, this time with
focus on [FluxCD](https://fluxcd.io/), which is a continuous delivery tool that
is used to keep kubernetes clusters in sync with configuration sources.

In this example we will use a virtual All-In-One Simplex (AIO-SX) setup of
StarlingX. If you want to follow along, you can install your own with
[this guide](https://docs.starlingx.io/deploy_install_guides/release/virtual/automated_install.html#dashboards).


## The Demo App

We will use the same application demonstraded in the previous post. If you want
to know more about the demo app, please visit the [demo app repository](https://github.com/bmuniz-daitan/poc-starlingx-messages)

