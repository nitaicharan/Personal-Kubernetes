<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
# Running your first helloworld

## Chapter Goals

1. Starting up minikube
2. Set up your first Kubernetes helloworld application
3. Run your application in a minikube environment
4. Expose application via a nodeport and see it running

### Starting up minikube

First, get minikube up and running with the command `minikube start`. This command sets up a Kubernetes dev environment for you via VirtualBox.

The last statement in the output states that kubectl can talk to minikube. We can verify this by running the command `kubectl get nodes`

This will show you that minikube is ready to use.

### Set up your helloworld

Make sure you have your files unzipped to your local machine (for example /documents/Kubernetes). You should be in your existing directory with the exercise files for chapter 03_04 as shown below.

```
$ pwd
/Users/kgaekwad/Documents/Kubernetes/03_04
$ ls -al
total 16
drwxr-xr-x   4 kgaekwad  staff   128 Apr 14 03:41 .
drwxr-xr-x  22 kgaekwad  staff   704 Apr 14 03:25 ..
-rw-r--r--   1 kgaekwad  staff  3035 Apr 14 03:55 Readme.md
-rw-r--r--@  1 kgaekwad  staff   448 Apr 14 08:01 helloworld.yaml
```

We will run one of the most common Docker helloworld applications out there- [https://hub.docker.com/r/karthequian/helloworld/]

To run this, type:

```
kubectl create -f helloworld.yaml
```

This command creates a deployment resource from the file helloworld.yaml, which, in this case, contains a deployment called "hellworld", pulling from the image karthequian/helloworld, and exposes port 80 of the container to the pod.

Running this command will give you this output, stating that the deployment "hw" was created.

```
$ kubectl create -f helloworld.yaml 
deployment.apps/helloworld created
```

We can run the command `kubectl get all` to see all our resources running, as shown in the output below.

```
$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/helloworld-7cf6df685c-kpnjv   1/1     Running   0          99s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        53m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/helloworld   1/1     1            1           99s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/helloworld-7cf6df685c   1         1         1       99s
$ 

```

### Setting up a load balancer
You'll notice that in the `kubectl get all` command, the service has a port mapping defined; however, when you try to hit that port via your web browser, you won't be able to reach the service.

This is because by default, the pod is only accessible by its internal IP address within the cluster. To make the helloworld container accessible from outside the Kubernetes virtual network, you have to expose the pod as a Kubernetes service.

To do this, we can expose the pod to the public internet using the kubectl expose command 
`kubectl expose deployment helloworld --type=NodePort`

The `--type=NodePort` flag exposes the deployment outside of the cluster. If you're using this on a cloud provider, you can use a `--type=LoadBalancer` that will provision an external IP address would be provisioned to access the service.

To view the final user interface, use the minikube service command.

`minikube service helloworld`

This will open your web browser to your application that is running in Kubernetes!


#### Commands run in this section
```
pwd
ls -al
kubectl get all
kubectl create -f helloworld.yaml
kubectl expose deployment helloworld --type=NodePort
minikube service helloworld
```
=======
# Adding labels to the application
=======
# Application health checks
>>>>>>> 07f826f (Application health checks)
=======
# Handling application upgrades
>>>>>>> ae1865c (Handling application upgrades)

## Chapter Goals
1. Upgrade a deployment from 1 version to another
2. Rollback the deployment to the 1st version

### Upgrade a deployment from 1 version to another

Let's deploy our initial version of our application `kubectl create -f helloworld-black.yaml --record`. After the deployment and service has completed, let's expose this via a nodeport by `minikube service navbar-service`. This will bring up the helloworld UI that we have seen before. We added the `--record` to this because we want to record our rollout history that I'll talk about later on.

Looking at the deployment, we see that there are 3 desired, current and ready replicas.

As developers, we're required to make changes to our applications and get these deployed. The rollout functionality of Kubernetes assists with upgrades because it allows us to upgrade the code without any downtime. In our application, I'd like to update the nav bar to a blue color rather than what it is right now. I'll package this in a container with a new label called "blue".

To update the image, I run this command: `kubectl set image deployment/navbar-deployment helloworld=karthequian/helloworld:blue`. This sets the image from what it is currently (karthequian/helloworld:black) to karthequian/helloworld:blue.

The deployment will update, and if you look at the webpage, you'll see the updated text.

Let's take a look at what happened here. When the deployment was edited, a new result set was created for it. Running the `kubectl get rs` command shows us this. One result set with 3 desired, current and ready pods, and another with 0.

We can also take a look at the rollout history by typing `kubectl rollout history deployment/navbar-deployment`.

### Rollback the deployment to the 1st version
To rollback the deployment, we use the rollout undo command `kubectl rollout undo deployment/navbar-deployment`. This will revert our changes back to the previous version.

Our webpage will be back to the Lionel version of the deployment.

<<<<<<< HEAD
```

We run this yaml the same as we had done before: `kubectl create -f helloworld-deployment-with-probes`


### Simulate a failing deployment that fails a readiness probe
We will now try to simulate a bad helloworld pod that fails a readiness probe. Instead of checking port 80 like the last example, we will run a readiness check on port 90 to simulate a failing scenario. Thus, our yaml now is:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment-with-bad-readiness-probe
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
        readinessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # Amount of time to wait before timing out
          initialDelaySeconds: 1
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 90
```

We will run this yaml with the command `kubectl create -f helloworld-with-bad-readiness-probe.yaml`. After about a minute, we will notice that our pod is still not in a read state when we run the `kubectl get pods` command.

```
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm   0/1       Running   0          1m
```

Describing the pod will show that the pod failed readiness:
```
MacbookHome:04_02 Application health checks karthik$ kubectl describe po helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm
Name:   helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm
Namespace:  default
Node:   minikube/192.168.99.101
Start Time: Sun, 29 Oct 2017 01:35:08 -0500
Labels:   app=helloworld
    pod-template-hash=4220863004
Annotations:  kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"helloworld-deployment-with-bad-readiness-probe-8664db7448","uid"...
Status:   Running
IP:   172.17.0.4
Controllers:  ReplicaSet/helloworld-deployment-with-bad-readiness-probe-8664db7448
Containers:
  helloworld:
    Container ID: docker://61fa8b27fe2e8239d5dc84fc097af17c9d4decc90b3bea09103cc9d73302f235
    Image:    karthequian/helloworld:latest
    Image ID:   docker-pullable://karthequian/helloworld@sha256:165f87263f1775f0bf91022b48a51265357ba9f36bc3882f5ecdefc7f8ef8f6d
    Port:   80/TCP
    State:    Running
      Started:    Sun, 29 Oct 2017 01:35:10 -0500
    Ready:    False
    Restart Count:  0
    Readiness:    http-get http://:90/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-x8qf6 (ro)
Conditions:
  Type    Status
  Initialized   True
  Ready   False
  PodScheduled  True
Volumes:
  default-token-x8qf6:
    Type: Secret (a volume populated by a Secret)
    SecretName: default-token-x8qf6
    Optional: false
QoS Class:  BestEffort
Node-Selectors: <none>
Tolerations:  <none>
Events:
  FirstSeen LastSeen  Count From      SubObjectPath     Type    Reason      Message
  --------- --------  ----- ----      -------------     --------  ------      -------
  5m    5m    1 default-scheduler         Normal    Scheduled   Successfully assigned helloworld-deployment-with-bad-readiness-probe-8664db7448-pv4fm to minikube
  5m    5m    1 kubelet, minikube         Normal    SuccessfulMountVolume MountVolume.SetUp succeeded for volume "default-token-x8qf6"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Pulling     pulling image "karthequian/helloworld:latest"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Pulled      Successfully pulled image "karthequian/helloworld:latest"
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Created     Created container
  5m    5m    1 kubelet, minikube spec.containers{helloworld} Normal    Started     Started container
  4m    1m    20  kubelet, minikube spec.containers{helloworld} Warning   Unhealthy   Readiness probe failed: Get http://172.17.0.4:90/: dial tcp 172.17.0.4:90: getsockopt: connection refused
MacbookHome:04_02 Application health checks karthik$
```

### Simulate a failing deployment that fails a liveness probe

Next, we will simulate a bad helloworld pod that fails a liveness probe. Instead of checking port 80 like the last example, we will run a liveness check on port 90 to simulate a failing scenario. Thus, our yaml now is:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment-with-bad-liveness-probe
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: helloworld
        image: karthequian/helloworld:latest
        ports:
        - containerPort: 80
        livenessProbe:
          # length of time to wait for a pod to initialize
          # after pod startup, before applying health checking
          initialDelaySeconds: 10
          # How often (in seconds) to perform the probe.
          periodSeconds: 5
          # Amount of time to wait before timing out
          timeoutSeconds: 1
          # Kubernetes will try failureThreshold times before giving up and restarting the Pod
          failureThreshold: 2
          # Probe for http
          httpGet:
            # Path to probe
            path: /
            # Port to probe
            port: 90
```


We will run this yaml with the command `kubectl create -f helloworld-with-bad-liveness-probe.yaml`. After about a minute, we will notice that our pod is still not in a read state when we run the `kubectl get pods` command.

```
MacbookHome:04_02 Application health checks karthik$ kubectl create -f helloworld-with-bad-liveness-probe.yaml
deployment "helloworld-deployment-with-bad-liveness-probe" created
MacbookHome:04_02 Application health checks karthik$ kubectl get deployments
NAME                                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment-with-bad-liveness-probe   1         1         1            1           5s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS        RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4    1/1       Running       0          9s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                              READY     STATUS        RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4    1/1       Running       1          26s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   1          32s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   2          38s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS    RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   1/1       Running   3          50s
MacbookHome:04_02 Application health checks karthik$ kubectl get pods
NAME                                                             READY     STATUS             RESTARTS   AGE
helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4   0/1       CrashLoopBackOff   3          1m
```

When we describe our pod, we notice that it is failing the liveness probe:

```
MacbookHome:04_02 Application health checks karthik$ kubectl describe po helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4
Name:   helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4
Namespace:  default
Node:   minikube/192.168.99.101
Start Time: Sun, 29 Oct 2017 01:47:59 -0500
Labels:   app=helloworld
    pod-template-hash=3331596669
Annotations:  kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"helloworld-deployment-with-bad-liveness-probe-77759fbbbf","uid":...
Status:   Running
IP:   172.17.0.4
Controllers:  ReplicaSet/helloworld-deployment-with-bad-liveness-probe-77759fbbbf
Containers:
  helloworld:
    Container ID: docker://41cfe81a3327b03831440e861eb2164b8a9f601ce630f024c76412d903781922
    Image:    karthequian/helloworld:latest
    Image ID:   docker-pullable://karthequian/helloworld@sha256:165f87263f1775f0bf91022b48a51265357ba9f36bc3882f5ecdefc7f8ef8f6d
    Port:   80/TCP
    State:    Running
      Started:    Sun, 29 Oct 2017 01:48:47 -0500
    Last State:   Terminated
      Reason:   Completed
      Exit Code:  0
      Started:    Sun, 29 Oct 2017 01:48:31 -0500
      Finished:   Sun, 29 Oct 2017 01:48:46 -0500
    Ready:    True
    Restart Count:  3
    Liveness:   http-get http://:90/ delay=10s timeout=1s period=5s #success=1 #failure=2
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-x8qf6 (ro)
Conditions:
  Type    Status
  Initialized   True
  Ready   True
  PodScheduled  True
Volumes:
  default-token-x8qf6:
    Type: Secret (a volume populated by a Secret)
    SecretName: default-token-x8qf6
    Optional: false
QoS Class:  BestEffort
Node-Selectors: <none>
Tolerations:  <none>
Events:
  FirstSeen LastSeen  Count From      SubObjectPath     Type    Reason      Message
  --------- --------  ----- ----      -------------     --------  ------      -------
  58s   58s   1 default-scheduler         Normal    Scheduled   Successfully assigned helloworld-deployment-with-bad-liveness-probe-77759fbbbf-kljs4 to minikube
  58s   58s   1 kubelet, minikube         Normal    SuccessfulMountVolume MountVolume.SetUp succeeded for volume "default-token-x8qf6"
  47s   12s   4 kubelet, minikube spec.containers{helloworld} Warning   Unhealthy   Liveness probe failed: Get http://172.17.0.4:90/: dial tcp 172.17.0.4:90: getsockopt: connection refused
  58s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Pulling     pulling image "karthequian/helloworld:latest"
  57s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Pulled      Successfully pulled image "karthequian/helloworld:latest"
  57s   11s   4 kubelet, minikube spec.containers{helloworld} Normal    Created     Created container
  41s   11s   3 kubelet, minikube spec.containers{helloworld} Normal    Killing     Killing container with id docker://helloworld:Container failed liveness probe.. Container will be killed and recreated.
  57s   10s   4 kubelet, minikube spec.containers{helloworld} Normal    Started     Started container
MacbookHome:04_02 Application health checks karthik$
```

And finally, when we check the deployment, we notice that deployment is not available as well.
```
MacbookHome:04_02 Application health checks karthik$ kubectl get deployments
NAME                                            DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloworld-deployment-with-bad-liveness-probe   1         1         1            0           3m
```

<<<<<<< HEAD
### Extending the label concept to deployments/services

As a bonus, labels will also work with `kubectl get services/deployments/all --show-labels` and will return labels for your services, deployments or all objects.

`kubectl get all --show-labels`

And, you can delete pods, services or deployments by label as well! For example `kubectl delete pods -l dev-lead=karthik` will delete all pods who's dev-lead was Karthik. 


To summarize, labels in Kubernetes is a powerful concept! Use the labeling feature to your advantage to build your infrastructure in an organized fashion.
>>>>>>> 985fe68 (Working with labels)
=======
To summarize, we've learned that we can use readiness and liveness probes to check the status of our pods and use them to restart pods when necessary and check pod health.
>>>>>>> 07f826f (Application health checks)
=======
In a real world setting, you might have a longer history, and might want to rollback to a specific version. To do this, add a `--to-revision=version` to the specific version you want to.
>>>>>>> ae1865c (Handling application upgrades)
