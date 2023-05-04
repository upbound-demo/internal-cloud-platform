# Internal Cloud Platform

This repository contains the manifests to build your own internal cloud platform
with Upbound Managed Crossplane (MXP).

* `main`: The host clusters and all the control planes that will be
  running on them. All YAMLs in this folder are to be synced to the main
  cluster.
  * `host-clusters`: The manifests for all host clusters.
  * `control-planes`: The manifests for all control planes.
* `platform`: The provider configurations and API definitions.
  * `dev`: The dev stage phase provider configurations, mostly contains
    `Provider`, `ProviderConfig` and `ControllerConfig`.
  * `staging`: The staging phase for provider configurations, mostly contains
    `Provider`, `ProviderConfig` and `ControllerConfig`.
  * `production`: The production phase for provider configurations. **All
    production control planes use these provider configurations.**
    * `api-dev`: The dev phase for API definitions and compositions.
    * `api-staging`: The staging phase for API definitions and compositions.
    * `api-prod`: The prod phase for API definitions and compositions. **All
      production control planes share these APIs and compositions.**
* `runtime`: The active and running infrastructure maintained by the platform
  team, which may include databases, kubernetes clusters etc. Kubernetes
  clusters are connected to a control plane by default. They may or may
  not be shared by multiple teams.
  * `dev`: The infrastructure for `dev` environments provided to the application
    teams.
  * `staging`: The infrastructure for `staging` environments provided to the application
    teams.
  * `prod`: The infrastructure for `prod` environments provided to the application
    teams.
