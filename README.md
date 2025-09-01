# Konecta Week 5 (Kubernetes)

## Part 1

### Q1
```bash
kubectl run my-nginx --image=nginx --restart=Never
```
This creates a pod named my-nginx from nginx image, restart=never makes sure that it is a single pod instead of kubectl run defaulting to a deployment
![](images/p1q1.png)

### Q2
```bash
kubectl run my-nginx --image=nginx123 --restart=Never
```
This should create a pod named my-nginx from nginx123 container image
![](images/p1q2.png)

### Q3
The status "AlreadyExists: pods "my-nginx" already exists" means that there is already a pod with the same name so I must choose a different name for the pod

### Q4
```bash
kubectl describe pod my-nginx
```
This command lists all the detailed information about a specific pod like its name, IP, container image used, etc... using its name
![](images/p1q4.png)

### Q5
```bash
kubectl delete pod my-nginx
```
This command deletes the pod named my-nginx
![](images/p1q5.png)

### Q6
```bash
kubectl apply -f k8s/p1q6.yaml
```
This command applies the yaml file with a pod named my-nginx, along with some test labels
![](images/p1q6.png)

### Q7
```bash
kubectl apply -f k8s/p1q7.yaml
```
This command applies the yaml file with a replicaset named my-nginx, then specified the number of replicas to 3 using the nginx image
![](images/p1q7.png)

### Q8
```bash
kubectl scale --replicas=5 rs/my-nginx 
```
This command scales the amount of replicas in the replicaset to 5, which automatically creates 2 more pods to accommodate for the new desired state
![](images/p1q8.png)

### Q9
```bash
kubectl delete pod my-nginx-8jgww
```
This command deletes one of the pods made by the replicaset. However, immediately after deleting it, the replicaset provisions a new pod to deploy
![](images/p1q9.png)

### Q10
```bash
kubectl edit rs my-nginx
```
This command allows me to modify the number of replicas from the cli without using the scale command, it opens vi for me to edit the spec.replicas to 2, then save the changes
![](images/p1q10.png)

### Q11
This Yaml provided has an error, where it uses the label "tier" to match the replicas but it did not specify this label in the template above, even attempting to apply this yaml proves my claim
![](images/p1q11.png)

### Q12
After careful inspection of the faulty provided yaml file, the error I found was that "deployment" was used instead of "Deployment" which does not match the known scheme
![](images/p1q12.png)

### Q13
After inspecting the faulty provided yaml file, the error I found was that the apiVersion was "v1" instead of "apps/v1" which does not match the known scheme
![](images/p1q13.png)

### Q14
```bash
kubectl get deployments -o wide
```
You get use the command above to get detailed information about all deployments, including used container image, or you could use ``` kubectl describe deployment <deployment-name> ```
![](images/p1q14.png)

### Q15
```bash
kubectl apply -f k8s/p1q15.yaml
```
Applying this yaml file creates the deployment with the specified parameters
![](images/p1q15.png)

### Q16
```bash
kubectl set image deployment/httpd-frontend httpd=nginx777
```
This command changes the image in the specified deployment. However since this new image does not exist, the deployment is stuck in an imagepullbackoff and the number of current pods is 4, 1 is up to date and in error state, and the rest are on the old version but functioning
![](images/p1q16.png)

### Q17
```bash
kubectl rollout undo deployment/httpd-frontend
```
This command allows me to undo to the previous version, in case the current version is faulty 
![](images/p1q17.png)

### Q18 to Q23 are in the ```/p1q18-23``` directory since they are related
#### Q18
```bash
docker build -t gimmeursocks/simple-webapp:v1 .
docker login
docker push gimmeursocks/simple-webapp:v1
```
These commands use the dockerfile to build, authenticate, then push the image to docker hub using my credentials
![](images/p1q18.png)

#### Q19
```bash
kubectl apply -f frontend-deployment.yaml
```
This command applies my frontend deployment that uses the image I previously pushed to docker hub
![](images/p1q19.png)

#### Q20
```bash
kubectl apply -f frontend-service.yaml
```
This command applies my nodeport service that exposes the frontend, and since im using minikube, I have to retrieve its IP first then check the port
![](images/p1q20-1.png)
![](images/p1q20-2.png)

#### Q21
```bash
kubectl apply -f backend-deployment.yaml
```
This command applies my backend deployment that uses a python slim image with a list of commands to start the http server at port 8080, now I must expose it internally
![](images/p1q21.png)

#### Q22
```bash
kubectl apply -f backend-service.yaml
```
This command applies my backend service, since this service does not need external access, then clusterIP is sufficient, then testing using port forwarding tells me that it is working
![](images/p1q22-1.png)
![](images/p1q22-2.png)

#### Q23
```bash
kubectl apply -f frontend-lb-service.yaml
```
This command applies my frontend service of type loadbalancer, since I am using minikube, there is no external IP, so just like the nodeport above, I should use minikube IP + assigned port to access the simulated loadbalancer
![](images/p1q23-1.png)
![](images/p1q23-2.png)

### Q24
```bash
kubectl apply -f k8s/p1q24.yaml 
```
This command applies my daemonset, which ensures that one pod runs on every node, it is usually used for logging agents and monitoring agents like Prometheus node exporter
![](images/p1q24.png)

### Q25
#### Components of the master node are:
- API Server: The entry point which exposes k8s api using REST
- etcd: Key-value store of cluster config and state
- Controller Manager: Ensures desired state matches the actual state of deployments, endpoints, etc..
- Scheduler: Assigns pods to nodes based on resources
- Cloud Controller Manager (optional): Integration with cloud providers on loadbalancers, volumes, etc..

### Q26
#### Components of the worker node are:
- kubelet: Agent running on each node, ensures that the pods are actually running
- kube-proxy: Manages networking and load balancing for services
- Container Runtime: The engine that actually runs the containers (Containerd)
- Node OS: Linux/Windows providing system resources

## Part 3

### Q1
```bash
minikube start --nodes=2
```
By default, minikube only starts with one node, but when you explicitly mention to create 2 this makes it so the first node is the master node and control plane, then the second node is the worker node
![](images/p3q1.png)

### Q2
```bash
kubectl create namespace fe
kubectl create namespace mongo-db
kubectl create namespace mongo-express
```
Creates 3 namespaces to deploy our seperate applications in them
![](images/p3q2.png)

### Rest of the implementations are in ```/bonus``` directory

#### Q3A
```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```
These commands create a deployment and service of the frontend service in the fe namespace, with 2 replicas and an emptyDir volume to store the web content
![](images/p3q3a.png)

#### Q3B
```bash
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml
kubectl apply -f mongodb-pv-pvc.yaml
kubectl apply -f mongodb-secret.yaml
```
The deployment runs mongodb with credentials from secret, the pvc+pv provide storage for ```/data/db```, and the clusterIP service exposes mongodb internally on port 27017
![](images/p3q3b.png)

#### Q3C
```bash
kubectl apply -f mongo-express-deployment.yaml
kubectl apply -f mongo-express-service.yaml
kubectl apply -f mongo-express-config.yaml
```
The deployment runs the mongo-express pod, nodeport exposes mongo express UI externally on port 30081, and the configmap stores username, password and service address
![](images/p3q3c-1.png)
![](images/p3q3c-2.png)

#### Q3D
```bash
kubectl apply -f mongo-db-networkpolicy.yaml
```
The networkpolicy restricts access to the mongodb pod so that only pods in the mongo-express namespace can connect. This improves security and only allows a subset of pods to access mongodb
![](images/p3q3d.png)

#### Q4A
```bash
kubectl taint nodes minikube-m02 db=NoSchedule:NoSchedule
```
This taint prevents any pod from scheduling on minikube-m02 unless it has a matching toleration
![](images/p3q4a.png)

#### Q4B
Add these lines to spec.template.spec in ```mongodb-deployment.yaml```:
```bash
tolerations:
    - key: "db"
    operator: "Equal"
    value: "NoSchedule"
    effect: "NoSchedule"
```
then apply the changes:
```bash
kubectl apply -f mongodb-deployment.yaml
```
Modifying the mongodb deployment by adding a toleration for ```db=NoSchedule``` makes this mongodb pod specifically scheduled on the tainted node, other pods won't be scheduled here
![](images/p3q4b.png)