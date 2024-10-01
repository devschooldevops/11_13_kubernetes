1. Install minikube using the instructions below:
https://minikube.sigs.k8s.io/docs/start/
If encounter error on starting minikube, download the suggested update.

#Check if minikube is correctly installed
minikube status

#Display addresses of the master and services
kubectl cluster-info

#Dispaly the node
kubectl get node

#Observe that the docker container created is in fact our worker node
docker ps

#Show kubeconfig settings
kubectl config view 						# cluster authentication can be checked at .kube/config location)

#Display list of contexts
kubectl config get-contexts                 

#Explore the app
kubectl get pod -A   						#get pods from all namespaces
kubectl get --help   						#help is your savior
kubectl get namespace						#get all namespaces
kubectl create namespace your_namespace     #create your own namespace

2. Let's create a pod
create a file named pod.yaml at a location of your choice with the following content:

apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args:
    - sleep
    - "1000000"
	
kubectl apply -f /your_location/pod.yaml    #creates a pod

#More about pods: https://kubernetes.io/docs/concepts/workloads/pods/

3. Let's deploy our first application:
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment

#Create the Deployment by running the following command with the content provided above:
kubectl apply -f /your_location/deployment.yaml -n your_namespace  

kubectl get all -n your_namespace  				      #List all the resources running in your_namespace (see the correlation between pods-replicaset-deployment)
kubectl describe deployment -n your_namespace   #explore the attributes of your deployment

4. Let's scale and update our deployment
kubectl scale --replicas=5 deployment nginx-deployment -n your_namespace   #scale the deployment up to 5 pods 
kubectl scale deployment/nginx-deployment --replicas=5 -n your_namespace   #alternative to the 1st command
kubectl edit deployment/nginx-deployment -n your_namespace				         #alternative to the 1st command

In fact, we will use the last command to upgrade our app with a new docker image.
First, let's open another terminal window and run this command:
watch kubectl get po -n webapp     #this way we'll keep an eye on our pods

#Go back to the initial terminal and run:
kubectl edit deployment/nginx-deployment -n your_namespace
#the yaml file will open. Search for the image field and replace it with 1.20.2
#save and exit the editor.

kubectl get deployment -n your_namespace -o yaml  #observe the new image and check again all the resources in your_namespace for a surprise (hint: replicasets)

5. Let's expose our application
https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

#Let's create our first service.
Create a file named service.yaml at a location of your choice with the following content:

apiVersion: v1
kind: Service
metadata:
  name: my-nginx-svc
  labels:
    app: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: nginx
	
#Can you guess the command to create this resource?	
kubectl apply -f service.yaml -n your_namespace

#check your service
kubectl get service -n your_namespace    #check out the IP and port

#Let's now create a pod imperatively which will do a curl on our service:
kubectl run -it --rm curl --image=nbrown/curl --restart=Never <your_cluster_ip>:80
#Tadaa! Our service is exposed internally and we checked that from the default namespace.

#The imperative way to create a service:
kubectl expose deployment/nginx-deployment -n your_namespace --type=ClusterIP --port=80

6. Clean-up resources
kubectl delete namespace your_namespace

Uninstall minikube from control panel, tho' it's fun.

