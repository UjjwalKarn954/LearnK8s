# LearnK8s

## Monolithic and Microservices

Monolithic : The whole application is tightly coupled and deployed as a single unit.

Microservice: In microservices the functionalities are developed in smaller unit and each are deployed individually.

## Microservices Benefits and Drawbacks.

- Benefits
Easy to manage and deploy small chunks.
Scalability 
Fault tolerarance

- Drawback
Lot of services to rake care of
Testing might be liitle tough
Multiple databese


## CLoud Native: Contains:

### 1. Contanierization
### 2. CI/CD
### 3. Orchestration
### 4. Obervability and Analysis
### 5. Service Proxy, Discovery and Mesh
### 6. Networking and Security

### 1. Contanierization:

- Container is a unit of deployment/Software
  mostly having Code, runtime, system tools and system libraries

- Why Containers?
move faster by deploying smaller units 
use few resources
faster automations
portable 
isolation

- Virtualization vs COntainerization

In virtual machine - hardware is virtualized, OS boots up everytime which takes time.
In Continer: the os is virtualized, it dont have to boot up so it is faster.


Containers always stores the image in its local cash so when ever we download newer version it only downloads the layer not present.


- Container Registery: Centralized repository the contains images.

- Orchestrator: mange, scale, monitors the application that we run on our containers.

## Docker:
It is a containerization platform the provides the container runtime
We write "Dockerfile" to create an docker image

- Few docker commands
  docker pull
  docker run
  docker run -d in detached mode
  docker start
  docker stop
  docker ps 
  docker ps -a
  docker kill
  docker rm containername (to remove the container from the memory) 
  docker images 
  docker rmi to remove the image
  docker system prune -a  (removces all the images not in use)


#### Exercise 1
- Run the ngnix container

docker pull nginx (will pull the nginx image)
docker run -d -p 8080:80 --name webserver nginx (will runt the container with the nginx image and conatainer name will be webserver)
docker ps to check the running containers
docker container exec -it webserver bash (Attach to the container- to run the commnads inside the running container)
docker stop webserver (to stop the container- but still will be in the memory, docker rm webserver to remove from the memory)

#### Exercise 2
- Building docker containers using Docker CLI

Dockerfile: It is a static HTML site, or a blueprint to create an image written in yml.

Basic structure of Docker file: It contains the layer of instruction to create an image.

FROM <base-image>
COPY <path-to-yourfiles> <path-to-the-image-folder>
WORKDIR <path-to-your-image-folder?>
RUN <run the commands inside to create an image> 
EXPOSE <PORTS:PORTS>
CMD [<Commands to rum after the image is created>]

#### Exercise 3
- Containerize an Express website

We need to have a sample app with us: will attch the link to my github for that.
We can use VS-Cose as editor and install plugin like docker.

-Create the Dockerfile

FROM node:lts-alpine
ENV NODE_ENV=production
WORKDIR /usr/src/app
COPY ["package.json", "package-lock.json*", "npm-shrinkwrap.json*", "./"]
RUN npm install --production --silent && mv node_modules ../
COPY . .
EXPOSE 3000
RUN chown -R node /usr/src/app
USER node
CMD ["npm", "start"]

- Build an Dockerfile

docker image build --pull --file '/Users/ujjwalkarn/Projects/PersonalProjects/LearnK8s/Dockerfile' --tag 'myexpressapp:latest' --label 'com.microsoft.created-by=visual-studio-code' '/Users/ujjwalkarn/Projects/PersonalProjects/LearnK8s'

docker build -t 'myexpressapp:latest' .

- Run the Container 

docker run -d -p 3000:3000 --name <name of container> myexpressapp:latest

Can check the status of container and localhost:3000

Now try stopping the container and removing them
Also try removing the images created


## Volumes:
Containers are ephemeral by nature; when they stop or are removed, any data stored within them is lost. Volumes provide a way to persistently store data outside the container's filesystem. This ensures that data remains even if the container is stopped or removed.
- Maps a folser on the host to a logical folder in the containers

Few commands for volumes

docker create volume <volumename>
docker volume ls
docker volume inspect
docker volume rm
docker volume prune

#### Mapping the volume

create colume
- docker volume create <volumename>
inspect volume
list the volume and then,
run the container with the volume
- docker run -v mydata:/path/in/container myimage

####  Creating a new volume

- Crete and run the container with the volume

docker create volume <volumename>

docker volume ls (to check the status of the volume)

docker run -d --name voltest -v myvol:/app nginx:latest (create new container with the volume)

Now we try to execute some command in side the container.
 
docker exec -it voltast bash
 takes us inside the volume and no can ls to check the file.

Try doing some stuff in the volume.

few commands that i tried to execute
 apt-get update 
 apt-get install nano

 cd app (enter the folder we created earler)

 nano test.txt 
  write few things save and exit

- Now stop the container and remove the contaier, still you will have your created volume with you.

to check that crete and run new container using the same volume and try to exec the volume and see the data, it will be still there. This is the beauty of volume.

- You can delete the volume as well
 docker volume rm <name>

## Yaml: Concetps 

It is human-readable language compatible for docker conpose and kubernetes (and many other )
- sample yaml file

version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    restart: always
  
  app:
    image: node:latest
    working_dir: /usr/src/app
    volumes:
      - ./app:/usr/src/app
    command: npm start
    restart: always

- we have few plugins to check the yaml file for errors like: YAML , yamllint and others

## Docker Compose: Concepts

- Assume you have multi container app: frontend in one container, backend in other, database in other. So, you will end up having multiple contianers in the app.

What if we have single file to manage all the contianers oa a single app.

#### Docker Compose: Helps to define and run multiple containers using simgle Yaml file

- Sample docker-compose.yaml file

version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    restart: always
  
  app:
    image: node:latest
    working_dir: /usr/src/app
    volumes:
      - ./app:/usr/src/app
    command: npm start
    restart: always
  
  db:
    image: mysql:latest
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: example_password
      MYSQL_DATABASE: example_db
      MYSQL_USER: example_user
      MYSQL_PASSWORD: example_password
    volumes:
      - db_data:/var/lib/mysql
    restart: always

volumes:
  db_data:

Here we have 3 containers in side one file.
- local volumes in Docker provide a way to persist data on the host machine's filesystem and share it with containers. Unlike named volumes that are managed by Docker and stored within Docker-managed directories, local volumes are directories created explicitly on the host's filesystem and then mounted into container

- Use cases: If we don't need full orchestration for a small apps, while development and testing, ECS, Virtual machine etc

- Few Commands

docker compose build ( to build the image)
docker compose start ( to start the container)
docker compose up -d (to build and start the container)  
docker compose down (stop and remove the container)
dcoker compose ps (lis the containers)
docker compose rm to remove from the memory
docker compose logs (get the logs)
docker compose exec containername bash ( to run the command inside the container)

#### Exercise to build the app using docker compose

- lets practice this using a simple python app

we have a simple in the app.py(web and redish) file, a docker file, a docker compose file and a requirements file that contins the rewuiremnet for the python add that needs to be installed.

docker compose up -d (will build and start ther containers)
can check the running containers with docker ps or docker compose ps
 you can see the containers with the root folder name: it takes bydefault.

- if we try to run the command again  then it will say the containers are already running, but we can create new project with new name. aslo have to change the port number as they are already in use
command: docker compose -p name up -d

- lets clean up those containers using
docker compose down (for default)
docker-compose -p name down (for created with the name)

#### Exercise to deploy bit complex app using docker compose

- we have a app with the frontend build in react, backend build iun node and databse in mariaDB

-- In the docker file cn see multi-stage build for smaller final dokcer size and more cleaner deployment.

as before run the docker with the up command and check in the browser.

- docker compose logs -f backend (command to see the logs of a particular service)

#### Docker compose features:

- resource limit:
  we can set the initial limit for the docker to utilize and the maximum limit.
- Environmrnt variables:
we can set the environment variables  and can override at runtime
- Networking:
all the services uses default network
also we can se the network conection in docker and define teh network in containers
- Dependence:
we can set dependence 
depends_on (to let the service to start the service that it depends on.)
- volumes:
we can set the volumes of the services
- Restart Policies:
we can set the restart policies to restart the services on different circumstances

#### Conainer Registery:
- Centerlized repository for container images.
like Docker Hub, ECR etc

#### Working on Docker HUB:

- Login to docker using your credentiasl
docker login (will ask your credentials)
- tag the image built
docker tag <imagenaem> <username/reponame:version>
- push the docker image
docker push <taged_name>
- pull the image
docker pull <taged_name>

- We can have the private repo as well if you don't want to share it to anyone.

# Kubernetes a.k.a. K8s

It was developed by Google (in 2015)
later was donated to CNCF

- Kubernetes is container orchestration tool, designed as a loosely coupled collection of components centerd around deploying, maintaining and scaling workloads.

#### What k8s can do?
- service descovery and loadbalancing
- storage orchestration ( local or cloud based)
- Automated rollout and rollbacks
- self healing
- secret and configuration management
- use same api across on-premise and every cloud providers

#### What K8s can't do?
- Does not deploy source code
- Does not build application
- Does not provide application level services (messag buses, databases, cashes etc)

#### K8S Architecture:

- K8s has two Nodes(Master and slave or worker)
- Master nodes contolls the worker nodes.
- master node has:
kube controll manager
cloud controll manager
kube api server
etcd
kube scheduler
- worker node has 
kubelet
kube proxy
(a master node can have multiple worker nodes)

Container runs in a pod
pod runs in a node
and a node forms a cluster

#### Running kubernetes locally

- requires virtualization
 docker desktop
 micro k8s
 minikube
- runs over docker desktop
kind

- Local k8s
docker-desktop is limited to a single node

for multiple nodes we can ue minikube microK8s or kind

- minikube: requires hypervisor like virtual box ( can run on linux, mac and windos )
- kubectl is a command line tool to interact with kubernetes.

#### Exercise to check the kubernetes

- install minikube on your machine
brew install minikube (for mac)
and then check with ( kubectl cluster-info)

#### K8s CLI and context

- K8S API: it is the service running on the master node that expose REST:API the only point to communicate in k8s
we define the desire state in yaml file (x num of instance) then send the desire state to cluste using cli

- kubectl can be also called kube control kube kuttle etc
it communicates with api server

#### K8s Context
- Group of access parameter that lets you connect to k8s cluster
- Contains a cluster, user, namespace
- current context is the cluster that is currently the default for kubectl
all kubectl commands run against that cluster

- kubectl context commands

kubectl config current-context (get the current context)
kubectl config get-contexts (list all contexts)
kubectl config use-context <context-name> set the current context (shortcut kubectx)
kubectl config delete-context <name> (delete that context from delete file)
kubectl config rename-context <oldname> <newname> (rename the context name)

the context data is stored locally on your machine

#### There are two ways to create resources in kubernetes: 
- Declerative and Imperative

Imperative: we use kubectl to create resources with series of commands
declerative: using kubectl and menefists defining the resources, can save for reuse.


#### Yaml file explanation:

root level rewuired properties

api-version
kind (type of obj)
metadata.name (unique name of the object)
metadata.namespace (scoped env name, by default it takes current)
spec (obj specification or desired state)
- kubectl create -f [yaml file] (create an obj using yaml file)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
        - name: sample-app-container
          image: nginx:latest
          ports:
            - containerPort: 80

to check the deployments 
kubectl get deployments
and kubectl delete deployment <name> (to delte the deployment)

#### Note: deploymet >> handles the creation and scaling of pods, Deployment and Pods: Deployments manage the creation and scaling of pods based on the defined specifications, ensuring the desired number of replicas are maintained.
#### Service and Pods: Services provide a stable endpoint to access pods regardless of changes in their underlying IP addresses or lifecycle, allowing pods to communicate with each other or exposing the application externally.
#### Node and Pods: Nodes host and run pods, providing the necessary resources and environment to execute containerized applications.

#### Namespaces

- allow to group resources: eg dev test prod 
k8s creates a default workspace
obj in one namespace can access obj in other
seleting a namespace will delete all its child onj

we define the namespace and specify the namespace when defining obj

few commands

kubectl get namespace (list all namespaces)
kubectl get ns *shortcut
kubectl create ns [namespace]
kubectl delete ns [namespace]
kubectl get pods --all-namespaces (list all the pods in all namespaces)

### Master Node: Control plane

Nodes are physical or VM, group of nodes forms a cluster.
we dont run containers on master nodes

#### kube api-server:
- Exposes REST api
- save state to etcd
- all clients interacts to it, never directly to datastore

#### etcd:
- stores the state of cluster
- key-value pair
- not a database for application

#### kube control manager:
- controller of controller!!

#### cloud control manager:
- interacts with cloud controller
- checks if nodes are created
- it manages load balancer
- manges volumes

#### Kube scheduler

- checks for the newly creaded pods if nodes are assigned to it or not

#### Addons:
- DNS
- webui (dashboard)
- cluster level logging
- container resource monitoring

### Worker Node: 
- The nodes runing the container is called worker node
- when the worker node is added to the cluster, few components are created by default.

#### kubelet
- manages the pods lifecycle
- ensure the resources defined in the container is working and healthy

#### kube proxy:
- network proxy
- manages network rules on nodes

#### Container runtime
- k8s supports several runtime
- must implement the container runtime interface

- manages the container lifecyle 
 the k8s interact with conatiner runtime interface to create conatiner and mange it allocate tht eresources to it

#### Node pool:
- node pool is group of virtual machine, all of same size
- a cluster can have multiple node pools

### Pods:
- atomic unit of k8s
- encapsulate application contaainer
- represents unit of deploment
- pods can run one or multiple conatiners
- contaienrs inside a pod shares same IP
- one pod faiels another new pod with nerw ip is created
- to scale we add pods not conatiner

#### Note: A node can have multiple pods  and a pod can have one or more containers
>>>>> in a pod if there is multiple containers running the one conatiner is main adn other is just helping conatiner

##### Pod lifecycle:
- creation of pod
- deletion of pod

##### pod state
- pending: accepted but not created
- running; bound to a node
- succedded: exited with status 0
- failed: 
- unknown : issue with communication
- crashloopbackoff: staerted, crashed, started again, crashed again....

#### Define and run pods:
- we create a yaml file and create with the command aslo we can execute inside in interactice mode 
 kubectl exec -it [podname] --sh

#### init containers

asume we have app with dependency on may be database or anything, we want the db to start first
so this init conatainer initialize the pods before the application container runs and then the init containers stops .
example temp

apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  containers:
    - name: main-app-container
      image: nginx:latest
      # Your main application container configuration

  initContainers:
    - name: init-setup
      image: busybox:latest
      command: ['sh', '-c', 'echo Initializing... && sleep 20']
      # Replace the command with the initialization task you need


### Selectors:
- selectors use lables to filter or select objects.
#### Labels: 
- key-value pair used to identify,describe and group related sets of obj or resources.


#### exercise: 
have two yaml files one with pod and other for service add same lebels and deploy both
to check the connection: the endpoint of service will point to pod ip.

kubectl get po -o wide

then check the service endpoint the id should match

kubectl get ep servicename

now portforward to the service and check the url

kubectl port-forward service/myservice 8080:80

#### Multi container pods
- Most common senerios in multi container pods there is always 1 main continer and other container provides services to the main node, like writing log files ,storing data ets

#### Defining multicontainer pods

apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: first-container
    image: your-first-image:tag
    ports:
    - containerPort: 8080
  - name: second-container
    image: your-second-image:tag
    ports:
    - containerPort: 8081


few commands
kubectl create -f {pod-defination.yaml}
kubectl exec -t podname -c containername (exec into container)
kubectl logs podname -c container

#### Networking Concepts:
- all continer can communicate within a pod
- all pod can communicate with each other
- all nodes can communicate eith all pods
- pods a re given an ip address
- services are given a presistent ip address

- Each pods get their own ip addresses
- contaiers inside shares the same ip spaces
- containers inseide pos can share same volume
 

- container inside one pod can access to container from other pod via service
- Wxternal access happens through loadbalancer provided by cloud

#### Workloads:
- workloads is an application running on kubernetes
  pod
  replicaset
  deployment
  statefulset
  deamonset
  task that run to completion 
     - job
     - cronjob

#### Replicaset:
- manage replicas and lifecycle of pods, always ensures the desired number of pods are running (helps in self-healing)
- while we can create replicaset, recomended way is to create Deployments ( because it provide extra functionality)

- we create replica set file and add the details of pod and required config 

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  --------------
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: your-image:tag
        ports:
        - containerPort: 8080
  ------------------ (this is for pod )


#### Deployments:

- compare pod vs deployment

Pods dont selfheal
sacle
updates
rollbacks

where as deployemt can.
- deployment manages single pod templates
- we create one for each microservice
- deployment creates replicaset in background which we need not to manage
example of deploymet

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80

#### DaemonSet:
- Ensures all nodes run an instance of a pod
- asume ther is one instance of pod needs to be running on every node

#### Statefulset:
-  it is mostly for database.
- if we need to have pre4sistent poda or sticky database then we use statefulset. 
- we can have presistent identifier as well as storage.
- if one pod dies other with the same identifier is created
- by default containers are stateless in nature so to make it statefull we will have to use some cloud provider database deleting the stateful wont delete the PVCs we have to delete them manually

PVC: presistent volumes

few commands:
kubectl create if --
kubectl get sts
kubectl describe sts name
kubectl delete sts name
kubectl delete pvc name (to delete the pvc)

#### JOB
- jobs are workloads for short lived task
- creates one or more pods and ensures that the specified number of them sucessfully terminated
- when the specified number of sucessfully completion is reached the jobs stops
 
 has the same commands 
 kubectl apply -f name
 delete job
 get job

example
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  parallelism: 3
  completions: 5
  template:
    metadata:
      name: my-job
    spec:
      containers:
      - name: my-container
        image: busybox:latest
        command: ["echo", "Hello, Kubernetes!"]
      restartPolicy: OnFailure


#### Cron Jobs: runs the job on schedule.
- extension of jobs 

to check if it was sucessfull
can check the history - kubectl get pods 

by default it stores last three sucessful jobs and last failed job

commands:
kubectl apply -f name
describe cj
delete cj
get cj

example
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          name: my-cronjob-pod
        spec:
          containers:
          - name: my-cronjob-container
            image: busybox:latest
            command:
            - /bin/sh
            - -c
            - date; echo "Hello from my CronJob"
          restartPolicy: OnFailure

### Strategy
we use this to create replicas

#### Rolling updates:
- one pod is deleted and and new is created and again other pod is deleted and new created, basically we can say every pod is replaced one by one not all ata a time 

#### Recreate 
- all previous pods are deleted and then created all at once

few commands:

kubectl apply -f 
kubectl rollout status
kubectl rollout undo (rolback to deployment)

#### Blue-green deployment:
- blue identifies what is in production and green identifies newer version deployed but not yet in production

you can point the service defination to green ( which becomes blue and the blue becomes green)

still have downtime while updating the service defination

#### Services:
- assume pod in green need to access pod in blue, need to use its ip address but the ip address of pod is not presistent so.

- Service is another type of k8s obj 
which has presistent ip address and DNS name which is used to access pods
- it target pods using selectors

- In k8s we can use these services

##### Cluster IP (default):
- it is default service and its visiswbility is cluster internal
port: it is the service listining to
targetport: redirecting to the port the pods are listining to
- provide durable communication way inside a cluster

- it is not possbibe to access from out side of cluster

##### Node Port: 
- it is extended od cluster ip
it is visible internally and externally
NODEPORT : the port service is listining to externally
 it is statically or dynalically taken froam a range 30000 - 32767
port: the port is listining to internally
targetport: redirecting to pods

- node must have public ip address, use nodeip + nodeport to access service

we have the commands to expose nodeport for running pods

##### Loadbalancing L4:
- l4 operating at transport layer (TCP) 
##### Ingress L7:
- Ingress id like loadbalancer but more intelligent
operates at application layer (HTTP SMTP)

#### Storage and presistence
- containers are stateless the data gets deleted after the deleteion of containers
so we use storage outside of the container in a volume.

we have different types of volume for aws we have EBS
and similarly for others as well
- we can have PVs presistent volume and can give different access modes like readonly read wrtie etc

#### ConfigMaps
-  it decoples the configuration arifacts from conatiner image, we can store the key value, parameters, env variables taht can be used by pods.
example
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_URL: "mysql://db.example.com:3306"
  API_KEY: "abcde12345"

------- This configmaps parameters will be consumed by the pods created below

apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: my-app-image
    envFrom:
      - configMapRef:
          name: app-config
- it is static
the value is consumed at the runtime so to make the update we have to restart ther container
- so to solve the issue
we mount the config file in the volumes.
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: my-app-image
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: config.ini
        path: app-config.ini


#### Helm: is package manager for k8s that simplifies the deployment and management of application with the predefined templets.
helm create app-chart ( to creade the directories structure for helm)
helm template [chart] [flags] (to generate template)
helm install [chart] [flags] --set image.tag=v2 (to update the deployed application)

#### Secrets
- stores the data in base64 as it is not encrypted its not that secure
so either we protect them with RBAC or we use any other secrets store

#### observability: 
- if the containers dies then k8s recreats the container (pods) but what if the app crash and the conatiner is still running? 
- so to monitor the apps we can use * Probes * 
 apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: probe-container
    image: my-app-image
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 10
      periodSeconds: 5
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      failureThreshold: 30
      periodSeconds: 10
  restartPolicy: Always

- we have liveleness, readyneess and startup probes

#### DASHBOARD:
- to see the logs and pods running on dashboard we can use either of these: K8s, lens, k9s
web-ui: dashboard
lens:IDE
k9s: dashboard in termianl

#### Scaling pods: We have horizontal pods autoscaling (HPA)


#### Monitoring and alerting: Prometheus, Grafana etc
- we also have k8s native: metrics server



xxxxxxxxx------------------------------------------------------------------------------------------------------------------------------------------------------------xxxxxxxxx