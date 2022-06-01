.. _gitops:

.. title:: Kubernetes on a Laptop


++++++++++++++++++++++++++++++++++++++++++
Kubernetes on a Laptop
++++++++++++++++++++++++++++++++++++++++++

Welcome to the third lab of the DevOps Bootcamp!

You will create a Kubernetes cluster on your laptop using Minikube.

.. note::

	Estimated time to complete this lab is 60 minutes


Lab Agenda
+++++++++++

- Configure minikube on the developer desktop


Prerequisites
++++++++++++++

- You will need Docker Desktop installed on your computer.
- You can download it from here - https://www.docker.com/products/docker-desktop
- You will need an Github account. Make sure you don't use the enterprise Github.
- If you don't have a Github account, you can create one here - https://github.com/

Setup a local cluster with Minikube
++++++++++++++++++++++++++++++++++++

Minikube is an open-source tool which provides you with a one-node cluster where both master processes and worker processes run inside that node. You can easily install Minikube on your computer and test your application using it. Minikube provides you pre-installed Docker as your container runtime so that you don’t need to install Docker separately.

We will be using our Laptop Macbookas our host for the Kubernetes cluster. The prerequisite for this step is to ensure that you have docker desktop installed on your computer.


On your terminal window, follow these steps to install minikube -
The instructions in this lab assume you are using a Macbook. If you are using Windows, follow these steps for the installation of minikube -

Installing Minikube For Windows 
................................

Download and run the stand-alone minikube Windows installer available here - https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe


Installing Minikube For Mac 
............................

Install Homebrew -

.. code-block:: bash

 $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


To install minikube on x86-64 Linux using binary download:

.. code-block:: bash

   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

.. code-block:: bash

  sudo install minikube-linux-amd64 /usr/local/bin/minikube

For a Mac, follow these steps -

We will be using Homebrew package manager. If you don't have it alreay installed on your mac, you can install it by running thi scommand in your terminal - 

.. code-block:: bash

  curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh


Update Homebrew - 

.. code-block:: bash

  $ brew update


Once you’re done with Homebrew, you will have to select a virtual machine manager to install Minikube.
Hyperkit is what we will use for this lab as it is easy to install with Homebrew. To install Hyperkit, run the below command on your terminal.

.. code-block:: bash

  brew install hyperkit

To install Minikube run the below command on your terminal.

.. code-block:: bash

  $ brew install minikube

  $ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64

  $ sudo install minikube-darwin-amd64 /usr/local/bin/minikube

Verify Minikube Installation 
.............................

We verify the setup is correct by running minikube command:

.. code-block:: bash

  $ minikube start --vm-driver=hyperkit

This command may take some time to complete. 

Now we’re almost done with the minikube installation. Next, you can try out different kubectl commands to get outputs. 
Run the below command to get your nodes inside the cluster.

.. code-block:: bash

  kubectl get nodes

The Kubernetes Client
++++++++++++++++++++++

The official kubernetes client is `kubectl`

`kubectl` can manage: pods, replicasets and services. You can also explore the overall health of a cluster.

Checking Cluster Status
+++++++++++++++++++++++
Get version

.. code-block:: bash
    kubectl version


Tells you the client and server version. They can be different versions as long as they are within 2 major versions.

Get componentstatuses:

.. code-block:: bash
    kubectl get componentstatuses

* `controller-manager` - regulates behaviour ensures components are healthy
* `scheduler` - places different pods on different nodes
* `etcd` server - storage for api objects

List Worker Nodes
++++++++++++++++++
    kubectl get nodes

    NAME       STATUS   ROLES    AGE   VERSION
    minikube   Ready    <none>   30m   v1.16.2

* `master` nodes contain the API server and scheduler
* `worker` nodes are where your container run

Get info about a specific node:

    kubectl describe nodes <nodename>
    kubectl describe nodes minikube

Get the:

* Operations
* Disk and Memory Space
* Software info: Docker, kubernetes and Linux Kernel versions
* Pod Information - You can get name, CPU and memory of each pod - requests and limits also tracked

## Cluster Components

Many of the components that make up the kubernetes cluster are deployed using kubernetes itself.
They run in the `kube-system` namespace

### Kubernetes Proxy

* Responsible for routing traffic to load balanced services
* Must be present on every node (uses `Daemonset` for this)

View the proxies:

    kubectl get daemonSets --namespace=kube-system kube-proxy

### Kubernetes DNS

* Naming and discovery for services
* DNS service is run as a `deployment`

Get the DNS deployment:

    kubectl get deployments --namespace=kube-system coredns

Get service that load balances dns:

    kubectl get services --namespace=kube-system kube-dns
    
    NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   28h

> It might be `core-dns`, `coredns` or `kube-dns` on other systems. Kubernetes 1.12 moved from `kube-dns` to `core-dns`

If you check a container in a cluster the cluster ip `10.96.0.10` will be in `/etc/resolv.conf`

## Kubernetes UI

The final component is the GUI. A single replica managed by kubernetes.

You can see it with:

    kubectl get deployments --namespace=kube-system kubernetes-dashboard

> On `minikube version: v1.5.0` it is in its own namespace

    kubectl get deployments --namespace=kubernetes-dashboard kubernetes-dashboard

and

    kubectl get services --namespace=kube-system kubernetes-dashboard

You can use `kubectl proxy` to access the UI

You can then access the service at: [http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)

Working with Kubernetes
++++++++++++++++++++++++

Let's begin with Pods, So what are Pods?

- A pod is the lowest unit of an application in Kubernetes
- A pod is not equal to a container in the Docker world
- A pod can be made up of multiple containers. 

So do we have any pods running?

To check if this worked, run the following command again

.. code-block:: bash

  kubectl get pods

If this is being run on a fresh Minikube installation the command will return no pods. Do how do pods get added? Let's try an add a ready server called kuard. To run kuard server, use:

.. code-block:: bash

  kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:1

To check if this worked, run the following command again

.. code-block:: bash

  kubectl get pods

So how do these pods get defined? They can be defined using the Pod manifest. Here's a manifest you can create -

.. code-block:: bash

  apiVersion: v1
  kind: Pod
  metadata:
    name: kuard1
  spec:
    containers:
      - image: gcr.io/kuar-demo/kuard-amd64:blue
        name: kuard1
        ports:
          - containerPort: 8080
            name: http
            protocol: TCP

Save the file and name it kuard1-pod.yaml. Then you can apply this file based details to your pod, by using this command -

.. code-block:: bash

  kubectl apply -f kuard1-pod.yaml  

To check if this worked, run the following command again

.. code-block:: bash

  kubectl get pods
