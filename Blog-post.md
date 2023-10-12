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

We will use the same application demonstrated in the previous post. If you want
to know more about the demo app, please visit the
[demo app repository](https://github.com/bmuniz-daitan/poc-starlingx-messages)


## Deploying the Demo App

### Via FluxCD resources

While the StarlingX platform does not have the FluxCD CLI, it does have its
resources. An application can be easily deployed and managed by means of the
resources made available by Flux. For a list of all available Flux resources
refer to the [official documentation](https://fluxcd.io/flux/components/).


There are many ways of deploying an application using Flux. In this demonstration
we will create a source controller and a helm controller

#### Source Controller

FluxCD offers 5 types of source controllers:

- GitRepository
- OCIRepository
- HelmRepository
- HelmChart
- Bucket

For this demonstration we will focus on the creation of a GitRepository. Make
sure to choose the best type of source controller for your specific use case.

First we are going to create the resource GitRepository that will contain the
helm chart repository.

```shell
cat <<EOF > gitrepository.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: poc-starlingx
spec:
  interval: 2m
  url: https://github.com/bmuniz-daitan/poc-starlingx-messages.git
  ref: 
    branch: main

EOF

kubectl apply -f gitrepository.yaml
```
The command above will create a GitRepository resource called poc-starlingx that
contains the contents of https://github.com/bmuniz-daitan/poc-starlingx-messages.git

```shell
kubectl get gitrepositories


```

#### Helm Controller

The helm controller is an operator that allows the management of helm charts
releases in a declarative way. The resource HelmRelease will define the desired
state of a Helm release, and based on actions upon this resource (creation,
deletion or mutation) the controller will perform Helm actions.

To deploy the demo application, we will execute the following command:

```shell
cat <<EOF > helmrelease.yaml
apiVersion: "helm.toolkit.fluxcd.io/v2beta1"
kind: HelmRelease
metadata:
  name: poc-starlingx
spec:
  releaseName: poc-starlingx
  interval: 2m
  chart:
    spec:
      chart: ./helm-chart  # Relative path of the helm chart inside the repo.
      version: 1.5.2
      sourceRef:
        kind: GitRepository
        name: poc-starlingx
  values: # Override values for the helm chart
    env:
      - name: MODE
        value: node
      - name: SERVER
        value: 127.0.0.1:8100
      - name: PORT
        value: "8000"

    image:
      tag: latest
      containerPort: 8000

    kube:
      port: 31234
      replicas: 1
      name: poc-starlingx

EOF

kubectl apply -f helmrelease.yaml
```

The command above will create two resources. First it will create a HelmChart
resource that will hold the helm-chart itself that will be loaded from the
GitRepository. 

```shell
kubectl get helmcharts


```


After the HelmChart resource is successfully created it will
create the HelmRelease for the deployment of the application with the values
informed inside the yaml file.

```shell
kubectl get helmreleases


```

