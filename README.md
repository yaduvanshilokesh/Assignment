## Environment :
- Kubernetes Cluster: AWS EKS 

- Container repository: AWS ECR

## Requirements :
- Must have **aws-cli** installed and configured on your local machine

- Must have **eksctl** installed on your local machine

- Must have **kubectl** installed on your local machine

- Must have **docker** installed on your local machine

## Creating docker image and pushing it to AWS ECR  :
- Create a container repository **nodejs-test** on ECR

- Authenticate your docker client to your registry by ruuning the following command

*aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 226026992666.dkr.ecr.us-east-1.amazonaws.com*


- Move to **docker-image** directory and build the docker image by running the following command

*sudo docker build -t 226026992666.dkr.ecr.us-east-1.amazonaws.com/nodejs-test:latest .*


- Push the image to container registry by running the following command

*sudo docker push 226026992666.dkr.ecr.us-east-1.amazonaws.com/nodejs-test:latest*

## Creating EKS cluster using eksctl :
- Run the following command to create the EKS cluster and node-group

*eksctl create cluster --name test --version 1.16 --region us-east-1 --nodegroup-name test-nodes --node-type t3.small --nodes 3 --nodes-min 1 --nodes-max 4 --ssh-access --ssh-public-key assignment --managed*

## Deploying metrics server on cluster :
- Metrics server collects the metrics of cpu and memory utilization by the pods and nodes, it is required by the horizontal pod autoscalar to autoscale the replicas of deployment based average cpu and memory utilization. 

- Move to **menifests** directory and execute the following command

*kubectl apply -f components.yaml*


- After successfully deploying the metrics server run following commands to fetch the metrics of nodes and pods

*kubectl top nodes*

*kubectl top pods*

## Deploying PriorityClass Object on cluster :
- PriorityClass object defines the priority of the associated pod, priority is a integer value that is mapped to the priority class object. Highest value of user-defined priority class object can be **1000000000.** 

- Move to **menifests** directory and run the following command to deploy priority class first

*kubectl apply -f priorityclass.yml*

## Deploying nodejs application on cluster :
- Move to **menifests** directory and run the following command

*kubectl apply -f deploy.yml*

###### Deployment properties :

- Ensures that 10 replicas are running

- Uses custom image nodejs-test:latest from ECR container registry.

- In spec.strategy, maxUnavailable=3 ensures that atleast 10-3=7 replicas are always running.

- priorityClassName = highestuserdefinedpriority, ensures that the pods are having higher priority than the daemonSet's pods.

- All pods are limited to use a specific amount of cpu and memory.

## Deploying horizontal pod autoscalar :
- Horizontal pod autoscalar auto scales the replicas of application based on cpu and memory utilization, it uses the metrics server to get the utilization metrics.

- Autoscales the replicas from 10 to 13 if average cpu utilization is more than 50% or average memory utilization is more than 60%.

- Move to **menifests** directory and run the following command

*kubectl apply -f hpa.yml*

## Deploying load balancer service :
- Load balancer service deploys the classic EC2 load balancer and load balances the traffic to the application.

- Exposing the application on port 3000 to the internet.

- Move to **menifests** directory and run the following command

*kubectl apply -f service.yml*
