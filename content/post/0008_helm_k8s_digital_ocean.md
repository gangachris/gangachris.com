---
title: "Deploying Developer Tools to Custom Subdomain names with HELM on Digital Ocean"
description: "We'll deploy three tools (concourse, grafana and ), to a custom subdomain name on  digital ocean using helm"
tags:
  - "helm"
  - "kubernetes"
categories:
  - "code"
slug: "deploying-dev-tools-with-helm"
date: 2019-08-29T17:06:36+02:00
draft: false
---


## Introduction
I know, the title is a mouthful. 

This article serves a guide to deploy common open source developer tools to a server you own (Digital Ocean in this case). The choice on the individual technologies was however made purely on the basis that all this should be reproducible. 

If it is reproducible, it means to deploy this to a different cloud provider, you would require very little effort. This part is mainly played by HELM, since it allows us to package deployments as configuration (some would say as code). 

## Prerequisites
To fully follow along this article, you need to have a basic understanding of a few things. But I try to be as clear as possible. 

- An Open Source Dev Tool (In this case, we will be deploying Concourse CI, and Grafana dashboard), but having knowledge of any free and open source developer tool should help
- HELM - Goes hand in hand with Kubernetes. In depth knowledge of both Helm and K8s is not required, as we'll try to explain as much as possible, but knowing what they do would be nice. 
- Domain Names, and Domain Name Service - Since we will be doing this with a custom domain, having a domain name available to you would be nice. Also, knowing a little bit about how to change domain name servers on your domain name provide would be nice. The example I use here is my domain (`gangachris.com`), which was registered with NameCheap.
- Digital Ocean Account - This is not really necessary, as if you are familiar with a different cloud provider, you should be able to translate what we do here to your specific cloud provider. 
- Terraform - Though not required, I use terraform to create the K8s cluster that will be used. Again, if you can go to your cloud provider and create a K8s cluster, and change your cluster context to point to it, you should be good to go.

 Your computer needs to have the following installed in order to follow along. 
  - kubectl - kubernetes cli
  - minikube - will allow us to run a cluster locally for testing purposes.
  - helm - HELM CLI installed
  - doclt - digital ocean cli will allow us to set up authentication for digital ocean


### Local First

 If it runs locally, it usually means you are one step away from a good deployment. We will first run this setup locally, then transition into deploying it to Digital Ocean. 

 Assuming you have all the prerequisites installed, we will create a local cluster with minikube. 
{{<highlight shell >}}
$ minikube start
{{< / highlight >}}


Minikube start command will instantiate a virtual server, and instantiate a kubernetes cluster, and automatically change your current context to point to it. It takes in a couple of arguments, such as disk mounts, and what type of virtualization software to use (e.g virtual box or x-hyve).

Once the command is done, you can confirm that your current cluster is checking the docker icon in the menu bar, under kubernetes, or by running the following command
```bash
kubectl config current-context
```
Result should be minikube.

Not we need to create our HELM chart. This Chart will contain our whole deployment. Since we are deploying developer tools, we'll call it tools. 
```
helm create tools
```

This will create a directory with the following structure.
```
tools
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 8 files

```

This is the the normal file structure of a Helm Chart. The `helm create` command helps in scaffolding HeLM charts, and has a lot of options to help in generating these files. Here however we use the default options. 

IF you are familiar with HELM, then this may already make sense to you. If you are not familiar, Here's a layman definition of this structure. 

The `templates` directory has general kubertnetes resources files (deployments, services, jobs) as go templates, and the `values.yaml` file in the root directory usually has variables that are used in the templates. Helm then reads all this as kubernetes configuration, and deplploys whatever configurations you have. 

To test this default chart created, run the following
```
helm init
helm install ./ 
```

This will output something similar to below, showing that your deployment is running.
```bash
NAME:   foppish-lobster
LAST DEPLOYED: Tue Aug 27 23:40:45 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME                   READY  UP-TO-DATE  AVAILABLE  AGE
foppish-lobster-tools  0/1    1           0          0s

==> v1/Pod(related)
NAME                                   READY  STATUS             RESTARTS  AGE
foppish-lobster-tools-f95f69749-6s78s  0/1    ContainerCreating  0         0s

==> v1/Service
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
foppish-lobster-tools  ClusterIP  10.99.203.247  <none>       80/TCP   0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=tools,app.kubernetes.io/instance=foppish-lobster" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

Following the instructions on the NOTES section <which is generated by the ./templates/NOTES.txt file>, you can view your server. 

## Deploy Concourse on local cluster
Now, to deploy concourse, you can do it in two ways. 

- Directly running helm install and putting in the concourse chart URL
- Defining Helm as a dependency to our chart, so that it can be installed by simply installing our chart.

The latter approach is what we will go for, ass the point of using all this is so that it is producable and configurable.

Helm dependencies are defined in a file `reequirements.yaml`. In this case, we will need concourse so create a file in the roo to the tools directorie called `reuirements.yaml` and add the following. 
{{< highlight go "linenos=table" >}}
// requirements.yaml
- dependencies:
  - name: concourse
    version: 8.0.2
    repository: "@stable"
{{</ highlight >}}

The version of the chart was gotten from the Chart.yaml file in the stable repo. You wan to make sure you have the dependency downloaded so you have to run
```bash
helm dep update
```

A directory called charts will be created to store external dependency charts. 

Now that we have the chart, to add it to our cluster, we simply need to add it's configurations to our values file. 

Add the following to the values file. These configuration values can be found in the documentation of the concourse chart.
