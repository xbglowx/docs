# Cluster Controller

Kubecost's Cluster Controller contains Kubecost's automation features, and thus has write permission to certain resources on your cluster.

Cluster controller enables actions like:

* Automated cluster scaledown
* 1-click cluster right-sizing
* [Automated request right-sizing](auto-request-sizing.md)

This document shows you how to setup and enable this functionality in the Kubecost product.&#x20;

{% hint style="warning" %}
Cluster Controller supports GKE and EKS clusters and is currently in beta. If you are enabling the Cluster Controller for a GKE/EKS cluster, follow the specialized instructions as usual. If you aren't using a GKE/EKS cluster, skip ahead to the [Deploying ](https://docs.kubecost.com/install-and-configure/advanced-configuration/controller#deploying)section below.
{% endhint %}

## GKE setup

The following command performs the steps required to set up a service account. [More info](https://github.com/kubecost/cluster-turndown/blob/master/scripts/README.md)

```bash
/bin/bash -c "$(curl -fsSL https://github.com/kubecost/cluster-turndown/releases/latest/download/gke-create-service-key.sh)" -- <Project ID> <Service Account Name> <Namespace> cluster-controller-service-key
```

To use [this setup script](https://github.com/kubecost/cluster-turndown/blob/master/scripts/gke-create-service-key.sh) supply the following required parameters:

* **Project ID**: The GCP project identifier you can find via: `gcloud config get-value project`
* **Service Account Name**: The desired service account name to create, e.g. `kubecost-controller`
* **Namespace**: This should be the namespace which Kubecost will be installed, e.g `kubecost`
* **Secret Name**: This should always be set to `cluster-controller-service-key`, which is the secret name mounted by the Kubecost Helm chart.

## EKS setup

Create a new User with `AutoScalingFullAccess` permissions. Create a new file, _service-key.json_, and use the access key ID and secret access key to fill out the following template:

```json
{
    "aws_access_key_id": "<ACCESS_KEY_ID>",
    "aws_secret_access_key": "<SECRET_ACCESS_KEY>"
}
```

Then, run the following to create the secret:

```bash
$ kubectl create secret generic cluster-controller-service-key -n <NAMESPACE> --from-file=service-key.json
```

## Deploying

Once the secret has been successfully created containing the provider service key, you can enable the `cluster-controller` in the Helm chart by finding the `clusterController` config block and setting `enabled: true`

```yaml
# Kubecost Cluster Controller for Right Sizing and Cluster Turndown
clusterController:
    enabled: true
```

You may also enable via `--set` when running Helm install:

```bash
--set clusterController.enabled=true
```

## Using automated cluster scaledown

Cluster Controller wraps all functionality in and provides the same interface/CRDs as https://github.com/kubecost/cluster-turndown. Follow that documentation for usage instructions. You can safely ignore the deployment instructions in that README because you have already deployed Cluster Controller.

{% hint style="info" %}
The v1 -> v2 breaking change mentioned in the cluster-turndown README also applies to Cluster Controller, but for v0.0.6 -> v0.1.0. Cluster Controller was upgraded to v0.1.0 in v1.94 of Kubecost. Follow the [migration guide](turndown-schedule-migration-guide.md) if you use turndown in a version of Kubecost earlier than v1.94 and are upgrading to v1.94+ of Kubecost.
{% endhint %}

