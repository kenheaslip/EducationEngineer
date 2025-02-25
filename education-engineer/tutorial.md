# Deploying Kubernetes Applications with Kind

In this execrise, we will practice deploying applications into Kubernetes. Kubernetes is a popular orchestration platform that is used by many organizations to deploy applications. Kubernetes enables developers to worry less about the infrastructure the applciation is running on and focus more on the things that help the application perform. Do you remember the role a hypervisor plays for it's VM'? That is similar to what Kubernetes is to apps it hosts.

# Before You Get Started

To complete this exercise, you will first need to download and install some apps. The table below lists the apps you need and contains links to the installation instructions for each app.

| Section                 | Description                                                                                                     | Download Link                        |
|-------------------------|-----------------------------------------------------------------------------------------------------------------|--------------------------------------|
| kind                    | kind is a lightweight software package that sets up a local kubernetes cluster on your workstation.             | [kind](https://kind.sigs.k8s.io/) 
| Docker                  | Docker leverages Virtual Machines to allow containerized applications to run on your workstation.               | [Docker](https://docs.docker.com/desktop/?_gl=1*corlfd*_gcl_au*NTgyNTA2NjExLjE3NDAzNjMwNzk.*_ga*OTEwMjk5Njc5LjE3NDAzNjMwNzk.*_ga_XJWPQMJYHQ*MTc0MDQ1NDA5NC4yLjEuMTc0MDQ1NDEwNi40OC4wLjA.)
| kubectl                 | kubectl is the command line utility that you will use to configure and interact with your Kubernetes cluster.   | [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Setup Tips and Common Issues

You may encounter some issues while completing your setup. Don't worry, we have you covered! Use the checklist below to help prevent issues from happening and solve some common issues with these apps.

### Docker Setup Tips
- Docker requires that you have Hypervisor services enabled in your OS. Be sure you have a supported Hypervisor installed and running on your system before running the Docker install.
- Docker also requires that your system BIOS has the appropriate virtualization settings enabled. This setting may vary by CPU and Motherboard manufacturer. Please refer to your manufacturer documentation for the values to enable.
- Bonus hint! For Intel CPU's, the settings is typically "Intel VT-x" and may be located with other advanced CPU settings
- Bonus hint! For AMD CPU's, the settings is typically "AMD-V" and may be located with other advanced CPU settings

### kubectl Setup Tips
- Kubectl is really just an executable file. You need to make sure that it is copied to a location in either your systems (or users) path variable or that you add the folder containing kubectl.exe to your path yourself. Follow the installation instructions for your OS with the link in the table above.
- If kubectl is NOT in your path variable, you will see errors like "not recognized as an internal or external command"

### kind Setup Tips
- kind has it's own software dependencies. Carefully review the install instructions prior to attempting to install your cluster.
- kind has multiple options for how to install it. Choose the one that suits you best but be aware that our examples will use the 'go' installation method.

# Creating Your Cluster
In this section we will be creating the cluster you will use to test your applciation deployments. Kind provides a simple and fully automated process to do this for you. Automations are great for quick excercises like this but it is recommended that you familiarize yourself with how a cluster is manually built. Doing this will help you get the full picture of what it takes to setup a Kubernetes cluster. It will pay off when planning upgrades or troubleshooting issues in the future!

## Creating Your Cluster with Go

Follow the steps below to get started!

1. Open up your CLI (Command Prompt for Windows, Terminal for MacOS or Linux)
2. The first thing we need to do is prepare the go environment by downloading the dependencies needed to create your cluster. To do this, run the command below
```
go install sigs.k8s.io/kind@v0.27.0
```
Your output should be similar to our output below.
```
C:\***\***>go install sigs.k8s.io/kind@v0.27.0
go: downloading sigs.k8s.io/kind v0.27.0
go: downloading github.com/spf13/pflag v1.0.5
go: downloading github.com/spf13/cobra v1.8.0
go: downloading github.com/pkg/errors v0.9.1
go: downloading github.com/mattn/go-isatty v0.0.20
go: downloading al.essio.dev/pkg/shellescape v1.5.1
go: downloading github.com/BurntSushi/toml v1.4.0
go: downloading github.com/evanphx/json-patch/v5 v5.6.0
go: downloading github.com/pelletier/go-toml v1.9.5
go: downloading gopkg.in/yaml.v3 v3.0.1
go: downloading sigs.k8s.io/yaml v1.4.0
go: downloading github.com/google/safetext v0.0.0-20220905092116-b49f7bc46da2
go: downloading github.com/inconshreveable/mousetrap v1.1.0
```
3.  Next up, we will execute the command to have kind create our cluster for us. 

Start kind with the command `kind create cluster` and wait for the setup to complete.

```
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

You should check for connectivity with the Kubernete cluster and the Kubernetes API. A good way to test for connectivity to the cluster and the Kubernetes API is by using the CLI.

```
$ kubectl cluster-info --context kind-kind
```

You should see output that contains the control plane IP address and more. 

## Deploy Application

Create a file named **app.yaml** and insert the following configuration. 

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-app
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: web
  type: NodePort
status:
  loadBalancer: {}
```

The configuration file contains a *deployment* configuration and a *service*. We use the deployment to inform Kubernetes the desired state we want for the application. The application, *web*, also has a service definition that exposes the port of the local cluster node to its external Azure network. The exposed *nodePort* is how you will access the application.

Next, deploy the application.

```shell
$ kubectl apply -f app.yaml
```

Now that the application, web, is deployed we can access the application by exposing the nodePort through port forwarding. However, to port forward the container port the container name is required.

To get the container name, issue the following command:

```shell
$ PODNAME=$(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
```

Now that you have the container name, start the port forwarding with the container to expose the port to the local network.
```
$ kubectl port-forward $PODNAME 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

If you visit localhost:8080 you will see the Hello World welcome page.

# Next Steps

As mentioned before Kubernetes is an orchestration platform used to deploy containerized applications. We hope you now better understand how one can deploy applications to Kubernetes and AWS. 


