---
fileID: tutorials-kubernetes-eks
title: Start ArangoDB on Amazon Elastic Kubernetes Service (EKS)
weight: 175
description: 
layout: default
---
## Requirements:

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (**version >= 1.10**)
* [helm](https://www.helm.sh/)
* [AWS IAM authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator)
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) (**version >= 1.16**)

{{< tabs >}}
{{% tab name="" %}}
```
$ aws --version
  aws-cli/1.16.43 Python/2.7.15rc1 Linux/4.15.0-36-generic botocore/1.12.33
```
{{% /tab %}}
{{< /tabs >}}

## Create a Kubernetes cluster

![clusters](/images/eks-clusters.png)

## Wait for cluster to be `ACTIVE`
![cluster-active](/images/eks-cluster-active.png)

## Continue with aws client

### Configure AWS client

Refer to the [AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
to accordingly fill in the below with your credentials.
Pay special attention to the correct region information  to find your cluster next.

{{< tabs >}}
{{% tab name="" %}}
```
$ aws configure
  AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
  AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
  Default region name [None]: us-west-2
  Default output format [None]: json
```
{{% /tab %}}
{{< /tabs >}}

Verify that you can see your cluster listed, when authenticated
{{< tabs >}}
{{% tab name="" %}}
```
$ aws eks list-clusters
{
  "clusters": [
    "ArangoDB"
  ]
}
```
{{% /tab %}}
{{< /tabs >}}

You should be able to verify the `ACTIVE` state of your cluster
{{< tabs >}}
{{% tab name="" %}}
```
$ aws eks describe-cluster --name ArangoDB --query cluster.status
  "ACTIVE"
```
{{% /tab %}}
{{< /tabs >}}

### Integrate kubernetes configuration locally

It's time to integrate the cluster into your local kubernetes configurations

{{< tabs >}}
{{% tab name="" %}}
```
$ aws eks update-kubeconfig --name ArangoDB
  Added new context arn:aws:eks:us-west-2:XXXXXXXXXXX:cluster/ArangoDB to ...

```
{{% /tab %}}
{{< /tabs >}}

At this point, we are ready to use kubectl to communicate with the cluster.
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl get service
  NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
  kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   23h
```
{{% /tab %}}
{{< /tabs >}}

{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl get nodes
  No resources found.
```
{{% /tab %}}
{{< /tabs >}}

### Create worker Stack

On Amazon EKS, we need to launch worker nodes, as the cluster has none.
Open Amazon's [cloud formation console](https://console.aws.amazon.com/cloudformation/)
and choose `Create Stack` by specifying this S3 template URL:

{{< tabs >}}
{{% tab name="" %}}
```
https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-08-30/amazon-eks-nodegroup.yaml
```
{{% /tab %}}
{{< /tabs >}}

![formation-template](/images/eks-create-template.png)

### Worker stack details

Pay good attention to details here. If your input is not complete, your worker
nodes are either not spawned or you won't be able to integrate the workers
into your kubernetes cluster.

**Stack name**: Choose a name for your stack. For example ArangoDB-stack

**ClusterName**: **Important!!!** Use the same name as above, refer to `aws eks list-clusters`.

**ClusterControlPlaneSecurityGroup**: Choose the same SecurityGroups value as above, when you create your EKS Cluster.

**NodeGroupName**: Enter a name for your node group for example `ArangoDB-node-group`

**NodeAutoScalingGroupMinSize**: Minimum number of nodes to which you may scale your workers.

**NodeAutoScalingGroupMaxSize**: Nomen est omen.

**NodeInstanceType**: Choose an instance type for your worker nodes. For this test we went with the default `t2.medium` instances.

**NodeImageId**: Dependent on the region, there are two image Ids for boxes with and without GPU support.

| Region    | without GPU           | with GPU              |
|-----------|-----------------------|-----------------------|
| us-west-2 | ami-0a54c984b9f908c81 | ami-0440e4f6b9713faf6 |
| us-east-1 | ami-0440e4f6b9713faf6 | ami-058bfb8c236caae89 |
| eu-west-1 | ami-0c7a4976cb6fafd3a | ami-0706dc8a5eed2eed9 |

**KeyName**: SSH key pair, which may be used to ssh into the nodes. This is required input.

**VpcId**: The same VPCId, which you get using `aws eks describe-cluster --name <your-cluster-name>  --query cluster.resourcesVpcConfig.vpcId`

**Subnets**: Choose the subnets that you created in Create your Amazon EKS Cluster VPC.

### Review your stack and submit
![create-review](/images/eks-create-review.png)

### Wait for stack to get ready
![eks-stack](/images/eks-stack.png)

### Note down `NodeInstanceRole`
Once stack is ready, navigate at the bottom to the Outputs pane and note down the `NodeInstanceRole`
![eks-stack](/images/eks-stack-ready.png)

### Integrate worker stack as Kubernetes nodes

* Download the configuration map here:
{{< tabs >}}
{{% tab name="" %}}
```
$ curl -O   https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2018-08-30/aws-auth-cm.yaml
```
{{% /tab %}}
{{< /tabs >}}
* Modify `data|mapRoles|rolearn` to match the `NoteInstanceRole`, you acquired after your node stack was finished

* Deploy node integration
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl apply -f aws-auth-cm.yaml
```
{{% /tab %}}
{{< /tabs >}}

### Wait for nodes to join the cluster and get ready
Monitor `kubectl get nodes` and watch your nodes to be ready
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl get nodes
  NAME                                          STATUS   ROLES    AGE   VERSION
  ip-172-31-20-103.us-west-2.compute.internal   Ready    <none>   1d    v1.10.3
  ip-172-31-38-160.us-west-2.compute.internal   Ready    <none>   1d    v1.10.3
  ip-172-31-45-199.us-west-2.compute.internal   Ready    <none>   1d    v1.10.3
```
{{% /tab %}}
{{< /tabs >}}

### Setup `helm`
* Create service account for `tiller`
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl create serviceaccount --namespace kube-system tiller
    serviceaccount/tiller created
```
{{% /tab %}}
{{< /tabs >}}
* Allow `tiller` to modify the cluster
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl create clusterrolebinding tiller-cluster-rule \
        --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
    clusterrolebinding.rbac.authorization.k8s.io/tiller-cluster-rule created
```
{{% /tab %}}
{{< /tabs >}}
* Initialize `helm`
{{< tabs >}}
{{% tab name="" %}}
```
$ helm init --service-account tiller
    $HELM_HOME has been configured at ~/.helm.
    ...
    Happy Helming!
```
{{% /tab %}}
{{< /tabs >}}

### Deploy ArangoDB cluster
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl apply -f https://raw.githubusercontent.com/arangodb/kube-arangodb/master/examples/simple-cluster.yaml
```
{{% /tab %}}
{{< /tabs >}}

### Wait for cluster to become ready
Get `LoadBalancer` address from below command to access your Coordinator.
{{< tabs >}}
{{% tab name="" %}}
```
$ kubectl get svc
```
{{% /tab %}}
{{< /tabs >}}

### Secure ArangoDB cluster
Do not forget to immediately assign a secure database `root` password once on Coordinator