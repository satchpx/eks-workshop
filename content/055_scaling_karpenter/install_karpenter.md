---
title: "Install Karpenter"
weight: 20
draft: false
---

In this section we will install Karpenter and learn how to configure a default [Provisioner CRD](https://karpenter.sh/docs/provisioner-crd/) to set the configuration. Karpenter is installed in clusters with a [helm](https://helm.sh/) chart. Karpenter follows best practices for kubernetes controllers for its configuration. Karpenter uses [Custom Resource Definition(CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) to declare its configuration. Custom Resources are extensions of the Kubernetes API. One of the premises of Kubernetes is the [declarative aspect of its APIs](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). Karpenter similifies its configuration by adhering to that principle.

## Install Karpenter Helm Chart

We will use helm to deploy Karpenter to the cluster. 

```bash
export KARPENTER_IAM_ROLE_ARN="arn:aws:iam::${ACCOUNT_ID}:role/${CLUSTER_NAME}-karpenter"
export CLUSTER_ENDPOINT="$(aws eks describe-cluster --name ${CLUSTER_NAME} --query "cluster.endpoint" --output text)"
echo "export KARPENTER_IAM_ROLE_ARN=${KARPENTER_IAM_ROLE_ARN}" >> ~/.bash_profile
echo "export CLUSTER_ENDPOINT=${CLUSTER_ENDPOINT}" >> ~/.bash_profile

helm repo add karpenter https://charts.karpenter.sh
helm repo update
helm upgrade --install --namespace karpenter --create-namespace \
  karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version ${KARPENTER_VERSION}\
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=${KARPENTER_IAM_ROLE_ARN} \
  --set serviceAccount.annotations."app\.kubernetes\.io/managed-by"=Helm \
  --set serviceAccount.annotations."meta\.helm\.sh/release-name"=karpenter \
  --set serviceAccount.annotations."meta\.helm\.sh/release-namespace"=karpenter \
  --set settings.aws.clusterName=${CLUSTER_NAME} \
  --set settings.aws.clusterEndpoint=${CLUSTER_ENDPOINT} \
  --set settings.aws.defaultInstanceProfile=KarpenterNodeInstanceProfile-${CLUSTER_NAME} \
  --set settings.aws.interruptionQueueName=${CLUSTER_NAME} \
  --set defaultProvisioner.create=false \
  --wait # for the defaulting webhook to install before creating a Provisioner
```

The command above:
* uses the  service account that we created in the previous step, hence it sets the `--set serviceAccount.create=false`

* uses the both the **CLUSTER_NAME** and the **CLUSTER_ENDPOINT** so that Karpenter controller can contact the Cluster API Server.

* uses the `--set defaultProvisioner.create=false`. We will set a default Provisioner configuration in the next section. This will help us understand Karpenter Provisioners.

* Karpenter configuration is provided through a Custom Resource Definition. We will be learning about providers in the next section, the `--wait` notifies the webhook controller to wait until the Provisioner CRD has been deployed.

To check Karpenter is running you can check the Pods, Deployment and Service are Running.

To check running pods run the command below. There should be at least two pods `karpenter-controller` and `karpenter-webhook`
```bash
kubectl get pods --namespace karpenter
```

To check the deployment. Like with the pods, there should be two deployments  `karpenter-controller` and `karpenter-webhook`
```bash
kubectl get deployment -n karpenter
```

{{% notice note %}}
You can increase the number of Karpenter replicas in the deployment for resilience. Karpenter will elect a leader controller that is in charge of running operations.
{{% /notice %}}


