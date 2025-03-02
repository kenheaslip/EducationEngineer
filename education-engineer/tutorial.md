# Deploying Kubernetes Applications

Kubernetes is a container orchestration platform that provides manegement, scalability, and greater deployment control of containers.

Some important Kubernetes features are:

- Ability to automate scaling of your container
- Greater control over the security of your environment
- Ability to self heal your deployments

This exercise walks through the creation of a small kubernetes cluster and deployment of a simple application into it. Throughout the exercise, we will be sharing some tips, tricks, recommendations, and explanations of what we are deploying.

Topics for this exercise:

- Prerequisites
- Creating your Cluster
- Preparing your Cluster
- Deploying your Application
- Cleanup
- Next Steps

# Prerequisites

This section outlines software you need to install and configure in order to copmlete the exercise. Use the table below for links to installation and configuration guides for each required application.

| Software                | Description                                                                                                                  | Download Link                        |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| kind                    | kind is software/package of scripts that sets up a kubernetes cluster on your workstation using docker containers as nodes   | [kind](https://kind.sigs.k8s.io/)
| Docker                  | Docker leverages Virtual Machines to allow containerized applications to run on your workstation.                            | [Docker](https://docs.docker.com/desktop/?_gl=1*corlfd*_gcl_au*NTgyNTA2NjExLjE3NDAzNjMwNzk.*_ga*OTEwMjk5Njc5LjE3NDAzNjMwNzk.*_ga_XJWPQMJYHQ*MTc0MDQ1NDA5NC4yLjEuMTc0MDQ1NDEwNi40OC4wLjA.)
| kubectl                 | kubectl is the command line utility that you will use to configure and interact with your Kubernetes cluster.                | [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Setup Tips and Common Issues

You may encounter issues while completing your setup. Don't worry, we have you covered! Review the tips below

### Docker Setup Tips

- For Windows users only. Docker requires a Hypervisor to operate. Ensure you have a supported Hypervisor, like WSL or Hyper-V, installed on your system before beginning the Docker installation.
- Docker also requires virtualization support to be enabled in your system BIOS/Fimrware. This setting vaires CPU and motherboard manufacturer. Please refer to your manufacturer documentation for more information.
- Bonus hint! For Intel CPU's, the settings is typically "Intel VT-x" and may be located with other advanced CPU settings
- Bonus hint! For AMD CPU's, the settings is typically "AMD-V" and may be located with other advanced CPU settings

### kubectl Setup Tips

- Kubectl is an executable file. Make sure it is copied to a location in your system or user path variable. Alternatively, you may add the folder containing kubectl.exe to your path yourself. Follow the installation instructions for your OS with the link in the table above.
- If kubectl is NOT in your path variable, you will see errors like "not recognized as an internal or external command"

### kind Setup Tips

- kind has it's own software dependencies. Carefully review the install instructions prior to attempting to create your cluster.
- kind has multiple installation options. Choose one that suits you best. The examples in this exercise will use the 'go' installation method.

# Creating Your Cluster

In this section, we will be creating your cluster using kind. Kind provides a simple and fully automated process for cluster creation. Automations are ideal for quick excercises but we recommended familiarizing yourself with manual cluster build processes. This will teach you more about how a Kubernetes cluster is created and will be beneficial in the future!

## Creating Your Cluster with Go

Follow the steps below to get started.

1. Open your CLI (Command Prompt for Windows, Terminal for MacOS or Linux)
2. Ensure your CLI utilities are installed correctly. To do this, execute the commands below and confirm your results are similar.

go command and expected output

>**Note**: Your version may be different. This is ok.

```shell
go version
```

```shell
C:\****\****>go version
go version go1.24.0 windows/amd64
```

kubectl command and expected output

```shell
kubectl version
```

```shell
C:\****\****>kubectl version
Client Version: v1.31.4
Kustomize Version: v5.4.2
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

>**Note**: You have not created your cluster yet. Your results will show "unable to connect to the server. This is expected and will be resolved after you create your cluster.
  
3. Prepare your go environment. The command below will download and install kind and it's dependencies.

```shell
go install sigs.k8s.io/kind@v0.27.0
```

Your results should be similar to the output shown below.

```shell
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

4. Execute the command below to create your cluster.

```shell
kind create cluster
```

Your results should be similar to the output shown below.

```shell
C:\****\****>kind create cluster
Creating cluster "kind" ...
 • Ensuring node image (kindest/node:v1.32.2) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
 • Preparing nodes 📦   ...
 ✓ Preparing nodes 📦
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Notice the output only references creation of the control-plane. This is fine in small labs (like this one) or any scnenarios where you are testing basic functionality. In a full Kubernetes environment, "worker nodes" would also be provisioned. Worker nodes are where your applications are deployed and they provide the compute resoureces needed for your application. 

Check out the [Official Kubernetes Architecture Page](https://kubernetes.io/docs/concepts/architecture/) for more information on Kubernetes architecture .

5. Confirm your cluster is operational by using the command below to query the cluster nodes.

```shell
kubectl get nodes
```

Your results should be similar to the output shown below.

>**Note**: Depending on your system resources, this may take a few minutes. Periodically execute the command until all is well.

```shell
C:\****\****>kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   28s   v1.32.2
```

To further confirm your cluster is healthy, you can execute the command below.

```shell
kubectl cluster-info --context kind-kind
```

Your results should be similar to the output shown below.

```shell
C:\****\****>kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:51397
CoreDNS is running at https://127.0.0.1:51397/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# Preparing Your New Cluster

In this section you will:

- Learn the importance of namespaces
- Create a new namespace for your application

## Create a New Namespace

A namespace is a logical construct that isolates your applications and resources. Namespaces can have multiple types of configurations applied (such as network and securtiy policies) to give you control over all resources that are deployed in it.

Think of a storage facility. Imagine the whole building as a single namespace. We know namespaces are able to exist across multiple nodes (VM's) so each individual storge unit will be a worker node, and the items in the units will be pods. Now we are looking at one namespace (the building) that spans across many nodes (the individual units) and operates many pods (the unit contents). This architecture is how Kubernetes can use highly available infrastructure to run your workloads.

Below is a tree diagram of how everything connects.

- cluster
  - namespace 1 (the storage building)
    - node 1 (storage unit 1)
      - pod 1
      - pod 2
    - node 2 (storage unit 2)
      - pod 1
      - pod 2

Your cluster has a few prebuilt namespaces. All of them contain core cluster services except for one, the "default" namespace. It is best practice to avoid deploying to the default namespace as it could lead to issues.

For example, you could end up with unexpected changes to the default namespace limits during a Kubernetes upgrade. If you are using a cloud K8s cluster with node scaling, your app could endlessly scale up resulting in a very expensive cloud services bill.

1. Execute the command below to create a new namespace named 'mylab'.

```shell
kubectl create ns mylab
```

Your results should be similar to the output below.

```shell
C:\*****\****>kubectl create ns mylab
namespace/mylab created
```

2. Execute the command below to ensure the 'mylab' namespace has been created successfully.

```shell
kubectl get ns
```

Your results should be similar to the output below.

>**Note**: Note the age for the namespace we created, "mylab", and that it's status is "active".

```shell
C:\****\****>kubectl get ns
NAME                 STATUS   AGE
default              Active   23m
kube-node-lease      Active   23m
kube-public          Active   23m
kube-system          Active   23m
local-path-storage   Active   23m
mylab                Active   3s
```

# Deploying your Application

In this section, you will:

- Deploy an Application to your Cluster
- Deploy a Kubernetes Service
- Configure Networking for App Access
- Other Common Configurations

## Deploy an Application to your Cluster

The Kubernetes deployment resource defines the state of how your application will operate in the environment. It is where we define things like how many replicas should be configured, what container image to use, and overall management of the application pods. 

For more information on Kubernetes deployments, visit the [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) web site.

Let's Deploy!

1. Load your code editor of choice and create an empty yaml file. Let's name it testAppDeploy.yaml
2. Copy and paste the yaml below into your editor. Pay attention to the comments in the file. They provide a brief explanation of what the configs are doing.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web                                           # Places a label on all pods for easy identification of your app.
  name: web                                            # defines the name of the deployment. For the sake of simplicity it's best to name it the same as your app label
spec:
  replicas: 1                                          # How many replicas you require. Other configs like Horizontal Pod Autoscaling (HPA) can be used to scale your pods dynamically
  selector:
    matchLabels:
      app: web                                         
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0     # the location of the image you want to use for your app. This can be a secure private repo, public repo, etc. Ensure your network allows access to the repo. If the repo is public, scrutinize the source to ensure you trust it.
        name: hello-app                                # the name of the application.
        imagePullPolicy: IfNotPresent                  # The deployment will only pull the image if it does not already exist. This eliminates unecessary compute, network traffic, and potential delay in starting your pods.
        ports:
        - containerPort: 8080                          # tells the deployment what port the container will be listening on

```

 >**Heads up**:Any configuration you apply, command you execute, etc will target the default namespace unless you explicitly define the namespace in your command or you set the namespace context in your kube config. Setting the namespace context will change the default namespace to whatever value you define. Always be aware of your context setting to avoid accidents. It may be less risky to get in the habit of explicitly defining your target namespace.

If you want to set your namespace context, run the below command.

```shell
kubectl config set-context --current --namespace=mylab
```

Your result should look similar to the output below.

```shell
C:\****\****>kubectl config set-context --current --namespace=mylab
Context "kind-kind" modified.
```

Kubernetes configurations are definitions of the state the resource will be in after the configuration is applied. 

3. Execute the command below to apply your depoloyment configuration.

>**Note**: Make sure your CLI PWD (present working directory) is set to the folder that contains the testAppDeploy.yaml file or that you are qualifying the path to the file.

```shell
kubectl apply -f testAppDeploy.yaml -n mylab
```

>**Note**: if you opted to change your context, you do not need to include the namespace identifier "-n mylab".

Your result should look similar to the ouput below.

```shell
C:\****\****\>kubectl apply -f testAppDeploy.yaml -n mylab
deployment.apps/web created
```

>**Another Note**: If you made mistakes you can update your yaml file and re-run the apply command. If you do this, your result will change from "created" to "configured. If you just want to start over, you can re-run the apply command but substitute "apply" with "delete". This will tell Kubernetes to remove applied configurations associated with the yaml file you specify.

5. There are two places you can check to make sure your application is opertating as expected.

Check pod status in the namespace where you deployed your app. To do this, execute the command below.

```
kubectl get pods -n mylab
```

Your result should look similar to the output below.

```
C:\****\****\>kubectl get pods -n mylab
NAME                   READY   STATUS    RESTARTS   AGE
web-75995f7dbf-4ww7k   1/1     Running   0          39s
```

Notice the pod has the name you defined in line 6 of your testAppDeploy.yaml file. The rest of the name is randomly assigned to prevent issues with duplicate pods running as your deployment scales up.

The other place you can check on how your deployment is working is the deployment itself. Execute the command below to query your deployment state. Don't forget to specify your namespace!

```shell
kubectl get deployments -n mylab
```

Your result should look similar to the output below.

```shell
C:\****\****>kubectl get deployments -n mylab
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           4h17m
```

This command will show the status of all the deployments currently in the targeted namespace. This is just a short summary. For more info you can add "-o wide" to the command. For more detailed information on the state of a deployment, we can check logs, events, and a description of the state of the deployment.

Query your cluster for your deployment state by executing the command below. Use the name of the deployment from the last command we ran.

```shell
kubectl describe deployment web -n mylab
```
Your results should look similar to the below output.

```shell
C:\****\****>kubectl describe deployment web -n mylab
Name:                   web
Namespace:              mylab
CreationTimestamp:      Tue, 25 Feb 2025 16:12:58 -0500
Labels:                 app=web
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=web
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=web
  Containers:
   hello-app:
    Image:         gcr.io/google-samples/hello-app:1.0
    Port:          8080/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   web-6c7ccf7dbb (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  10s   deployment-controller  Scaled up replica set web-6c7ccf7dbb from 0 to 1
```

>**Note**: What you configured in your deployment is only a small portion of what you can do. For more information, visit the [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) web site.

## Deploying a Kubernetes Service

Your application can increase or decrease the number of pods supporting it on demand, resulting in frequent changes to pod names and their IP addresses. The service acts as load balancer that keeps track of how many pods your deployment has and the information it needs to send traffic to them.

>**Note**: For more information visit the [Official Kuberentes Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/) web site.

Let's get your service configuration deployed!

1. Load your code editor of choice and create an empty yaml file. Let's name it testAppService.yaml
2. Copy and paste the yaml below into your empty testAppService.yaml file and save the changes.

```yml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: web                   # Applies the label to the service resource
  name: web                    # The name the service will use
spec:
  ports:
  - port: 8080                 # The port number your the service will use to accept inbound connections
    protocol: TCP              # Specifying the protocol that communications will use. Could be UDP, HTTP, etc
    targetPort: 8080           # The port number your pods will be listening on
  selector:
    app: web                   # Tells the service which pods are targetted by the service
  type: NodePort               # Tells Kubernetes to  setup a NodePort mapping so your app can be reached from outside the cluster
```

3. Apply your service config using the command below. Don't forget your namespace identifier!

```shell
kubectl apply -f testAppService.yaml -n mylab
```

Your result should look similar to the output below.

```shell
C:\Users\seph\Documents\Spectro>kubectl apply -f testAppService.yaml
service/web created
```

4. Execute the command below to confirm the status of your service.

```shell
kubectl get service -n mylab
```

Your result should look similar to the output below.

```shell
C:\Users\seph\Documents\Spectro>kubectl get service -n mylab
NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.96.95.69   <none>        8080:30278/TCP   19s
```



## Configuring Network Access

The last thing you need to do is expose your container to your local network. To do this we will setup a port forward rule that maps the listening port on your workstation to the port we assigned your service in the configuration file. There are other targets we can use for port-forwarding like pods, deployments, etc.

1. Execute the command below to apply your port forward rules. Make sure you don't break out of the results after the command runs.

```shell
kubectl port-forward service/web 8080:8080
```

>**Note**: port-forwarding is temporary. Once you break out of the result in your command prompt, you will lose connectivity to the application. For permanent access, you would deploy an ingress gateway, which will be covered in future, more advanced, content.

Your result should look like the output below.

```shell
C:\****\****>kubectl port-forward service/web 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

2. To test your application, open a web browser and navigate to localhost:8080. Alternatively, if you have curl installed, you can curl localhost:8080. Whether you use your browser or curl, you should see the following text confirming you have successfuly completed this activity!

```
Hello, world!
Version: 1.0.0
Hostname: web-6c7ccf7dbb-z4rdn
```

>**Note**: Notice the output is identifying a hostname. This is the name of the pod that has responded to your request. If you were to scale up your deployment and continually refresh your browser, you would notice the hostname changing between the avialable pods for your application.

# Cleanup

Wow, we have loaded a lot of stuff onto your workstation. You have the option to keep the work you completed if you wish, in which case, skip ahead to the "Next Steps" section.

If you want to clean up, here is a bit of help to remove everything we just installed.

|          Applciation to Remove                         |            How to Remove It                                                  | Should I just keep it to play with Kubenetes and kind in the future?             |
---------------------------------------------------------|------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| the cluster you just built and all its configs         | execute command "kind delete cluster"                                        | Absolutely!                                                                      |
| go                                                     | Follow these [steps](https://go.dev/doc/manage-install#uninstalling)         | Absolutely!                                                                      |
| docker desktop                                         | Follow these [steps](https://docs.docker.com/desktop/uninstall/)             | You guessed it, Aboslutely!                                                      |

# Next Steps

Thank you for taking the time to complete this activity! If you want to learn more about what we did, what other configurations options there are, or things that were not covered, check out the links below!

Things we did or discussed:

- [Official Kubernetes Architecture Page](https://kubernetes.io/docs/concepts/architecture/)
- [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Official Kuberentes Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

Next topics to dive into:

- [Official Kubernetes Gateway Documentation](https://kubernetes.io/docs/concepts/services-networking/gateway/)
- [Official Kubernetes Horizontal Pod AutoScaler documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
