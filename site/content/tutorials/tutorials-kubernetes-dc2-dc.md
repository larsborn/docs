---
fileID: tutorials-kubernetes-dc2-dc
title: Start ArangoDB Cluster to Cluster Synchronization on Kubernetes
weight: 190
description: 
layout: default
---
This tutorial guides you through the steps needed to configure
an ArangoDB Datacenter-to-Datacenter Replication between two ArangoDB
clusters running in Kubernetes.

{{< tag "ArangoDB Enterprise" >}}

## Requirements

1. This tutorial assumes that you have 2 ArangoDB clusters running in 2 different Kubernetes clusters.
1. Both Kubernetes clusters are equipped with support for `Services` of type `LoadBalancer`.
1. You can create (global) DNS names for configured `Services` with low propagation times. E.g. use Cloudflare.
1. You have 4 DNS names available:
   - One for the database in the source ArangoDB cluster. E.g. `src-db.mycompany.com`
   - One for the ArangoDB syncmasters in the source ArangoDB cluster. E.g. `src-sync.mycompany.com`
   - One for the database in the destination ArangoDB cluster. E.g. `dst-db.mycompany.com`
   - One for the ArangoDB syncmasters in the destination ArangoDB cluster. E.g. `dst-sync.mycompany.com`

## Step 1: Enable Datacenter Replication Support on source ArangoDB cluster

Set your current Kubernetes context to the Kubernetes source cluster.

Edit the `ArangoDeployment` of the source ArangoDB clusters.

Set:

- `spec.tls.altNames` to `["src-db.mycompany.com"]` (can include more names / IP addresses)
- `spec.sync.enabled` to `true`
- `spec.sync.externalAccess.masterEndpoint` to `["https://src-sync.mycompany.com:8629"]`
- `spec.sync.externalAccess.accessPackageSecretNames` to `["src-accesspackage"]`

## Step 2: Extract access-package from source ArangoDB cluster

Run:

{% raw %}
{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl get secret src-accesspackage --template='{{index .data "accessPackage.yaml"}}' | \
  base64 -D > accessPackage.yaml
```
{{% /tab %}}
{{< /tabs >}}
{% endraw %}

## Step 3: Configure source DNS names

Run:

{{< tabs >}}
{{% tab name="bash" %}}
{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl get service
```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}}

Find the IP address contained in the `LoadBalancer` column for the following `Services`:

- `<deployment-name>-ea` Use this IP address for the `src-db.mycompany.com` DNS name.
- `<deployment-name>-sync` Use this IP address for the `src-sync.mycompany.com` DNS name.

The process for configuring DNS names is specific to each DNS provider.

## Step 4: Enable Datacenter Replication Support on destination ArangoDB cluster

Set your current Kubernetes context to the Kubernetes destination cluster.

Edit the `ArangoDeployment` of the source ArangoDB clusters.

Set:

- `spec.tls.altNames` to `["dst-db.mycompany.com"]` (can include more names / IP addresses)
- `spec.sync.enabled` to `true`
- `spec.sync.externalAccess.masterEndpoint` to `["https://dst-sync.mycompany.com:8629"]`

## Step 5: Import access package in destination cluster

Run:

{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl apply -f accessPackage.yaml
```
{{% /tab %}}
{{< /tabs >}}

Note: This imports two `Secrets`, containing TLS information about the source cluster,
into the destination cluster

## Step 6: Configure destination DNS names

Run:

{{< tabs >}}
{{% tab name="bash" %}}
{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl get service
```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}}

Find the IP address contained in the `LoadBalancer` column for the following `Services`:

- `<deployment-name>-ea` Use this IP address for the `dst-db.mycompany.com` DNS name.
- `<deployment-name>-sync` Use this IP address for the `dst-sync.mycompany.com` DNS name.

The process for configuring DNS names is specific to each DNS provider.

## Step 7: Create an `ArangoDeploymentReplication` resource

Create a yaml file (e.g. called `src-to-dst-repl.yaml`) with the following content:

{{< tabs >}}
{{% tab name="yaml" %}}
```yaml
apiVersion: "replication.database.arangodb.com/v1alpha"
kind: "ArangoDeploymentReplication"
metadata:
  name: "replication-src-to-dst"
spec:
  source:
    masterEndpoint: ["https://src-sync.mycompany.com:8629"]
    auth:
      keyfileSecretName: src-accesspackage-auth
    tls:
      caSecretName: src-accesspackage-ca
  destination:
    deploymentName: <dst-deployment-name>
```
{{% /tab %}}
{{< /tabs >}}

## Step 8: Wait for DNS names to propagate

Wait until the DNS names configured in step 3 and 6 resolve to their configured
IP addresses.

Depending on your DNS provides this can take a few minutes up to 24 hours.

## Step 9: Activate replication

Run:

{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl apply -f src-to-dst-repl.yaml
```
{{% /tab %}}
{{< /tabs >}}

Replication from the source cluster to the destination cluster will now be configured.

Check the status of the replication by inspecting the status of the `ArangoDeploymentReplication` resource using:

{{< tabs >}}
{{% tab name="bash" %}}
```bash
kubectl describe ArangoDeploymentReplication replication-src-to-dst
```
{{% /tab %}}
{{< /tabs >}}

As soon as the replication is configured, the `Add collection` button in the `Collections`
page of the web interface (of the destination cluster) will be grayed out.