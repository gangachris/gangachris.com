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
- [Digital Ocean Account](https://m.do.co/c/872896dd3f77) - This is not really necessary, as if you are familiar with a different cloud provider, you should be able to translate what we do here to your specific cloud provider. Digital Ocean will provide you with $50 worth of credit for a month if you sign up. [Here's my referral link](https://m.do.co/c/872896dd3f77)
- Terraform - Though not required, I use terraform to create the K8s cluster that will be used. Again, if you can go to your cloud provider and create a K8s cluster, and change your cluster context to point to it, you should be good to go.


Your computer needs to have the following installed in order to follow along. 

  - kubectl - kubernetes cli
    - MAC: `brew install kubectl`
  - minikube - will allow us to run a cluster locally for testing purposes.
    - MAC: `brew cask install minikube`
  - helm - HELM CLI installed
    - MAC: `brew install kubernetes-helm`
  - terraform (optional)
    - MAC: `brew install terraform`
  - doclt - digital ocean cli will allow us to set up authentication for digital ocean
    - MAC: `brew install doctl`


### Local First

 If it runs locally, it usually means you are one step away from a good deployment. We will first run this setup locally, then transition into deploying it to Digital Ocean. Minikube requires that a virtualization software be installed. In my case, I have [Virtual Box](https://www.virtualbox.org/) already installed.

 Assuming you have all the prerequisites installed, we will create a local cluster with minikube. 
```sh
$ minikube start -p tools
```


Minikube start command will instantiate a virtual server, and instantiate a kubernetes cluster, and automatically change your current context to point to it. It takes in a couple of arguments, such as disk mounts, and what type of virtualization software to use (e.g [virtual box](https://www.virtualbox.org/) or [x-hyve](https://github.com/machyve/xhyve)).

```sh
😄  [tools] minikube v1.3.1 on Darwin 10.14.6
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.15.2 on Docker 18.09.8 ...
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "tools"
```

Once the command is done, you can confirm that your current cluster is checking the docker icon in the menu bar, under kubernetes, or by running the following command
```bash
kubectl config current-context
```
Result should be `tools`.

Now we need to create our HELM chart. This Chart will contain our whole deployment. Since we are deploying developer tools, we'll call it tools. 
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

This is the the default structure of a Helm Chart. The `helm create` command helps in scaffolding Helm charts, and has a lot of options to help in generating these files. Here however we use the default options. 

If you are familiar with HELM, then this may already make sense to you. If you are not familiar, Here's a layman definition of this structure. 

The `templates` directory has general kubernetes resources files (deployments, services, jobs) as go templates in yaml, and the `values.yaml` file in the root directory usually has variables that are used in the templates. Helm then reads all this as kubernetes configuration, and deploys whatever configurations you have. 

To test this default chart created, run the following. (`tools` here refers to the directory with the Helm chart we created)
```
helm init
helm install tools
```

This will output something similar to below, showing that your deployment is running.
```bash
NAME:   callous-duck
LAST DEPLOYED: Mon Sep  2 12:25:37 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME                READY  UP-TO-DATE  AVAILABLE  AGE
callous-duck-tools  0/1    1           0          0s

==> v1/Pod(related)
NAME                                 READY  STATUS             RESTARTS  AGE
callous-duck-tools-756fb96bdf-r8tqh  0/1    ContainerCreating  0         0s

==> v1/Service
NAME                TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
callous-duck-tools  ClusterIP  10.105.110.47  <none>       80/TCP   0s


NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=tools,app.kubernetes.io/instance=callous-duck" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl port-forward $POD_NAME 8080:80
```

Following the instructions on the NOTES section <which is generated by the ./templates/NOTES.txt file>, you can view your server, by using kubernetes port forwarding.
```sh
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=tools,app.kubernetes.io/instance=callous-duck" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:80
```

Then visiting localhost:8080, should show you the Nginx server page. You could also get the pod name with `kubectl get pods`, copy the name, and paste it in the `$POD_NAME` section above.


## Deploy Concourse on local cluster
Now, to deploy concourse, you can do it in two ways. 

1. Directly running helm install and putting in the concourse chart
```sh
helm install stable/concourse
``` 

2. Defining Helm as a dependency to our chart, so that it can be installed by simply installing our chart.

The latter approach is what we will go for, as the point of using all this is so that it is reproducible and configurable.

Helm dependencies are defined in a file `requirements.yaml`. In this case, we will need concourse so create a file in the root to the tools directory called `requirements.yaml` and add the following. 
{{< highlight go "linenos=table" >}}
// requirements.yaml
dependencies:
  - name: concourse
    version: 8.2.3
    repository: "@stable"
{{</ highlight >}}

The version of the chart was gotten from the [Chart.yaml](https://github.com/helm/charts/blob/master/stable/concourse/Chart.yaml#L3) file in the stable repo. To pull down these dependency, we need to run 
```bash
helm dep update
```
Which will result in the charts directory being updated with a compressed version of the Concourse CI Helm Chart. 
```shell
charts
└── concourse-8.0.2.tgz

0 directories, 1 file

```
A file `requirements.lock` is also downloaded, to lock in the dependencies. 


Before adding concourse to our deployment, let's do some clean up. Our initial generated helm chart had nginx configured as a default deployment but we really don't need it right now. So we're going to get rid of it. 

In fact, lets delete the templates directory completely, since we will not be using it. New directory structure looks like this.
```sh
.
├── Chart.yaml
├── charts
│   └── concourse-8.2.3.tgz
├── requirements.lock
├── requirements.yaml
└── values.yaml

1 directory, 5 files

```

Next, let's clean up what we had deployed. (We could instead just upgrade the installation with `helm upgrade <path>`), but we''l just delete the existing release. Get the existing release with:
```sh
helm list
```

Then delete it with:
```sh
helm delete --purge <release_name>
```

Next, let's install our chart. (assuming you are in the tools directory). We'll specify the release name and the namespace this time. 
```sh
helm install --name tools-dev --namespace tools-dev
```

You should see the following warning, but everything should be fine. The warning tells us that we haven't provided any configurations for our Concourse CI chart. We'll do this later.
```sh
2019/09/02 15:20:23 Warning: Building values map for chart 'concourse'. Skipped value (map[]) for 'image', as it is not a table.
NAME:   tools-dev
LAST DEPLOYED: Mon Sep  2 15:20:23 2019
NAMESPACE: tools-dev
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME              DATA  AGE
tools-dev-worker  1     1s
```

You should see 4 new pods created, as is expected with the councourse ci chart.
```sh
kubectl get pods --namespace tools-dev

NAME                            READY   STATUS    RESTARTS   AGE
tools-dev-postgresql-0          1/1     Running   0          3m12s
tools-dev-web-8456bb4bc-dlt2z   1/1     Running   0          3m12s
tools-dev-worker-0              1/1     Running   0          3m12s
tools-dev-worker-1              1/1     Running   0          3m12s
```

We do not have instructions to help us reach our server, but the first thing we could check  is if any services are exposed by default.
```
kubectl get services --namespace tools-dev

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
tools-dev-postgresql            ClusterIP   10.100.112.8    <none>        5432/TCP            4m45s
tools-dev-postgresql-headless   ClusterIP   None            <none>        5432/TCP            4m45s
tools-dev-web                   ClusterIP   10.107.42.159   <none>        8080/TCP,2222/TCP   4m45s
tools-dev-worker                ClusterIP   None            <none>        <none>              4m45s
```

We have no external-ips, so no service is exposed. We'll use the normal port forwading we used earlier to access the concource web app. From above, we know that our POD_NAME is `tools-dev-web-8456bb4bc-dlt2z `. 

So to let's portforward our `localhost:8080` to `$POD:8080`. I got the port from the services listed above. Replace pod name below with your pod name.
```sh
kubectl port-forward tools-dev-web-8456bb4bc-dlt2z 8080:8080 --namespace tools-dev
```

Going to `localhost:8080` should show the concourse page.

![concourse](/concourse.png)

🥳 Local Works


## Digital Ocean Deployment
If you are not using Digital Ocean, then you can skip the part of setting up Digital Ocean Part. But you need to have a kubernetes cluster within your cloud provider.


### Setup Digital Ocean Account and Domain Name


First thing we need to do is have a [digital ocean account](https://m.do.co/c/872896dd3f77). Next, add your domain name to digital ocean account. Here's an article that clearly explains how to do this. 

1. [Add domain name to Digital Ocean](https://www.digitalocean.com/docs/networking/dns/how-to/add-domains/)

You then need to go to your Domain Name Provider, and change the nameservers to the ones provided by Digital Ocean. 
```sh
ns1.digitalocean.com
ns2.digitalocean.com
ns3.digitalocean.com
```

Once this is done, it means Digital Ocean will handle traffic coming to `<domain_name>.com`. In my case, I'll be using `gangachris.com`.

### Setup a Digital Ocean Kubernetes Cluster
You can set up digital ocean cluster directly in the Web UI Dashboard, which involves clicking on `CREATE >> Clusters`, and following the prompt. 

I'll however use [Terraform](https://www.terraform.io/), with the [Digital Ocean Terraform provider](https://www.terraform.io/docs/providers/do/index.html), so that next time I want the same cluster up, it will only be a command away. Same applies to destroying the resources created.

Create a directory called `terraform` in your tools directory. Create the following file `main.tf`.
{{< highlight tf "linenos=table" >}}

variable "do_token" {
  type    = "string"
}


provider "digitalocean" {
  token = var.do_token
}

resource "digitalocean_kubernetes_cluster" "tools" {
  name    = "tools"
  region  = "fra1"
  version = "1.14.6-do.1"

  node_pool {
    name       = "worker-pool"
    size       = "s-1vcpu-2gb"
    node_count = 1
  }
}

{{</ highlight >}}

This is a simple terraform config file, that will allow us to create a cluster from within the terraform cli. It represents a Kubernetes Cluster with only one node, and located in frankfurt region. You can get the version of kubernetes to deploy [here](https://www.digitalocean.com/docs/kubernetes/changelog/). 

Instantiate Terraform with:
```sh
terraform init
```

Terraform will download the correct provider, based on the `.tf` file. It will also create a directory `.terraform`, and your directory should have this structure.
```sh
terraform
├── .terraform
│   └── plugins
│       └── darwin_amd64
│           ├── lock.json
│           └── terraform-provider-digitalocean_v1.7.0_x4
└── main.tf

3 directories, 3 files
```

Make sure to get your [Digital Ocean API Access Token](https://cloud.digitalocean.com/account/api/tokens).  Replace `<do_token>` below with your API Access token.

The create the cluster by running.
```
terraform apply -var="do_token=<do_token>"
```


You'll get a nice diff, telling you whats changes are going to be made to your Digital Ocean Instance, 
```sh
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_kubernetes_cluster.tools will be created
  + resource "digitalocean_kubernetes_cluster" "tools" {
      + cluster_subnet = (known after apply)
      + created_at     = (known after apply)
      + endpoint       = (known after apply)
      + id             = (known after apply)
      + ipv4_address   = (known after apply)
      + kube_config    = (known after apply)
      + name           = "tools"
      + region         = "fra1"
      + service_subnet = (known after apply)
      + status         = (known after apply)
      + updated_at     = (known after apply)
      + version        = "1.14.6-do.1"

      + node_pool {
          + id         = (known after apply)
          + name       = "worker-pool"
          + node_count = 1
          + nodes      = (known after apply)
          + size       = "s-1vcpu-2gb"
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

Type in `yes`, and press enter. You cluster should be created withing a few minutes. You can go over to your Digital Ocean Dashboard, and you'll see the cluster being created. 
![digital_ocean](/do.png)

🥳 Cluster Created.

### Switch to remote cluster. 
Once the cluster is created, the last thing we need to do is switch our Kubernetes context to point to our new cluster, from the initial minikube. For digital ocean, the easiest way is to download the `doctl` cli tool and authorize with.
```
doctl auth init
```

Then changing the cluster with below. Remember we name our cluster `tools` in the `main.tf` file. 
```
doctl kubernetes cluster kubeconfig save tools

Notice: adding cluster credentials to kubeconfig file found in "/Users/<user_name>/.kube/config"
Notice: setting current-context to do-fra1-tools
```

Now checking the current cluster should tell us the name of our cluster. 
```sh
kubectl config current-context

do-fra1-tools
```

🥳 Connected to Cluster.

### Install HELM in your cluster
By default helm is instantiated in the `kube-system` namespace. The default user in helm usually doesn't have permission to perform some actions within the cluster, so we need to add a cluster admin. This is achieved through the following commands, and then initialize helm with this service account.

```sh
kubectl create serviceaccount tiller --namespace kube-system
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```


## Ingress Controller
Let's quickly pull in the official definitions:

> #### Ingress
> An API object that manages external access to the services in a cluster, typically HTTP.
> Ingress can provide load balancing, SSL termination and name-based virtual hosting.

And

> #### Ingress Controller
> An Ingress Controller is a daemon, deployed as a Kubernetes Pod, that watches the apiserver's /ingresses endpoint for updates to the Ingress resource. Its job is to satisfy requests for Ingresses.

Since the goal of the article is to deploy these developer tools to our own custom domain, we need a way for our domain names to point to our kubernetes cluster. 

This is where the ingress controller comes in, and in this case, we are going to use the [nginx-ingress](https://github.com/kubernetes/ingress-nginx) to point our host names to the specific ingress instance.

Let's add nginx-ingress as part of our chart dependency. 
{{< highlight yaml "linenos=table" >}}
dependencies:
- name: nginx-ingress
  version: 1.17.1
  repository: "@stable"
  
- name: concourse
  version: 8.2.3
  repository: "@stable"

{{</ highlight >}}

Then download the nginx-ingress chart with
```sh
helm dep update
```

The charts directory will have the following tree
```sh
charts
├── concourse-8.2.3.tgz
└── nginx-ingress-1.17.1.tgz

0 directories, 2 files

```

This is enough to deploy our nginx-controller, but we need to know a little more. Whenever one deploys an ingress controller to any Kubernetes cluster, for any cloud providers, chances are the cloud provider will create a Load Balancer. It's after this instance that you can go ahead and point your domain name to the IP of the load balancer, and boom, done!

This is exactly what will happen, so remember to destroy everything once we are done, because Load Balancers are usually charged for the time they've been on. 

Let's add the following values to our `values.yaml` to configure our nginx-controller. If your `values.yaml` file has some data, feel free to delete everything.
{{< highlight yaml "linenos=table" >}}
nginx-ingress:
  controller:
    scope:
      namespace: tools-dev
    publishService:
      enabled: true

{{</ highlight >}}

These settings will be applied as values to the `nginx-controller` deployment. The values are directly picked from the helm chart documentation. We scope the ingress controller to the `tools-dev` namespace, where we will install out tools.

While at it, let's add the configuration for Concourse. 

{{< highlight yaml "linenos=table" >}}
# values.yaml
nginx-ingress:
  controller:
    publishService:
      enabled: true

concourse:
  web:
    nameOverride: concourse
    env:
      - name: CONCOURSE_EXTERNAL_URL
        value: http://ci.gangachris.com
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: "nginx"
      hosts:
        - "ci.gangachris.com"
{{</ highlight >}}

The above settings for `concourse` can also be found in the HELM Chart documentation. We essentially enable ingress and give it a host domain name to resolve.

Finally, we need to ignore the terraform directory, since it's not part of our chart. You may have different terraform files for other cloud providers. Add the following `.helmignore` file to your directory.
```text
terraform/
```

Now, we can install the chart with
```sh
helm install ./ --name tools-dev --namespace tools-dev
```

Wait a bit, and your chart should be deployed to your cluster. 
```sh
kubectl get pods --namespace tools-dev

NAME                                                       READY   STATUS    RESTARTS   AGE
tools-dev-concourse-7565575c7-dhb87                        1/1     Running   0          27m
tools-dev-nginx-ingress-controller-cb78b8b4d-sxkdj         1/1     Running   0          27m
tools-dev-nginx-ingress-default-backend-56dfdf76b4-nmzrt   1/1     Running   0          27m
tools-dev-postgresql-0                                     1/1     Running   0          27m
tools-dev-worker-0                                         0/1     Pending   0          27m
tools-dev-worker-1                                         0/1     Pending   0          27m
```

And to check the ingress
```sh
kubectl get ingress --namespace tools-dev

NAME                  HOSTS               ADDRESS        PORTS   AGE
tools-dev-concourse   ci.gangachris.com   67.207.76.11   80      28m
```

If you check your digital ocean dashboard, you should see a under network/loadbalancers tab, you should see a Load Balancer provisioning. Once done and healthy, copy the IP Address of the load balancer.
![load balancer](/load_balancer.png)

Go to your domain name and an A record of with 
```
* : <load_balancer_ip_address>
```
![dns a record](/arecord.png)

Once this is done, give it a few seconds, and heading to `ci.yourdomain.com`, will give you the Concourse Default page. In my case it was `ci.gangachris.com`. 

![concourse2](/concourse2.png)

🥳Something works.

## Deploy Grafana in 3 minutes.
Now that we have the infrastructure in place, to add any other tool, such as grafana, we'll follow the same procedure. 

**STEP 1**:  Google `Grafana Helm Chart`, which should direct you to https://github.com/helm/charts/tree/master/stable/grafana

**STEP 2:**  Copy the version from the Chart.yaml file, and add it as a dependency in our `requirements.yaml` file.

{{< highlight yaml "linenos=table" >}}
dependencies:
- name: nginx-ingress
  version: 1.17.1
  repository: "@stable"
  
- name: concourse
  version: 8.2.3
  repository: "@stable"

- name: grafana
  version: 3.8.8
  repository: "@stable"
{{</ highlight >}}
  
Run
```sh
helm dep update
```

Your charts directory should have the following tree.
```sh
charts
├── concourse-8.2.3.tgz
├── grafana-3.8.8.tgz
└── nginx-ingress-1.17.1.tgz

0 directories, 3 files
```

**STEP 3:** Configure grafana in the values.yaml file

{{< highlight yaml "linenos=table" >}}
# values.yaml
nginx-ingress:
  controller:
    publishService:
      enabled: true

concourse:
  web:
    nameOverride: concourse
    env:
      - name: CONCOURSE_EXTERNAL_URL
        value: http://ci.gangachris.com
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: "nginx"
      hosts:
        - "ci.gangachris.com"

grafana:
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: "nginx"
    hosts:
      - "grafana.gangachris.com"
{{</ highlight >}}


**STEP 4:** Upgrade your release with
```sh
 helm upgrade tools-dev ./
```

Check that new deployments have occured.
```sh
kubectl get pods --namespace tools-dev  

NAME                                                       READY   STATUS    RESTARTS   AGE
tools-dev-concourse-7565575c7-dhb87                        1/1     Running   0          54m
tools-dev-grafana-74fcf8f46f-6j9vh                         1/1     Running   0          105s
tools-dev-nginx-ingress-controller-cb78b8b4d-sxkdj         1/1     Running   0          54m
tools-dev-nginx-ingress-default-backend-56dfdf76b4-nmzrt   1/1     Running   0          54m
tools-dev-postgresql-0                                     1/1     Running   0          54m
tools-dev-worker-0                                         0/1     Pending   0          54m
tools-dev-worker-1                                         0/1     Pending   0          54m
```
```sh
kubectl get ingress --namespace tools-dev

NAME                  HOSTS                    ADDRESS        PORTS   AGE
tools-dev-concourse   ci.gangachris.com        67.207.76.11   80      53m
tools-dev-grafana     grafana.gangachris.com   67.207.76.11   80      7s
```

Head over to `grafana.domain.com`. For me its (`grafana.gangachris.com`). 
![grafana](/grafana.png)

🥳 grafana deployed

Go ahead and deploy any other tool you want.

## Conclusion and Some important points. 
- We haven't enabled https. (I will update here (or pass a link))
- We are using a lot of defaults which may not be ideal for some environments. But luckily most helm charts provide configuration to a granular level to help in allocating enough resources to these tools. 
- Remember to tear down everything with.
  ```sh
terraform destroy -var="do_token=..."
  ```
  You may also need to go to Digital Ocean dashboard and delete the Load Balancer, and the Persistent Volumes that were automatically created as a requirements for the above charts.