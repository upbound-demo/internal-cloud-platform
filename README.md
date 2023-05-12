# Internal Cloud Platform

This repository contains the manifests to build your own internal cloud platform
with Upbound Managed Crossplane (MXP).

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