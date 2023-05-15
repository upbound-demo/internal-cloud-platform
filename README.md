# Internal Cloud Platform

This repository contains the manifests to build your own internal cloud platform
on top of Upbound Self-Hosted Environments. Your cloud platform will be powered
by Managed Crossplane (MXP) control planes.

## Usage

### Personas

The repo is a reference implementation for an organization that is built around
having a **platform team** and **app teams**. **Platform teams** are responsible
for creating the MXP instances, crafting the set of `XRDs` and `compositions` to be
installed on these MXPs, and operating Crossplane. **App teams** are concerned only
with their applications and consuming the APIs created by the Platform team.

### Prerequisites

The usage instructions in this repo assume you have already bootstrapped an Upbound
Self-Hosted Environment in your own Kubernetes Cluster using the `up` CLI. If you
have not done this, consult instructions from the Upbound team before continuing.
This repo also assumes Flux has been provisioned as the CI/CD engine to power "GitOps".

### Deploy an Application

In your app team's repository (an example
[here](https://github.com/upbound-demo/team-blue)), you can create Kubernetes
resources and Crossplane claims together. The following is an example where
Wordpress is deployed on the app team's cluster together with an `SQLInstance`
object whose API is defined by the platform team in this repository.

```yaml
# Content of ./dev/database.yaml
apiVersion: demo.upbound.io/v1alpha1
kind: SQLInstance
metadata:
  name: wordpress-db
  namespace: default
spec:
  engine: mariadb
  engineVersion: "10.6.12"
  storageGB: 10
  initialDBName: wordpress_db
  writeConnectionSecretToRef:
    name: sqlinstance-conn
```
```yaml
# Content of ./dev/wordpress.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          valueFrom:
            secretKeyRef:
              name: sqlinstance-conn
              key: endpoint
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: sqlinstance-conn
              key: username
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: sqlinstance-conn
              key: password
        - name: WORDPRESS_DB_NAME
          value: wordpress_db
        ports:
        - containerPort: 80
          name: wordpress
```

### Register an App Team Repository

You can register your app team's repository to a `KubernetesCluster` so that the
team's manifests are continuously synced. The following is an example where
`team-blue` is registered to sync their application to `app-dev-1` cluster.

```yaml
apiVersion: demo.upbound.io/v1alpha1
kind: KubernetesCluster
metadata:
  name: app-dev-1
  namespace: default
spec:
  teams:
  - name: team-blue
    repository: "https://github.com/upbound-demo/team-blue"
    environments:
    - name: dev
      path: ./dev
    - name: staging
      path: ./staging
    - name: production
      path: ./production
  nodes:
    count: 3
    size: small
  writeConnectionSecretToRef:
    name: kubeconfig-app-dev-1
```

In your app team repository, you can create manifests consuming the APIs defined in
the control plane that the cluster is registered. For example, if the
`KubernetesCluster` above resides in
[`runtime/system-production/apis-production/app-dev`](./runtime/system-production/apis-production/app-dev/),
you can find the claim APIs offered by going to
[`platform/apis/production`](./platform/apis/production/) folder.

### Develop a New API

This reference implementation assumes a platform teams follow standard software
development practices as part of building new APIs in Crossplane. This
implementation is set up to have the platform team promote their API definitions
through 3 stages.

First, you need to add the APIs to `platform/apis/dev` folder that is synced
to the [`prod-dev` control plane](./main/control-planes/system-production/apis-dev).

Once you are happy with the changes, you can promote it to staging whose APIs
are located in `platform/apis/staging` folder that is synced to the
[`prod-staging` control plane](./main/control-planes/system-production/apis-staging).

If it passes all your tests, then it's time to promote to production by adding
the manifests to `platform/apis/production`. Once the API makes it to
production, **it will be deployed to all production control planes**, like
`app-dev`, `app-staging` and `app-production`.

### Install a New Provider

Providers are system components. This implementation assumes they need to go through dev,
staging and production stages under `platform/system` folder. Once your `Provider`
manifests make it to the production stage, they will be deployed to **all production
control planes**.

### Create a New Control Plane

This implementation is structured such that Control plane manifests are stored in
`main/control-planes`. If you'd like to add a production control plane, you can
add necessary manifests to `main/control-planes/system-production/apis-production` folder.

A control plane configuration consists of the following manifests:
* `ControlPlane`: The definition of an MXP instance.
* `Kustomization`s: The FluxCD configurations to tell it from what system, apis
  and runtime stages it should pull manifests from.
* `Role` and `RolePolicyAttachment`: AWS roles that are provisioned so that the
  provider-aws installed in the control plane can make use of IRSA
  authentication method.

## Folders

* `main`: The host clusters and all the control planes that will be
  running on them. All YAMLs in this folder are to be synced to the main
  cluster.
  * `host-clusters`: The manifests for all host clusters.
  * `control-planes`: The manifests for all control planes.
  * `flux-system`: The Flux manifests running in the main cluster.
* `platform`: The platform configurations.
  * `system`: Configuration of the system components such as providers and
    custom controllers.
    * `dev`: The dev stage for system components.
    * `staging`: The staging phase for system components.
    * `production`: The production phase for provider configurations. **All
      production control planes use these provider configurations.**
  * `apis`: API definitions and Compositions.
      * `dev`: The dev phase for API definitions and compositions.
      * `staging`: The staging phase for API definitions and compositions.
      * `production`: The prod phase for API definitions and compositions. **All
        production control planes share these APIs and compositions.**
* `runtime`: Actual runtime manifests that are maintained by the platform team
  and deployed to every control plane. Each control plane has a set of
  `EnvironmentConfig` and `ProviderConfig`s. They may or may not include cloud
  resources such as Kubernetes clusters to be used by teams. 
  * `system-dev`: The manifests for the `dev` control plane.
  * `system-staging`: The manifests for the `staging` control plane.
  * `system-production`: The manifests of all control planes running
    `production` system components.
    * `apis-dev`: The manifests of the dev control plane of APIs that's running
      `production` system components.
    * `apis-staging`: The manifests of the staging control plane of APIs that's
      running `production` system components.
    * `apis-production`: The manifests of all control planes running
      `production` system components and `production` APIs. **Customers of the
      platform team only interact with only these control planes.** These
      control planes can be connected to multiple clusters they provision.
      * `app-dev`: The manifests of the application dev stage clusters.
      * `app-staging`: The manifests of the application staging stage clusters.
      * `app-production`: The manifests of the application production stage
        clusters.
