# Deploying Kubernetes Applications with Kind

In this execrise, we will practice deploying applications into Kubernetes. Kubernetes is a popular orchestration platform that is used by many organizations to deploy applications. Kubernetes enables developers to worry less about the infrastructure the applciation is running on and focus more on the things that help the application perform. Do you remember the role a hypervisor plays for it's VM'? That is similar to what Kubernetes is to apps it hosts.

Here is a high level view of what we will be doing in this lab:
- Installing and configuring pre-requisite software and workstation settings
- Reviewing some handy setup tips
- Creating a new local kubernetes cluster using 'kind'
- Deploying a pre-made sample application

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
2. The first thing we need to do is make sure you CLI utilities are installed correctly. To do this execute the commands below and confirm your output is similar and that you are not recieving any error messages.

go command and expected output (it's ok if your version is different)
```
go version
```
```
C:\****\****>go version
go version go1.24.0 windows/amd64
```
ubectl command and expected output
```
kubectl version
```
```
C:\****\****>kubectl version
Client Version: v1.31.4
Kustomize Version: v5.4.2
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
````
Note: If you have just setup kubectl and have not connected to any clusters, the "unable to connect to the server" error will be displayed. Don't worry, this is ok! We are only validating that kubectl is installed correctly. The kind installation will update our kube config file for us automatically.  

3. Next we need to prepare the go environment by downloading the dependencies needed to create your cluster. To do this, run the command below
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

4.  Next up, we will execute the command to have kind create our cluster for us. The command below will kick off the cluster installation process.
```
kind create cluster
```
Expected ouput
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

4.  We now have a kubernetes cluster installed but, before we continue, let's make sure everything is working properly. Let's query the cluster nodes and see if our Control Plane has started.
```
kubectl get nodes
```
If everything completed sucessfully, you should get a response similar to the below output. The key here is ensuring the Control Plane node status is ready. Depending on your systems resources, this may take a few minutes. Periodically execute the command until all is well.
```
C:\****\****>kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   28s   v1.32.2
```
To further confirm cluster health, you can execute the command below.
```
kubectl cluster-info --context kind-kind
```
Your output should look similar to the below.
```
C:\Use\****>kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:51397
CoreDNS is running at https://127.0.0.1:51397/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## Preparing Your New Cluster
Congratulations! At this point you have successfully completed installation/configuration of pre-requisite software and built your local kubernetes cluster.

In this section we will be reviewing the configurations your application needs to operate and be reachable on your network. We will be using a sample application from Google repositories to keep things simple.

### Create a New Namespace
It's best practice to avoid deploying apps in the default namespace in your cluster. If you use default, you may encounter issues in the future where a cluster version upgrade modifies settings you have applied and creates outages or performance impacts on your apps. 

An example of an issue you could run into is changes to the default namespace limits or quotas:
- Cluster upgrade removes the limits you set on the default namespace. If you are using a cloud K8s Cluster with node scaling, your app could endlessly scale up and surprise you with a very expensive bill.

1. To create a new namespace named 'mylab', execute the command below.
```
kubectl create ns mylab
```
Your output should be similar to the below.
```
C:\*****\****>kubectl create ns mylab
namespace/mylab created
```
2. Let's check to make sure our namespace is created by executing the command below.
```
kubectl get ns
```

You should be able to see all the namespaces that exist on your cluster. Note the age for the namespace we created, "mylab" and that it's status is "active"
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


## At Last, Deploying Your App!
We are now ready to deploy your application! 

In this section, we will discuss the following:
- Deploying a Kubernetes Service
- Deploying a Kubernetes.... Deployment :)
- Configuring Networking for App Access
- Other Common Configurations

### Deploying a Kubernetes... Deployment :)
The kubernetes deployment resource that defines the state of how your application will run in the environment. It is where we define things like how many replicas should be running, what image will be used for the containers, and overall management of the pods running your application. For more information on Kubernetes deployments, visit [Official Kubernetes Deployment Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Let's deploy!

1. Load your code editor of choice and create an empty yaml file. Let's name is testAppDeploy.yaml
2. Copy and paste the yaml below into your editor. Note the comments in the file. They will provide a brief explanation of what the configs are doing.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web                                           # Places a label on all pods for easy identification of your app.
  name: web                                            # defines the name of the deployment. For the sake of sanity, it's best to name it the same things as your app label
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
      - image: gcr.io/google-samples/hello-app:1.0     # the location of the image you want to use for your app. This can be a secure private repo, public, etc. Just make sure your network allows access to the repo.
        name: hello-app                                # the name of the application being ran.
```

3. Lets get your app deployed. Remember when we talked about not deploying apps to the default namepsace? This is where that matters. Any configuration you apply, command you run, etc will target the default namespace unless you either explicitly define the namespace in your command or you set the namespace context in your kube config.

Setting the namespace context will change the default namespace to whatever value you define. This can be handy but make sure you always know what your default is. It may be less risky to get in the habit of explicitly defining your target namespace. If you want to set your namespace context, run the below command.
```
kubectl config set-context --current --namespace=mylab
```
Your result should look similar to the output below.
```
C:\****\****>kubectl config set-context --current --namespace=mylab
Context "kind-kind" modified.
```

4. Deploy your app! Kubernetes views its configurations, including app deployments, as a definition of state. As such, the commands it uses to do the work reflect that. In this case, deploying your application is actually applying a config state for a deployment resource. Make sure your CLI PWD (present working directory) is the folder that contains the testAppDeploy.yaml file. To apply your configuration, run the command below.

Note: if you opted to change your context, you do not need to include the namespace "-n mylab"
```
kubectl apply -f testAppDeploy.yaml -n mylab
```

Your result should look similar to the ouput below.
```
C:\Users\seph\Documents\Spectro>kubectl apply -f testAppDeploy.yaml -n mylab
deployment.apps/web created
```

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
Notice the pod has the name you defined in line 6 of your testAppDeploy.yaml file. The rest of the name is randomly assigned to prevent issues with duplicate pods running as your application scales up.

### Deploying a Kubernetes Service
The Kubernetes service resource plays a crucial role for applications in your clusters. We will briefly go over the the purpose of the service is and why it's needed for every application you deploy.

Ingress access to your application - The service resource can be configured to define how your application will be accessed. Think of this as connecting a network cable to your application and controlling where the other end of the connection can terminate. In this case, we are stating "nodeport" as the connection method. When the "nodeport" value is set, Kubernetes will direct all traffic into your service by mapping a defined port number on the node to the port of your application (port forwarding). There will be examples of this in the "Deploy Your Service Section".

Internal cluster routing to your pods - When you deploy your application into Kubernetes, it runs in a resource called a pod. The number of pods that run is controlled in your deployment file and can be set to 1 pod 22 pods, 100 pds, etc. These pods can start and stop many times a day resulting in frequent internal IP changes for your workloads. The service resource is tied to your deployment and keeps track of how many pods are running and the information it needs to send traffic to them. View the kubernetes service resource as more of a dynamic load balancer that is always aware of where your application can be reached.

NOTE: There are other service configurations available for you to use if you need them. For more information visit [Official Kuberentes Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

Let's get your service deployed!

1. Load your code editor of choice and create an empty yaml file. Let's name is testAppService.yaml
3. Copy and paste the yaml below into our testAppService.yaml file and save the changes.
```
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
5. 




```shell
$ PODNAME=$(kubectl get pods -n mylab --template '{{range .items}}{{.metadata.name}}{{end}}' --selector=app=web)
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


