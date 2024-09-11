# Creation of Multi Container Pod Init Container
## MyApp Pod with Init Containers
This project demonstrates how to create a Kubernetes Pod with init containers to ensure that certain services are available before the main application container starts.

### Overview
The Pod consists of:
- A main application container (myapp-container) that runs a simple busybox image.
- Two init containers (init-myservice and init-mydb) that ensure the dependent services are available via DNS  lookup before starting the main container.

### Key Components
 - Init Containers:
     - init-myservice: Ensures the service myservice is reachable before starting the main container.
     - init-mydb: Ensures the service mydb is reachable before starting the main container.

 -   Main Container:
     - myapp-container: Runs after the init containers have completed their tasks, and it prints a message before sleeping for 1 hour.

### Prerequisites
 - A running Kubernetes cluster (e.g., kind, minikube, or a cloud-managed Kubernetes service).
 - kubectl configured to communicate with your cluster.

## Tasks

 ### 1. Create Services (myservice and mydb)
 Before creating the Pod, we need to set up the two services that the init containers will wait for:
 
 1. Create the myservice service:
    ```
    kubectl expose deployment myservice --name=myservice --port=80 --target-port=8080
    ```
    
 2. Create the mydb service:
    ```
    kubectl expose deployment mydb --name=mydb --port=80 --target-port=3306
    ```
    
 These services should now be resolvable by DNS within the cluster.

 ###  2. Create the Pod
 Create the myapp-pod using the YAML file below. This Pod has two init containers to ensure the services are available before starting the main container.

 ```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: FIRSTNAME
      value: "Piyush"
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup myservice.default.svc.cluster.local; do echo waiting for myservice; sleep 2; done']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c']
    args: ['until nslookup mydb.default.svc.cluster.local; do echo waiting for mydb; sleep 2; done']
 ```
 Save the YAML file as myapp-pod.yaml, and apply it using kubectl:
 ```
 kubectl apply -f myapp-pod.yaml
 ```
 ### 3. Verify the Pod
 After applying the YAML, verify that the init containers are waiting for the services and that the main container starts successfully after they complete:
 ```
 kubectl get pods
 ```
 To get more details on the pod's status and logs from the init containers, use:
 ```
 kubectl describe pod myapp-pod
 kubectl logs myapp-pod -c init-myservice
 kubectl logs myapp-pod -c init-mydb
 kubectl logs myapp-pod -c myapp-container
 ```
 ### 4. Cleanup
 If you wish to remove the Pod and services afterward, use the following commands:
 ```
 kubectl delete pod myapp-pod
 kubectl delete service myservice
 kubectl delete service mydb

 ```
