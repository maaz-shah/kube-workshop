# Kube Workshop


In this workshop we plan to install a Kubernetes cluster and to deploy an application and try out some `kubectl` commands. We will also learn how Deployments and Services look like in YAML and how to create ConfigMaps.

## Installing Kubernetes and the CLI

There are multiple tools available which can be used to create clusters in local environment in this Workshio we plan to use the tool called `minikube`. To install `minikube`, follow the instructions below:

1. For Mac run the following:

###  Intel Macs
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

### M1 / ARM Macs

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube

```
>Alternatively, use `brew install minikube` if using Homebrew.


>If `which minikube` fails after installation via brew, you may have to remove the old minikube links and link the newly installed binary:
```
brew unlink minikube
brew link minikube
```

### Linux
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

```

# Creating the cluster

```
minikube start
```
>If minikube fails to start, see the [drivers](https://minikube.sigs.k8s.io/docs/drivers/) page for help setting up a compatible container or virtual-machine manager and make sure that you have docker engine installed and running, install it from [here](https://www.docker.com/products/docker-desktop/) 

# Interact with your cluster
1. If you already have kubectl installed, you can now use it to access your shiny new cluster:
```
kubectl get po -A
```
Alternatively, minikube can download the appropriate version of kubectl and you should be able to use it like this:
```
minikube kubectl -- get po -A
```
You can also make your life easier by adding the following to your shell config:

```
alias kubectl="minikube kubectl --"
```
Initially, some services such as the storage-provisioner, may not yet be in a Running state. This is a normal condition during cluster bring-up, and will resolve itself momentarily. For additional insight into your cluster state, minikube bundles the Kubernetes Dashboard, allowing you to get easily acclimated to your new environment:

```
minikube dashboard
```

>If you want latest version of kubectl you can download it [here](https://kubernetes.io/releases/download/#kubectl)
### Other Tools for Kubernetes

Note that these are optional and not needed to go through the workshop. 

- [K9s - Kubernetes CLI To Manage Your Clusters In Style](https://github.com/derailed/k9s)
- [Kubectx](https://github.com/ahmetb/kubectx/)
- [Popeye - Kubernetes cluster resource sanitizer](https://github.com/derailed/popeye)
- [Lens - The Kubernetes Platform ](https://k8slens.dev/)
- [Play with Kubernetes - A simple, interactive and fun playground to learn Kubernetes ](https://labs.play-with-k8s.com/)
## Getting familiar

Kubernetes CLI (`kubectl`) is what you will use to talk to your Kubernetes cluster.

Let's make sure we have a cluster running:

```
$ kubectl get nodes
NAME             STATUS   ROLES    AGE    VERSION
minikube   Ready    control-plane   9m37s   v1.25.3
```

The `get` command is used for retrieving resources from the cluster. In the example above you used `get nodes` to get the information about all the nodes in the cluster.

## Creating Deploypments.

Create a sample deployment and expose it on port 80:

```
kubectl create deployment hello-world --image=docker.io/nginx:latest

```


With the command above we are creating a Kubernetes deployment called `hello-world` and telling Kubernetes to create a container from the provided image and run it on port `80`.

You can look at all the deployments in your cluster with the get command:

```
$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   0/1     1            0           22s
```

Let's also check the pods we have running for this deployment:

```
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-world-6dcd4c85cf-cjnhb   1/1     Running   0          34s
```

To get more information about the resources, you can use the `describe` command, like this:

```
$ kubectl describe deployment hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Tue, 15 Nov 2022 15:59:35 +0100
Labels:                 app=hello-world
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-world
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-world
  Containers:
   nginx:
    Image:        docker.io/nginx:1.23
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-6dcd4c85cf (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  63s   deployment-controller  Scaled up replica set hello-world-6dcd4c85cf to 1


```

Above command gives you more details about the desired resource instance.

## Accessing services

Any containers running inside Kubernetes need to be explicitly exposed in order to be able to access them from the outside of the cluster.

You can use the `expose` command to create a Kubernetes service and 'expose' your application. Note that since you are using `kind` you will have to use `port-forward` command to proxy the calls from your local computer to the service and pods running inside the cluster.

Expose the deployment first (this creates a Kubernetes service):
```
kubectl expose deployment hello-world --port=80 --target-port=80 --type=LoadBalancer
```

The output of the above command will simply be: `service/hello-world exposed`.

The above command exposes the deployment called `hello-world` on port `80`, talking to the target port (container port) `80`. Additionally, we are saying we want to expose this on a service of type LoadBalancer - this will allocate a 'public' IP for us (`localhost` when running Docker for Mac and an actual IP address if using a cloud-managed cluster).
Minikube has a native function to make the service available to be accessed via local web browser you can run
```
minikube service hello-world

```
Using Kubectl we can also do it by command below (open in a seperate terminal)
```
kubectl port-forward --address localhost service/hello-world 80:80
```
Now you can acccess the application on e.g. `http://localhost`.


Expose command creates another Kubernetes resource - a Service. To look at the details of the created service, run:

```
$ kubectl describe service hello-world
Name:                     hello-world
Namespace:                default
Labels:                   app=hello-world
Annotations:              <none>
Selector:                 app=hello-world
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.111.169.15
IPs:                      10.111.169.15
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32432/TCP
Endpoints:                172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

```

## Scalling up/down

Now that you can access the application through exposed service, you can try and scale the deployment and create more replicas of the application:

```
kubectl scale deployment hello-world --replicas=5
```

If you look at the pods now, you should see 5 different instances:

```
$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-world-5b445dc7b8-4bf55   1/1     Running   0          8s
hello-world-5b445dc7b8-8z8hw   1/1     Running   0          8s
hello-world-5b445dc7b8-pcqgv   1/1     Running   0          9s
hello-world-5b445dc7b8-sbpqb   1/1     Running   0          6m1s
hello-world-5b445dc7b8-t4g7c   1/1     Running   0          8s
```

Similarly, you can scale down the pods by running the same command and providing a smaller number of replicas:

```
kubectl scale deployment helloworld --replicas=1
```

## Kubernetes Dashboard

If you are using Minikube the dashboard is already installed - just run `minikube dashboard` to open it.

If you're using Docker for Mac/Windows, follow the steps below to install the dashboard.

1. Install the Kubernetes dashboard:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml

```

1. List all pods running in the cluster:

```
kubectl get pods --all-namespaces
```

> Note: instead of using `--all-namespaces`, you can also use the short version of the flag `-A`

1. Look for the pod named `kubernetes-dashboard-XXXXX` and ensure that it's up and running (check the `STATUS` column). This is the dashboard you installed in the previous part.

1. Open another terminal window and create a proxy to the cluster:

```
kubectl proxy
```

1. Try to open the dashboard:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#
```

1. You will notice that you need to login first. To be able to log in, you need to create a new user that has permissions to log into the dashboard.

1. Deploy the `admin.yaml` file - this will create the account and a binding:

```bash
$ kubectl apply -f admin.yaml

serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

1. With this deployed, you need a bearer token that you will use to log in to the dashboard. Run the command below to create the token for the user you created. We will use this token to login.

```
kubectl -n kubernetes-dashboard create token admin-user
```

1. Copy the token value, open the dashboard again, select the Token option. Paste in the token from the previous step and click Sign in.

1. Familiarize yourself with the dashboard - check all the namespaces, deployments and pods that are running in the cluster.

## Config maps and Secrets

To demonstrate config maps and secrets in Kubernetes, you will use a sample Node.js application from the `./hello-kube` folder.

Let's make some changes to the application, build the Docker image and then deploy it to the cluster.

1. Open `routes/index.js` and change the `firstName` variable value to your name.

1. Build the Docker image:

   ```
   docker build -t hello-kube:0.1.0 .
   ```

1. Run the Docker image on your machine to make sure changes work:

   ```
   docker run -it --rm -p 8080:3000 hello-kube:0.1.0
   ```

1. Tag the image to the Docker hub repo:

   ```
   docker tag hello-kube:0.1.0 REPONAME/hello-kube:0.1.0
   ```

1. Push the image to the Docker hub:

   ```
   docker push REPONAME/hello-kube:0.1.0
   ```

1. Update `deploy/deployment.yaml` to use the image name you pushed.

1. Using `kubectl` create the Kubernetes deployment and service:

   ```
   kubectl apply -f deploy/deployment.yaml
   kubectl apply -f deploy/service.yaml
   ```

1. Open http://localhost to see the application running inside the Kubernetes cluster.

### Using config maps

Instead of hardcoding the first name in the code, we want to read the first name value from a config map called `hello-kube-config`.

1. Create a Kubernetes config map resource to store the `firstName` value. The quickest way to create a config map is using `kubectl`

   ```
   kubectl create configmap hello-kube-config --from-literal=firstName=Max
   ```

   To see the YAML file that was created, use the following command:

   ```
   $ kubectl get cm hello-kube-config -o yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
       name: hello-kube-config
       namespace: default
   data:
       firstName: Max
   ```

1. Mount the config map and the value as an environment variable in the deployment file. Make copy of the existing `deployment.yaml` file (you can name it `deployment-configmaps.yaml` and update it to mount the config map and it's values under the `env` key, like this:

   ```yaml
   ...
   containers:
       - name: web
       ...
         env:
           - name: FIRST_NAME_VARIABLE
             valueFrom:
               configMapKeyRef:
                   name: hello-kube-config
                   key: firstName
       ...
   ```

> Note: make sure you indent the YAML correctly, otherwise you will get errors deploying the YAML.

1. Modify the code to read the `firstName` from an environment variable.

   > Note: to read environment values in NodeJs, you can write `const firstName = process.env.MY_VARIABLE_NAME`

1. Rebuild the Docker image using tag `0.1.1` and push it to the Docker registry.
1. Apply the updated `deployment.yaml` file (one with environment variable from the config map and updated image name). You can use `kubectl apply -f deploy/[deployment.yaml]`.

### Using secrets

The application reads all file names from the `app/my-secrets` folder and displays them on the web page. Let's create a Secret with a couple of files and mount it as a volume in the deployment.

1. Create a couple of files (the contents are not really important):

   ```
   echo "Hello World" >> hello.txt
   echo "secretpassword" >> passwd
   echo "some=config" >> config.env
   ```

1. Create a Secret using `kubectl` command:

   ```
   kubectl create secret generic hello-kube-secret --from-file=./hello.txt --from-file=./passwd --from-file=config.env
   ```

1. You can use the `describe` command to look at the secret details:

   ```
   kubectl describe secret hello-kube-secret
   ```

1. Update your deployment file (you can either create a copy of the previous file or update the existing one) and mount the created secret into the pod:

   ```yaml
   ...
   spec:
       containers:
         - name: web
           ...
           volumeMounts:
             - name: my-secret-volume
               mountPath: "/app/my-secrets"
               readOnly: true
       volumes:
         - name: my-secret-volume
           secret:
             secretName: hello-kube-secret
       ...
   ```

1. Apply the updated deployment YAML file. Note that we didn't have to rebuild the Docker image as there were no changes to the code.

## Kubernetes Health checks: liveness, readiness and startup probes

Application has the following three endpoints implemented that can be used to learn how health and readiness probes work.

### Endpoints

#### `/healthy`

Returns an HTTP 200. If `HEALTHY_SLEEP` environment variable is set, it will sleep for the amount of millisecond defined in the variable. For example if `HEALTHY_SLEEP=5000` the endpoint will wait for 5 seconds before returning HTTP 200.

#### `/healthz`

The endpoint returns HTTP 200 for the first 10 seconds. After that it starts returning an HTTP 500.

#### `/ready`

Functionally equivalent to the `/healthy` endpoint (returns HTTP 200). Uses `READY_SLEEP` environment variable to sleep for the amount of millisecond defined in the variable.

#### `/readyz`

The endpoint returns HTTP 500 for the first 10 seconds. After that it starts returning an HTTP 200.

#### `/fail`

Returns an HTTP 500 error. Uses `FAIL_SLEEP` environment variable to sleep for the amount of miliseconds defined in the variable before returning.

### Liveness probe

1. Make a copy of the `deployment.yaml` file or use the one from the previous exercise.
1. Add the following snippet to the container spec:

   ```yaml
   ---
   - name: web
       ...
     livenessProbe:
       httpGet:
         path: /healthz
         port: 3000
       initialDelaySeconds: 3
       periodSeconds: 3
   ```

1. Deploy the YAML file.

   ```
   kubectl apply -f deploy/deployment.yaml
   ```

1. Use the describe command to observe how the pod gets restarted after the probe starts to fail.

   ```
   # Get the pod name first
   kubectl get pods

   kubectl describe pod [pod_name]
   ```

The first health check happens 3 seconds after the container starts (`initialDelaySeconds`) and it's repeated every 3 seconds (`periodSeconds`). As soon as the health check fails, container gets restarted and the whole process repeats.

### Readiness probe

Readiness has very similar format as the liveness probe, the only difference is the field name (`readinessProbe` instead of `livenessProbe`). You would use a readiness probe when your service might need more time to start up, yet you don't want the liveness probe to kick in and restart the container. If the readiness probe determines that the service is not ready, no traffic will be sent to it.

Let's use the `/readyz` endpoint - this endpoint returns HTTP 500 for the first 10 seconds and then starts returning HTTP 200.

1. Edit the `deployment.yaml` and add the `readinessProbe` as well as the environment variable:

   ```yaml
   ...
   - name: web
     ...
     readinessProbe:
       httpGet:
         path: /readyz
         port: 3000
       initialDelaySeconds: 3
       periodSeconds: 3
   ```

1. Deploy the YAML file:

   ```
   kubectl apply -f deployment.yaml
   ```

1. Try making requests to the `http://localhost`. You will notice that you won't be able to get any responses through, and after 10 second the pod will be ready and you will be able to make responses.

Another difference between the readiness probe when compared to the liveness probe is that if readiness probe fails, it won't restart the container.

## Resource quotas

With resource quotas you can limit the resource consumption per every Kubernetes namespace. For example, you can limit the quantity of resource types that can be created in a namespace as well as by total amount of resources that are consumed (e.g. CPU, memory).

Using a `ResourceQuota` resource we can set the limits. If users try to create or update resources in a way that violates this quota, the request will fail with HTTP 403 (forbidden).

1. Deploy a resource quota that limits the number of pods to 5:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pods-limit
spec:
  hard:
    pods: "5"
EOF
```

1. Deploy the `hello-kube` deployment (if not already deployed) and try to scale it to 10:

   ```
   kubectl scale deploy hello-kube-deployment --replicas=10
   ```

1. Get the list of pods and you will notice only 5 replicas are created, meaning we are hitting the limit.

   ```
   kubectl get pods
   ```

1. Let's try to create another, one off pod using image `radial/busyboxplus:curl`:

   ```
   kubectl run extra-pod --image=radial/busyboxplus:curl --generator=run-pod/v1
   ```

1. You will notice that this time, we get an actual error:

   ```
   Error from server (Forbidden): pods "extra-pod" is forbidden: exceeded quota: pods-limit, requested: pods=1, used: pods=5, limited: pods=5
   ```

Make sure you don't forget to delete the quota before moving on:

```
kubectl delete quota pods-limit
```


## Clean up

Before continuing, making sure you delete the deployments, services and other resources you created as part of this exercise:

```
kubectl delete deployment helloworld
kubectl delete service helloworld

kubectl delete deployment hello-kube
kubectl delete service hello-kube
```