# Certified Kubernetes Application Developer

## Introduction

These are my study notes for the Certified Kubernetes Application Developer exam Curriculum v1.15.0. The structure follows the exam curriculum but are not meant to be exhaustive. Always check the official documentation, tasks and tutorials for a topic.

This is the exam topics and weighting

| Topic                 | Exam weight |
|-----------------------|-------------|
| Core Concepts         |     13%     |
| Configuration         |     18%     |
| Multi-Container Pods  |     10%     |
| Observability         |     18%     |
| Pod Design            |     20%     |
| Services & Networking |     13%     |
| State Persistence     |      8%     |

## Useful links

* [A beginners guide to cron jobs](https://www.ostechnix.com/a-beginners-guide-to-cron-jobs/)

* [CKA vs CKAD](https://medium.com/faun/cka-vs-ckad-1dd45527505)
* [Study Guide by Matthew Palmer](https://medium.com/@_matthewpalmer/smash-your-kubernetes-ckad-exam-my-study-guide-and-exam-resources-58326755e2c3)
* [Practice exam by Matthew Palmer](https://matthewpalmer.net/kubernetes-app-developer/articles/ckad-practice-exam.html)
* [CKAD-excercies](https://github.com/dgkanatsios/CKAD-exercises)
* [k8s-labs](https://github.com/wiewioras/k8s-labs)
* [ckad-prep-notes](https://github.com/twajr/ckad-prep-notes)
* [Exam notes by Igor Izotov](https://medium.com/@iizotov/exam-notes-ckad-c1c4f9fb9e73)
* [CKAD exam tips](https://devblogs.microsoft.com/premier-developer/certified-kubernetes-application-developer-ckad-exam-tips/)
* [CKAD study notes](https://github.com/walidshaari/Kubernetes-Certified-Administrator/blob/master/README-ckad.md)
* [KodeKloud practice test](https://kodekloud.com/courses/kubernetes-certification-course/lectures/6743640)
* [Codeburst CKAD example questions](https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681)
* [Preparing for the CKAD by Matthias](https://dev.to/fullstack_to/preparing-for-kubernetes-application-developer-certification-ckad-54e0)
* [CKAD study guide by Evan Ng](https://www.cloudreach.com/en/insights/blog/study-guide-certified-kubernetes-application-developer-ckad-exam/)

## Table of Contents

1. [Core Concepts](#Core-Concepts)
2. [Configuration](#Configuration)
3. [Multi-Container Pods](#Multi-Container-Pods)
4. [Observability](#Observability)
5. [Pod Design](#Pod-Design)
6. [Services & Networking](#Services-&-Networking)
7. [State Persistence](#State-Persistence)

## Core Concepts

### Understand Kubernetes API primitives

Primitives are also called objects and they represent the state of the cluster.

Basic commands to view the objects:

```bash
kubectl api-resources
kubectl get <object name>
kubectl describe <object type> <object name>
```

All objects have a spec and status. The spec is the desired state of the object and the status is the current state of the object.

### Create and configure basic pods

Depending on the restart option the run command will deploy a pod in a few different ways;

Never = pod
OnFailure = job
Always = Deployment

```bash
kubectl run <podname> --image=<image name>

Common options
--rm # Deletes the resources once the pod has finished
--env # Adds environment variables
--port # Exposes the port
-r, --replicas # Deploys given number of replicas
--command # Runs a command inside the pod
```

Exec command allows us to run a command inside a running pod. Commonly used to get a shell inside the container to help troubleshoot an issue.

```bash
kubectl exec <pod name> -it -c <container name> -- bash
```

## Configuration

### Understand ConfigMaps

ConfigMaps are one of the ways Kubernetes allows us to decouple configuration data from the container image.
Configuration data can be consumed by pods via:

1. Populating environment variables
2. Setting command-line arguments for a container
3. Populate config files in a volume.

In the manifest the data field contains the configuration data in the form of key value pairs.

```yaml
apiVersion: v1
kind: ConfigMap
metatdata:
  name: myapp-settings
  namespace: myapp
data:
  dayofweek: tuesday
```

ConfigMaps can be created from directories and files.

Creating from directories and files both use the ```--from-file``` flag

```sh
kubectl create configmap app-settings --from-file=/settings/app-settings
```

Each file in the directory will create a key: value pair with the key being the filename.

```sh
kubectl create configmap calendar-data --from-file=random-data=app/settings/calendar-data
```

Use the ```from-literal``` flag to create a configMap from values on the command line.

```sh
kubectl create configmap calendar-data --from-literal=FIRSTDAY=Monday
```

Environment variables can be added under ```envFrom:``` and ```env:``` depending on which key:values we want to load.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: calender-data
  namespace: temp
data:
  FIRSTDAY: Monday
  FIRSTMONTH: January
---
#
# Load all key: values from the configmap
#
apiVersion: v1
kind: Pod
metadata:
  name: nginxtemp1
  namespace: temp
spec:
  containers:
    - name: nginxtemp
      image: nginx
      envFrom:
        - configMapRef:
            name: calender-data
---
#
# Load a single value from the configmap
#
apiVersion: v1
kind: Pod
metadata:
  name: nginxtemp2
  namespace: temp
spec:
  containers:
    - name: nginxtemp
      image: nginx
      env:
        - name: CAL
          valueFrom:
            configMapKeyRef:
              name: calender-data
              key: FIRSTDAY
```

The concept is similar when loading as volumes.

```yaml
# Load the configmap as a volume
apiVersion: v1
kind: Pod
metadata:
  name: nginxtemp2
  namespace: temp
spec:
  containers:
    - name: nginxtemp
      image: nginx
      volumeMounts:
        - name: cal-data
          mountPath: /cal/data
  volumes:
    - name: cal-data
      configMap:
        name: calender-data
---
# Load a single value from the configmap
apiVersion: v1
kind: Pod
metadata:
  name: nginxtemp2
  namespace: temp
spec:
  containers:
    - name: nginxtemp
      image: nginx
      volumeMounts:
        - name: cal-data
          mountPath: /cal/data
  volumes:
    - name: cal-data
      configMap:
        name: calender-data
        items:
        - key: FIRSTDAY
          path: NAMEOFDAY
```

ConfigMaps must be created before they are consumed unless they are marked as optional. References to ConfigMaps that do not exist will prevent the pod from starting.

### Understand SecurityContexts

Security contexts can be applied at either the pod or container level. If the context is set at a pod level it will apply to all containers.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-mover
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```

A value set at the container level will override any set at pod level.

Linux capabilities can be granted to a process without giving full root privileges.

```yaml
apiVersion: v1
kind: Pod
meatadata:
  name: data-mover
spec:
  containers:
  - name: busybox
    image:
    securityContext:
      capabilities:
        add: ["SETGID", "LEASE"]
```

SELinux labels can also be applied to a pod or container.

```yaml
apiVersion: v1
kind: Pod
meatadata:
  name: data-mover
spec:
  containers:
  - name: busybox
    image:
    securityContext:
      seLinuxOptions:
        level: "s0:c123,c456"
```

### Define an application's resource requirements

CPU and memory are collectively referred to as compute resources, or just resources. Compute resources are measurable quantities that can be requested, allocated, and consumed. Resources can specify a request and a limit. If a pod is scheduled the container is guaranteed the requested resources. If a pod exceeds its CPU limit it will be throttled.
If it exceeds its memory limit a process using the most memory will be killed by the kernel.

CPU is specified in units of cores such as 0.5CPU or the suffix m to mean milli. 100m, 100 milliCPU and 0.1CPU are the same.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: web
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "0.5"
```

Memory is specified in units of bytes; E, P, T, G, M, K and the power of 2 equivalent Ei, Pi, Ti, Gi, Mi, Ki.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: web
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

```sh
kubectl run nginx --restart=Never --image=nginx --requests="cpu=100m,memory=256Mi" --limits="cpu=200m,memory=512Mi"
```

### Create & consume Secrets

Secrets are for storing sensitive information such as passwords or tokens.
They can be mounted as files in a volume or as environment variables.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: madeupdata
stringData:
  key: thisandthat
```

```bash
kubectl create secret generic madeupdata --from-literal=key=thisandthat
```

```bash
cat >data.conf <<EOL
password:123
secret:abc
EOL

kubectl create secret generic testing --from-file=data.conf
```

```yaml
apiVersion: v1
Kind: Pod
metadata:
  name: apptest
spec:
  containers:
  - name: apptest
    image: nginx
    env:
      - name: APP_KEY
        valueFrom:
          secretKeyRef:
            name: madeupdata
            key: key
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: datatest
spec:
  containers:
  - name: datatest
    image: nginx
    volumeMounts:
    - name: appdata
      mountpath: "/etc/appdata"
  volumes:
  - name: appdata
    secret:
      secretName: testing
```

To view a secret

```bash
echo 'dGhpc2FuZHRoYXQ=' | base64 --decode
```

### Understand ServiceAccounts

When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).

When you create a pod, if you do not specify a service account, it is automatically assigned the default service account in the same namespace.

The service account has to exist at the time the pod is created, or it will be rejected.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: text-reader
---
apiVersion: v1
kind: Pod
metadata:
  name: reader-app
spec:
  containers:
  - image: busybox
    name: reader-app
  serviceAccountName: text-reader
```

## Multi-Container Pods

### Understand Multi-Container Pod design patterns

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecartest
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
  - name: busy
    image: busbox
    command: ['sh', '-c', 'while true; do sleep 30; done;']
```

3 common multi-container design patterns:

1. Sidecar: adds functionality like shipping logs or fetching files.
2. Ambassador: network traffic routes through the ambassador container before the main container.
3. Adapter: transforms the output from the main container

## Observability

### Understand LivenessProbes and ReadinessProbes

Kubelet uses liveness probes to know when to restart a Container and readiness probes to know when a Container is ready to start accepting traffic.

Three types of probes:

* Exec: Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
* TCP check: Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
* HTTP Get: Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: busybox
    readinessProbe:
      exec:  
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 60
      periodSeconds: 10
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: busybox
    ports:
    - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /live
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 60
      periodSeconds: 10

```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: <Image>
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 60
      periodSeconds: 10

```

### Understand container logging

```bash
kubectl logs <pod> <container> --previous
```

### Understand how to monitor applications

```bash
kubectl top pods

kubectl top nodes
```

### Understand debugging in Kubernetes

Nothing special here just understanding the commands available for seeing errors and logs.

```bash
# Describe gathers a host of information such as resources and events
kubectl describe pod nginx

# View the logs of a container/pod
kubectl logs webapp

# Get events for a namespace
kubectl get events -n webapp

# Run commands inside a container
kubectl exec -it webrunner -- sh

# Edit the current state of a resource
kubectl edit pod webrunner
```

## Pod Design

### Understand how to use Labels, Selectors, and Annotations

Labels contain identifying information and are used by selectors. Annotations contains non identifying information that can be used by external tools and libraries.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
inx  name: auth-app
  labels:
    app: auth
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
      - name: auth-app
        image: ng
```

```sh
admin@comp01:~$ kubectl get all --selector=app=auth
NAME                           READY   STATUS    RESTARTS   AGE
pod/auth-app-5f5cb6f99-cskz8   1/1     Running   0          2m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/auth-app   1/1     1            1           2m1s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/auth-app-5f5cb6f99   1         1         1       2m1s
```

Annotations are key/value pairs. Valid annotation keys have two segments: an optional prefix and name, separated by a slash (/).
The kubernetes.io/ and k8s.io/ prefixes are reserved for Kubernetes core components.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: jumper
  annotations:
    version: 1.2.3-stable
    owner: sre
    contact: 01234 657891 option 1
    prometheus.io/port: 10254
    prometheus.io/scrape: true
spec:
  containers:
    name: jumper
    image: busybox
```

### Understand Deployments and how to perform rolling updates

Deployments allow us to gradually update pods to minimize service interruptions. Under the deployment spec is the strategy field, this covers how existing pods are replaced with new ones.
You only need to specify these if you want settings that differ from the defaults which are RollingUpdate and 25% for both max surge and max unavailable.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
inx  name: auth-app
  labels:
    app: auth
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      # Absolute numbers are also accepted
      maxUnavailable: 2
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        # One of these labels should match the selector above
        app: auth
    spec:
      containers:
      - name: auth-app
        image: nginx
```

To update the image of a deployment

```sh
kubectl set image deployment auth-app auth-app=nginx:1.9.1 --record
```

The record flag adds the command as an annotation to the deployment and appears in the change-cause field in the rollout history.

To view the status of a rollout

```sh
kubectl rollout status deployment auth-app
```

### Understand Deployments and how to perform rollbacks

To undo the last rollout change

```sh
kubectl rollout undo deployment auth-app
```

The settings under the strategy spec applies to rollbacks as well as rollouts.

To rollback to a specific revision, add the revision number flag.

```sh
kubectl rollout history deployment auth-app

deployment.extensions/auth-app
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment auth-app auth-app=nginx:1.17 --record=true
```

```sh
kubectl rollout undo deployment auth-app --to-revision=1
```

### Understand Jobs and CronJobs

A job runs a container until its workload is complete. A CronJob is a job run on a schedule.

```sh
kubectl create job dateprint --image=busybox -- /bin/sh -c 'echo The date and time is;date'
```

```sh
kubectl create cronjob dateprint --image=busybox --schedule="* 1 * * *" -- /bin/sh -c 'echo The date and time is;date'
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: dateprint
spec:
  template:
    spec:
      containers:
      - name: dateprint
        image: busybox
        command:
        - /bin/sh
        - -c
        - echo The date and time is;date
      restartPolicy: Never
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: dateprint
spec:
  schedule: '* 1 * * *'
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: dateprint
            image: busybox
            command:
            - /bin/sh
            - -c
            - echo The date and time is;date
```

## Services & Networking

### Understand Services

A service object uses a selector to proxy traffic to a set of pods.

4 types of service objects:

* ClusterIp: Uses an cluster internal IP
* NodePort: Allocates a port on all nodes
* LoadBalancer: launches a cloud providers LoadBalancer
* ExternalName: Provide a CNAME pointing to a resource outside of Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      web: frontend
  template:
    metadata:
      labels:
        web: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: webapp
spec:
  selector:
    web: frontend
  type: NodePort
  ports:
  - name: frontend
    port: 8080
    targetPort: 80
```

Creating the service from the command line will set the selector key as *app* and the value as the service name.

```sh
kubectl create svc clusterip webapp-svc --tcp=8080:80
```

Ports are ordered as ```<service port>:<target port>```.

### Demonstrate basic understanding of NetworkPolicies

Network policies define how groups of pods are allowed to communicate with each other. By default pods accept traffic from any source.
Network policies only work if the installed networking solution supports them.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name:
  namespace:
spec:
  podSelector:
    matchLabels:
      app: auth
  policyTypes:
  - Ingress
  - Egress
  ingress:
 - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

## State Persistence

### Understand PersistentVolumeClaims for storage

Persistent Volumes can be created dynamically or statically by an Administrator. The storageClass must already exist for the dynamic provisioning to work.

Lots of storage options are supported by Kubernetes.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  storageClassName: fast
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  storageClassName: fast
  accessModes:
  - ReadWriteOnce
  requests:
    storage:
      capacity: 10Gi
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql
spec:
  containers:
  - name: db
    image: mysql
    volumeMounts:
    - name: mysql-db
      mountpath: /var/lib/mysql
  volumes:
  - name: mysql-db
    persistentVolumeClaim:
      claimName: mysql-pvc
```
