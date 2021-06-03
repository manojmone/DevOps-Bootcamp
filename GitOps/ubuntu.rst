Setup a local cluster with K3D

We've covered already k3d and k3s in a previous post. In the meantime, k3d has been significantly rewritten and gets only better and better, but some of the commands aren't compatible with the previous versions. For this article we're gonna use the freshly released v3.0.0 by following the installation instructions. Our local environment is an Ubuntu 20.04.

$ sudo wget https://github.com/rancher/k3d/releases/download/v3.0.0/k3d-linux-amd64 -O /usr/local/bin/k3d
$ sudo chmod +x /usr/local/bin/k3d
We verify the setup is correct by running k3d version command:

$ k3d version
k3d version v3.0.0
k3s version v1.18.6-k3s1 (default)
We can install the auto-completion scripts (which are really helpful) using the k3d completion <shell> command and adding it to your shell resources. Eg for linux with bash:

$ echo "source <(k3d completion bash)" >> ~/.bashrc
$ source ~/.bashrc
Let's setup our cluster with 2 worker nodes (--agents in k3d command line) and expose the HTTP load balancer on the host on port 8080 (so that we can interact with our application)

$ k3d cluster create my-cluster --api-port 6443 -p 8080:80@loadbalancer --agents 2
By default, creating a new cluster will:

update your kubeconfig file with the new context, cluster and user details (--update-default-kubeconfig flag)

make it the default (--switch-context flag)

So you can directly use the kubectl command:

$ kubectl cluster-info

