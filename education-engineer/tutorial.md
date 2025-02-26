# Deploying Kubernetes Applications

Kubernetes is a popular container orchestration platform that is used by many organizations to deploy applications. Kubernetes provides the ability for developers and production support teams to gain fine tuned control over how their containers are ran. 

Some of the stand out Kubernetes features are:
- Ability to automate scaling of your container
- Greater control over the security of your environment
- Ability to self heal your deployments

In this exercise we will be walking through the detailed steps to create a small scale kubernetes cluster on your workstation. We will use this environment to deploy a small sample application and of course, we will be sharing some tips, tricks, recommendations, and explanations of what we are setting up and why it matters. 

Our topics for this exercise will be:
- Installing and configuring pre-requisite software and workstation settings
- Setup Tips and Common Issues
- Creating Your Cluster with Go
- Deploying a pre-made sample application

## Before You Get Started

To complete this exercise, you will first need to download and install some pre-requisite apps. The table below lists the apps you need and contains links to the installation instructions for each app.

| Software                | Description                                                                                                                  | Download Link                        |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| kind                    | kind is software/package of scripts that sets up a kubernetes cluster on your workstation using docker containers as nodes   | [kind](https://kind.sigs.k8s.io/) 
| Docker                  | Docker leverages Virtual Machines to allow containerized applications to run on your workstation.                            | [Docker](https://docs.docker.com/desktop/?_gl=1*corlfd*_gcl_au*NTgyNTA2NjExLjE3NDAzNjMwNzk.*_ga*OTEwMjk5Njc5LjE3NDAzNjMwNzk.*_ga_XJWPQMJYHQ*MTc0MDQ1NDA5NC4yLjEuMTc0MDQ1NDEwNi40OC4wLjA.)
| kubectl                 | kubectl is the command line utility that you will use to configure and interact with your Kubernetes cluster.                | [kubectl](https://kubernetes.io/docs/tasks/tools/)

## Setup Tips and Common Issues

You may encounter some issues while completing your setup. Don't worry, we have you covered! Use the checklist below to help prevent issues from happening and solve some common issues with these apps.

### Docker Setup Tips
- For Windows users only. Docker requires that you have Hypervisor services enabled in your OS. Be sure you have a supported Hypervisor installed and running on your system before running the Docker install.
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
In this section we will be creating the cluster you will use to test your applciation deployments. Kind provides a simple and fully automated process to do this for you. Automations are great for quick excercises like this but it is recommended that you familiarize yourself with how a cluster is manually built. Doing this will help you get the full picture of what it takes to setup a Kubernetes cluster. It will pay off when the time comes to plan upgrades or troubleshoot issues in the future!

## Creating Your Cluster with Go

Follow the steps below to get started!

1. Open up your CLI (Command Prompt for Windows, Terminal for MacOS or Linux)
2. The first thing we need to do is make sure your CLI utilities are installed correctly. To do this, execute the commands below and confirm your output is similar and that you are not recieving any error messages.

go command and expected output (it's ok if your version is different)
```
go version
```
```
C:\****\****>go version
go version go1.24.0 windows/amd64
```

kubectl command and expected output
```
kubectl version
```
```
C:\****\****>kubectl version
Client Version: v1.31.4
Kustomize Version: v5.4.2
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

>**Note**: If you have just setup kubectl and have not connected to any clusters, the "unable to connect to the server" error will be displayed. Don't worry, this is ok! We are only validating that kubectl is installed correctly. The kind installation will update our kube config file for us automatically.  
  
3. Next we need to prepare the go environment by downloading the dependencies needed to create your cluster. To do this, run the command below.
```
go install sigs.k8s.io/kind@v0.27.0
```
Your results should be similar to the output shown below.
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

4.  Next up, we will execute the command below to have kind create our cluster for us.
```
kind create cluster
```
Your results should be similar to the output shown below.
```
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
>**Note**: Notice the output from the installation script is only referencing the creation of the control-plane. This is fine for use with small labs (like this one) or any scnenarios where you are testing very basic functionality. In a real environment, Kubernetes would also have "worker nodes" built and those nodes are where your applications would be deployed. Check out the [Official Kubernetes Architecture Page](https://kubernetes.io/docs/concepts/architecture/) for more information on Kubernetes architecture .

5.  We now have a Kubernetes cluster setup but, before we continue, let's make sure everything is working properly. Use the command below to query the cluster nodes and see if our control plane has started.
```
kubectl get nodes
```
If everything completed sucessfully, you should get a response similar to the output below. The key here is ensuring the control plane node status is "ready". Depending on your system resources, this may take a few minutes. Periodically execute the command until all is well.
```
C:\****\****>kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   28s   v1.32.2
```
To further confirm your cluster is healthy, you can execute the command below.
```
kubectl cluster-info --context kind-kind
```
Your results should be similar to the output shown below.
```
C:\Use\****>kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:51397
CoreDNS is running at https://127.0.0.1:51397/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

# Preparing Your New Cluster
Congratulations! You have successfully completed installation/configuration of pre-requisite software and built your local kubernetes cluster.

In this section we will be reviewing:
- Recommended configurations for your environment
- How to deploy your application
- Configurations your application needs to operate and be reachable on your network. We will be using a sample application from Google repositories to keep things simple.

## Create a New Namespace
What is a namespace? In Kubernetes, a namespace is a logical construct that is used to isolate your applications from other ones. It serves as a base where security and network policies can be applied to give you control over any applictions that are deployed into it. Since a namespace is a logical construct, let's try and line this up with a real world example.

Let's picture a storage unit. Yes, those places where you can rent individual units. Let's picture the whole building as a namespace. We know namespaces are able to exist across multiple nodes (VM's) so let's call each individual storge unit inside the building a worker node, and the items in the units, pods. Now we are looking at one namespace (the building) that is spanning across many nodes (the individual units) and running many pods (the unit contents). This is the basis on which Kubernetes is able to guarnatee high availablity for your application workloads.

How about a picture. Below is a tree diagram of how everything connects. 

- Cluster 
  - namespace 1 (the storage building)
    - node 1 (storage unit 1)
      - pod 1
      - pod 2
    - node 2 (storage unit 2)
      - pod 1
      - pod 2

Your cluster will come with a few namespace prebuilt. Most of them contain core services your cluster needs to run except for one. The "default" namespace. It's best practice to avoid deploying apps in the default namespace in your cluster. If you use default, you may encounter issues in the future where a cluster version upgrade modifies settings you have applied and creates outages or performance impacts for your apps. To avoid this we will need to create a new namespace for the applciations we want to deploy.

An example of an issue you could run into is:
Changes to the default namespace limits or quotas:
- A cluster upgrade removes or modifies the limits you set on the default namespace. If you are using a cloud K8s Cluster with node scaling, your app could endlessly scale up and surprise you with a very expensive bill.

1. To create a new namespace named 'mylab', execute the command below.
```
kubectl create ns mylab
```

Your results should be similar to the output below.
```
C:\*****\****>kubectl create ns mylab
namespace/mylab created
```

2. Let's check to make sure our namespace is created by executing the command below.
```
kubectl get ns
```

You should be able to see all the namespaces that exist on your cluster. Note the age for the namespace we created, "mylab", and that it's status is "active".
```
C:\****\****>kubectl get ns
NAME                 STATUS   AGE
default              Active   23m
kube-node-lease      Active   23m
kube-public          Active   23m
kube-system          Active   23m
local-path-storage   Active   23m
mylab                Active   3s
```

# At Last, Deploying Your App!
We are now ready to deploy your application! 

In this section, we will complete the following:
- Deploying a Kubernetes.... Deployment
- Deploying a Kubernetes Service
- Configuring Networking for App Access
- Other Common Configurations

## Deploying a Kubernetes... Deployment
The Kubernetes deployment resource defines the state of how your application will run in the environment. It is where we define things like how many replicas should be running, what image will be used for the containers, and overall management of the pods running your application. For more information on Kubernetes deployments, visit the [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) web site.

Let's deploy!

1. Load your code editor of choice and create an empty yaml file. Let's name it testAppDeploy.yaml
2. Copy and paste the yaml below into your editor. Pay attention to the comments in the file. They will provide a brief explanation of what the configs are doing.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web                                           # Places a label on all pods for easy identification of your app.
  name: web                                            # defines the name of the deployment. For the sake of sanity, it's best to name it the same as your app label
spec:
  replicas: 1                                          # How many replicas you want to run. Be aware this is just a starting state. Other configs like Horizontal Pod Autoscaling (HPA can be used to scale your pods dynamically)
  selector:
    matchLabels:
      app: web                                         
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0     # the location of the image you want to use for your app. This can be a secure private repo, public, etc. Just make sure your network allows access to the repo. If its a public repo, scrutinize the source to make sure its trusted.
        name: hello-app                                # the name of the application being ran.
        imagePullPolicy: IfNotPresent                  # tells the deployment to only pull the image if it does not already exist. This eliminates unecessary compute, network traffic, and potential delay in starting your pods.
        ports:
        - containerPort: 8080                          # tells the deployment what port the container will be listening on

```

3. Remember when we talked about not deploying apps to the default namepsace? This is where we make sure that doesn't happen. Any configuration you apply, command you run, etc will target the default namespace unless you either explicitly define the namespace in your command or you set the namespace context in your kube config.

Setting the namespace context will change the default namespace to whatever value you define. This can be handy but make sure you always know what your default is. It may be less risky to get in the habit of explicitly defining your target namespace. If you want to set your namespace context, run the below command.
```
kubectl config set-context --current --namespace=mylab
```
Your result should look similar to the output below.
```
C:\****\****>kubectl config set-context --current --namespace=mylab
Context "kind-kind" modified.
```

4. Kubernetes views its configurations, including app deployments, as a definition of resource state. As such, the commands it uses to do the work reflect that. In this case, deploying your application is actually applying a configuration state for a deployment resource. Make sure your CLI PWD (present working directory) is set to the folder that contains the testAppDeploy.yaml file. 

To apply your configuration, run the command below.

```
kubectl apply -f testAppDeploy.yaml -n mylab
```
>**Note**: if you opted to change your context, you do not need to include the namespace identifier "-n mylab".

Your result should look similar to the ouput below.
```
C:\Users\seph\Documents\Spectro>kubectl apply -f testAppDeploy.yaml -n mylab
deployment.apps/web created
```
>**Another Note**: If you made mistakes you can simply update your yaml file and re-run the apply command. If you do this, your result will change from "created" to "configured. If you just want to start over, you can re-run the apply command but substitute "apply" with "delete". This will tell Kubernetes to remove applied configurations associated with the yaml file you specify.

5. Let's confirm your pod is deployed. There are two places you can check to make sure your app is running as expected. The first is to check for running pods in the namespace where you deployed your app. To do this, run the command below.
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

```
kubectl get deployments -n mylab
```
Your result should look similar to the output below.

```
C:\****\****>kubectl get deployments -n mylab
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           4h17m
```
This command will show the status of all the deployments currently in the targeted namespace. This is just a short summary. For more info try adding "-o wide" to the command. Still not enough? For more detailed information on the state of a deployment, we can check logs, events, and most important when starting any troubleshooting, a description of the state of the deployment.

Query your cluster for your deployment state by executing the command below. Use the name of the deployment from the last command we ran for the deployment name. If you followed instructions, your deployment name should be "web".

```
kubectl describe deployment web -n mylab
```

Your results should look similar to the below output.

```
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
>**Note**: What we configured in our deployment is only a small portion of what we can do. For more information, visit the [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) web site.

## Deploying a Kubernetes Service
The Kubernetes service resource plays a crucial role for applications in your cluster. Let's discuss the purpose of the service resource and why it's needed for every application you deploy.

Ingress access to your application - The service resource can be configured to define how your application will be accessed. Think of this as connecting a network cable to your application and controlling where the other end of the connection can terminate. In this case, we are stating "nodeport" as the connection method. When the "nodeport" value is set, Kubernetes will direct all traffic into your service by mapping a de'fined port number on the node to the port of your application (port forwarding). There will be examples of this in the "Deploy Your Service" section.

Internal cluster routing to your pods - When you deploy your application into Kubernetes, it runs in a resource called a pod. The number of pods that run (replicas) is controlled in your deployment file and can be set to 1 pod, 22 pods, 100 pds, etc. These pods can start and stop many times a day resulting in frequent internal IP changes and pod names for your workloads. The service resource is tied to your deployment and keeps track of how many pods are running and the information it needs to send traffic to them. Try to view the Kubernetes service resource as more of a dynamic load balancer that is always aware of where your application can be reached.

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
  type: NodePort               # Tells Kubernetes to configure the nodes running your pods to setup a NodePort mapping so your app can be reached from outside the cluster

```

3. Let's apply your service config using the command below. Don't forget your namespace identifier!

```
kubectl apply -f testAppService.yaml -n mylab
```

Your result should look similar to the output below.
```
C:\Users\seph\Documents\Spectro>kubectl apply -f testAppService.yaml
service/web created
```

To confirm the status of your newly deployed service, execute the command below.
```
kubectl get service -n mylab
```

Your result should look similar to the output below.
```
C:\Users\seph\Documents\Spectro>kubectl get service -n mylab
NAME   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.96.95.69   <none>        8080:30278/TCP   19s
```
>**Note**: There are other service configurations available for you to use if you need them. For more information visit the [Official Kuberentes Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/) web site.

##Configuring Network Access

Almost done! The last thing we need to do is expose your container to your local network. To do this we will setup a port forward rule that maps the listening port on your workstaion to port we assigned to your service in your configuration file. There are other targets we can use for port-forwarding like pods, deployments, etc. We are using service because we have configured a service to handle connectivity to our application. This will ensure that if we scale our deployment, all traffic will route to the intended pods regardless of how many replicas we are running.

1. Execute the command below to configure your port forward rules. Make sure you don't break out of the results after the command runs.

```
kubectl port-forward service/web 8080:8080
```
>**Note: Using this method for port-forwarding is temporary. Once you break out of the result in your command prompt, you will lose connectivity to the application. For more permanent access, you would deploy an ingress gateway, which will be covered in future, more advanced, content.

Your result should look like the output below.
```
C:\****\****>kubectl port-forward service/web 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```

2. We are now ready to test connectivity to your application! Simply open a web browser and navigate to localhost:8080. Alternatively, if you have curl installed in your CLI, you can curl localhost:8080. Whether you use your browser or curl, you should see the following text confirming you have successfuly completed this activity!

```
Hello, world!
Version: 1.0.0
Hostname: web-6c7ccf7dbb-z4rdn
```
>**Note**: Notice the output is identifying a hostname. This is the name of the pod that has responded to your request. If you were to scale up your deployment and continually refresh your browser, you would notice the hostname changing between the pods your application is runnin.


# Time to Cleanup

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
