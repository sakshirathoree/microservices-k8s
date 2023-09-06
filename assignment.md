# Deploying Microservices: A Guide to Persistent Volumes in Kubernetes

Microservices architecture has become increasingly popular in modern software development due to its ability to break down complex applications into smaller, more manageable components 
In this project guide, We'll deploy a two-tier application that combines the power of a Flask Python backend with a MongoDB database—all within the Kubernetes ecosystem, We will also be using Persistent Volumes (PV) and Persistent Volume Claim (PVC) while doing this project.

But, before we begin this excitingMicroservice deployment journey, let's make sure you meet the following prerequisites:
## Prerequisites

1. **Knowledge of Kubernetes (K8s):** Understanding the basics of Kubernetes, including Pods, Deployments, Services, and Ingress controllers, is essential. You should be familiar with how K8s manages containerized applications.

2. **Microservice Application Code:** You need the source code of your Microservice application. Ensure it's well-structured and containerized, with necessary dependencies specified in a Dockerfile.

3. **Docker Knowledge:** A basic understanding of Docker is crucial, as containers are fundamental to Kubernetes deployments. You should be able to create Docker images from your application code.

4. **A Running Kubernetes Cluster:** You must have access to a Kubernetes cluster where you can deploy your application. This could be a local cluster (e.g., Minikube) or a cloud-based solution (e.g., Google Kubernetes Engine - GKE).

5. **Kubernetes CLI (kubectl):** Install and configure the kubectl command-line tool to interact with your Kubernetes cluster. It's essential for managing your deployments and resources.

6. **DockerHub Account:** This blog requires an active DockerHub account that will be used to upload and download the docker image we will be building. You can create an account by visiting [Docker Hub](https://hub.docker.com/).

7. **Git:** Git installed on your Ubuntu machine.

8. **Docker & Minikube Installed:** If you haven't already set up docker & Minikube cluster, please refer to the installation guide https://github.com/LondheShubham153/kubestarter/blob/main/minikube_installation.md to get started.

### With all the prerequisites satisfied, let’s dive into our project!

# PART 1

## Step 1: Launch the EC2 instance: 
Launch the ubuntu instance with the instance type as `t2.medium`

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/41e1cf5f-7d81-47e1-9de1-58251023d83e)

## Step 2: Install the Minikube & docker engine on Ubuntu
Refer to below guide for installation:
https://github.com/LondheShubham153/kubestarter/blob/main/minikube_installation.md

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/66b2dd78-af78-4998-a508-58a15e202158)

## Step 3: Clone the source code of the project
Run the given command in your Linux terminal to download the project files:

```bash
https://github.com/sakshirathoree/microservices-k8s.git
```

## Step 4: Move to the project directory & Build the Dockerfile
The project folder includes a Dockerfile that will be used to build the image.

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/d21eeede-54bf-4a03-a849-5aee1b9b4564)


- **Dockerfile**
```
FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
ENV PORT 5000
EXPOSE 5000
ENTRYPOINT [ "python" ]
CMD [ "app.py" ]

```
Run the below-given commands inside the root of the project folder to build the docker image, and replace the placeholder <username> with your DockerHub username.
```
docker build --rm -t <username>/microservicesflaskapp:latest .
```
**The --rm flag ensures that any intermediate containers created during the build process will be automatically removed once the build is finished, helping to maintain a cleaner development environment.**

Use the command `docker images` to view your newly built image.

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/0339a8c4-bb39-46ea-9125-82af78c35e12)

## Step 5: Push the docker image to Docker Hub
After building the image, we need to push it to docker hub, our Kubernetes deployment will later fetch the same image from docker hub to run the app.

- **Log in to the DockerHub from CLI using `docker login`**

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/64bccc02-dd78-4207-8828-76bc8c6aa592)

- **Push the Docker image to DockerHub:**
  
  Use the command `docker push <username>/microservicesflaskapp:latest` to push the image, replace the placeholder `<username>` with your username.
  After pushing the image, refresh your DockerHub website, and you will be able to see a new repository has been created containing your docker image.
  
![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/d422becd-53b1-423d-af27-cfca43968bc9)


Congrats on reaching here! 🥳 You have now successfully built a docker image and uploaded it to Docker Hub, thus completing the basic steps of the continuous integration Process.
Now we will be focusing on deploying the docker image in a K8s cluster. For this, we will be using the Deployment object of K8s to create Pods that run our image.

Now we will be focusing on deploying the app in our K8s cluster. For this, we'll be creating a Persistent volume object that will store data of our MongoDB.

# PART 2

 Before we begin with our deployment, Let's first create the `namespace` 

 ##  What is namespace?
 - namespace is a logical entity that allows you to isolate resources like pods, volumes, deployments etc. It's a way to logically partition or segregate resources within a cluster.
 - **Resource Isolation:** Namespaces help isolate resources, preventing naming conflicts between objects like Pods, Services, ConfigMaps, and more. This separation enhances security and resource management.
 - **Organization:** Namespaces enable logical organization and categorization of resources. You can group related objects and apply policies at the namespace level.

Let's create a `namespace` in a project folder:
```
kubectl create namespace flask
```
![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/61624808-7b80-4367-8b72-90bc87fe7eb3)


 ##  What is Deployment?
 - Deployment in k8s is a controller which helps your applications reach the desired state, the desired state is defined inside the deployment.yml file
 - Deployments are used to manage and scale the replica sets of Pods(A Pod is the smallest deployable unit in Kubernetes, representing a single instance of a running process.
Pods can contain one or more containers, and  are used to deploy applications, and each Pod has its own unique IP address within the cluster)
 - They ensure a specified number of replicas are running and provide updates and rollbacks for application changes.

## Step 1: Creating a Deployment Manifest File of the Flask App
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: taskmaster
  namespace: flask
  labels:
    app: taskmaster
spec:
  replicas: 1
  selector:
    matchLabels:
      app: taskmaster
  template:
    metadata:
      labels:
        app: taskmaster
    spec:
      containers:
        - name: taskmaster
          image: rajlaxmii/microservicesflaskapp:latest
          ports:
            - containerPort: 5000
          imagePullPolicy: Always
```
Make sure to replace the value of image key with that of your image name as shown in the YAML file.

Run `kubectl apply -f deployment.yml` to create the deployment object in K8s. You should see an output similar to this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/693cf254-cf79-47d2-a62d-4b63739a2811)

The deployment creates 1 pods running 1 container each with the `rajlaxmii/microservicesflaskapp:latest` image we pushed earlier. The key **containerPort** declares that the container accepts connections from port 5000.

Run `kubectl get deployment -n flask` and `kubectl get pods -n flask` to ensure that your deployment has been successfully created and pods are running. 

Your output should look like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/c50e0963-dbee-4038-8bd7-cc7c9c4edb50)


**Don't forget to specify namespace using -n else it'll consider the default namespace & will not show any resources**

Our next step is to create a service object that will allow us to connect to the pods created by the deployment file.

##  What is Service?
- A Service in K8s serves as a link between distinct pods or microservices inside a cluster.
- It provide network access to a set of Pods, allowing them to be accessed by other Pods or external clients.
- They can be used for load balancing, service discovery, and routing network traffic.
- There are different types of Services, including ClusterIP, NodePort, and LoadBalancer.

## Step 2: Create the Service manifest File of the Flask App
You can view the taskmaster-svc.yml file present in GitHub or copy the YAML from here, create a file and then apply it.

```
apiVersion: v1
kind: Service
metadata:
  name: taskmaster-svc
  namespace: flask
spec:
  selector:
    app: taskmaster
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30007
  type: NodePort
```
Run kubectl apply -f taskmaster-svc.yml to create this service in K8s. Run kubectl get svc to check if the service has been created, the output should look like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/b1cd8974-3b04-4c8d-a010-eb041a4fe89b)

##  What are Persistent Volumes?
Persistent Volumes are storage resources in a cluster, decoupled from Pods, and provide a way to manage storage.
They ensure that even if your applications or containers change or restart, your valuable data remains safe and accessible. PVs are essential for preserving your database information within the Kubernetes environment.
They can be provisioned from physical disks or cloud storage services.
PVs can be dynamically or statically provisioned.

## Step 3: Creating a Persistent Volume Manifest for K8s
You can view the mongo-pv.yml file present in GitHub or copy the YAML from here:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
  namespace: flask
spec:
  capacity:
    storage: 256Mi
  volumemode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /tmp/db/mongodb
```
We are provisioning a 256Mi persistent volume with Retain policy, which means that the data in this volume will be preserved even if the corresponding persistent volume claim policy has been deleted.

You need to replace the value of the key path in the above YAML to point to a folder present in the node. In my case, I have already created a folder mongodata in the given path.

Create the persistent volume in your cluster using the command `kubectl apply -f mongo-pv.yml`, then run `kubectl get pv` to see the volumes available.

You should see an output like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/8cdb8555-abda-495b-8377-117b5cedaba1)


##  What are Persistent Volume Claim (PVC)?
Persistent Volume Claims are requests for storage by Pods.
They allow Pods to consume storage resources from PVs.
PVCs provide a way to abstract the underlying storage details from the application.

## Step 4: Creating a Persistent Volume Claim (PVC) Manifest in K8s
You can view the `mongo-pvc.yml` file present in GitHub or copy the YAML from here:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
  namespace: flask
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```
We are requesting a 256Mi volume with access mode as ReadWriteOnce, similar to what we have in the persistent volume we created above.

Create the persistent volume in your cluster using the command `kubectl apply -f mongo-pvc.yml`, then run `kubectl get pvc -n flask` & `kubectl get pv -n flask` to see the PVC and the volume state.

You should see an output like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/eef57cde-07d7-41c9-ac4a-79089b33f00a)

The persistent volume is now bound to the persistent volume claim policy as expected, Now let’s use this PVC in the deployment of MongoDB to allow it to store data.

## Step 4: Create a Deployment Manifest file of MongoDB
You can view the `mongo.yml` file present in GitHub or copy the YAML from here, create a file and then apply it.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  namespace: flask
  labels:
      app: mongo
spec:
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: storage
              mountPath: /data/db
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: mongo-pvc
```
Notice the key spec.template.spec.containers.volumeMounts where we ask Kubernetes to mount the volume to mount path /data/db, this is the path where Mongo db stores its data. We also declare the PVC that the containers will use while claiming data under spec.template.spec.volumes

Create the deployment in your cluster using the command `kubectl apply -f mongo.yml` and then run `kubectl get deployment -n flask` after a few seconds to make sure that the pod(s) are ready.

The output should look like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/4c338877-f211-42e8-a450-70cdb5094825)


## Step 5: Create a Service Manifest file of MongoDB
You can view the `mongo-svc.yml` file present in GitHub or copy the YAML from here, create a file and then apply it.
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: mongo
  name: mongo
  namespace: flask
spec:
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    app: mongo
```
Create the service in your cluster using the command `kubectl apply -f mongo-svc.yml` and then run `kubectl get svc -n flask` after a few seconds to make sure that the service is ready.

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/9c02838e-44b0-4cda-b029-80e8bf324ca6)


## Step 6: Test the Application

**Make sure to open port 5000 & 30007 in the Deployment Server of the security group**

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/aef315ff-9440-42aa-8c76-b1633b3892c8)

###  To get the URL for your application:
Get the url for your application using :
```
minikube service list

``
![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/2911b627-96a9-4ebf-8f24-f2f5468c9d32)

You can now test the application by using the curl command with the url you got above, make sure to replace `192.168.0.159` with your ip
```
curl 192.168.49.2:30007
```
![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/5f7c72e0-eb6d-4bdf-8329-96198c97c428)

Test MongoDB:

You can test if the application is working by trying to insert and then fetch data from the DB using the flask app requests GET /tasks and POST /task.

GET /tasks :
```
curl 192.168.49.2:30007/tasks
```
![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/02c1c3b1-227d-4f17-8a03-7e58ef465fa4)

The output would be like this considering this is the first time you have run this app and haven’t inserted any data yet:

Insert Data to MongoDB using POST /task path and then get all data using GET /tasks path:
```
curl -X POST -H "Content-Type: application/json" -d '{"task":"Show everyone the project"}' http://192.168.49.2:30007/task
curl 192.168.49.2:30007/tasks
```
The output will be like this:

![image](https://github.com/sakshirathoree/microservices-k8s/assets/67737704/3477190d-0c67-41b6-b3e6-094ae5290cf8)


## Congratulations on Your Microservices Project!

If you've any queries, feel free to reach me on LinkedIn:
https://www.linkedin.com/in/rajlaxmi-rathore29/
