# Charts
Helm Charts for Fraudaverse

## Login with the registry
We use AWS ECR as a container registry. To receive the images, we use a **secret** in K8S. Please create the secret in the according namespace using:

```bash
export FA_AWS_SECRET=$(aws ecr get-login-password --region us-east-1)

kubectl create secret docker-registry fraudaverse-ecr --docker-server=560924052112.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=$FA_AWS_SECRET --namespace=<NAMESPACE_HERE>
```

## Install processing and ui

To install processing or UI, navigate to `charts` or `charts` and just type `helm install [processing/ui] --namespace <NAMESPACE_HERE> [processing/ui]`.

## Installation for local testing

### Preparing kubectl

For local testing, some steps are involved. First, please install any kind of Kubernetes implementation for your Desktop System. There an plenty to choose from:

- [Docker Deskop](https://docs.docker.com/desktop/kubernetes/), please enable Kubernetes in Settings
- [Rancher Desktop](https://rancherdesktop.io/), Docker and Kubernetes implementation by SuSE
- [minikube](https://minikube.sigs.k8s.io/docs/start/)

(if you have more, please add it to the list)

Once you have installed and activated Kubernetes with one of these, please use the commandline tool `kubectl` to check if you are connected to the kubernetes cluster running on your system.

```bash
$ kubectl get nodes -A

output:
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   27d   v1.25.4
```

The node being displayed should have a name similar to docker-deskop / rancher / minikube. **If you have any other name**, especially containing "eks" or "gcp" or "azure", please check the available kubectl contexts for your local kubectl installation. Because you are most likely connected to a remote server then

```bash
$ kubectl config get-contexts

output:
CURRENT   NAME
          docker-desktop
          rancher-desktop
*         tobias.schmitt@fa-demo-eks.eu-central-1.eksctl.io
```

This shows that you are currently connected to a different context on eks (which is aws). Switch with `kubectl config use-context docker-desktop`.

### Install aws-cli

To connect with our AWS Image repository (ECR), you need to have your iris AWS account credentials ready. You need the Username, The ACCESS_ID and the ACCESS_SECURITY_KEY.

Please install the aws cli using [this tutorial](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). Don't forget to login with our accounts using [the quickstart](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html).


### Add AWS Login credentials to Kubernetes Cluster.

To enable the local kubernetes cluster to download the docker images from our AWS ECR, please use the following command to login your k8s cluster (yes, it's long).

```
$ kubectl create secret docker-registry fraudaverse-ecr --docker-server=560924052112.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-east-1) --namespace=default

output:
secret/fraudaverse-ecr created
```

If you instead see the output
```
error: failed to create secret secrets "fraudaverse-ecr" already exists
```

Then just delete the current secret by using

```
§ kubectl delete secret fraudaverse-ecr

output:
secret "fraudaverse-ecr" deleted
```

and rerun the `kubectl create ...` command.

### Start the charts

To start the charts, please refer to [Install processing and UI](#install-processing-and-ui). You can leave out the namespace part to use namespace `default`.