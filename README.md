# Ambassador pattern with microk8s and redis #
This project highlights the Ambassador Pattern from the book "Designing Distributed Systems".  I used k3d.io for managing k8s clusters. I like the simplistic feel. Yes, the kubernetes OOB are great tools as well for single node (minikube + kind).

## create k3d cluster
```
sudo k3d cluster create <cluster name>
```
## stop cluster
```
sudo k3d cluster stop
```

## List all namespaces
```
sudo kubectl get all --all-namespaces
```

## create a namespace
```
sudo kubectl apply -f ./ns.yaml
```

## deploy the Dashboard
The Kubernetes documentation gives you the url to download the dashboard.

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

```
sudo kubectl apply -f kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
```
### Create Dashboard User & Role ###
```
sudo kubectl create -f dashboard-admin-user.yaml -f dashboard-admin-user-role.yaml
```
### Retrieve the token ###
In order to login we must get the token for the admin user we created.
```
sudo kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token
```
## Open the dashboard
I recommend using a dedicated terminal as this command will create a token that
you need to login.  Also, use firefox to open the url: 
```
sudo kubectl proxy
```
Dashboard url
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

## Expose Services to localhost
NodePort

Expose the service by defining a nodePort:
```
apiVersion: v1
kind: Service
metadata:
 name: apaches
spec:
 type: NodePort
 ports:
   - port: 80
     nodePort: 30000
 selector:
   app: apache
```
## Install and deploy cert-manager
Install:
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```

deploy:
```
kubectl -n cert-manager rollout status deploy/cert-manager
```
## General Commands
Create the redis shards as Stateful Sets.  
```
sudo kubectl apply -f ./redis-shards.yaml --namespace=kjyo
```
Create the Redis Service.
```
sudo kubectl apply -f ./redis-service.yaml --namespace=kjyo
```
Create the ConfigMap for the Ambassador
```
sudo kubectl create configmap twem-config --from-file=./nutcracker.yaml --namespace=kjyo
```
Create a shared storage volume.
```
sudo kubectl apply -f ./pv-volume.yaml --namespace=kjyo
```
Claim the storage.
```
sudo kubectl apply -f ./pv-claim.yaml  --namespace=kjyo
```
Describe the service.
```
sudo kubectl describe svc redis --namespace=kjyo
```
Describe the Pods
```
sudo kubectl get pods -o wide --namespace=kjyo

```
## Appendix K3d ##
Create a cluster, mapping the port 30080 from agent-0 to localhost:8082

k3d cluster create mycluster -p "8082:30080@agent[0]" --agents 2

Note 1: Kubernetes’ default NodePort range is 30000-32767
Note 2: You may as well expose the whole NodePort range from the very beginning, e.g. via k3d cluster create mycluster --agents 3 -p "30000-32767:30000-32767@server[0]" (See this video from @portainer)

Warning: Docker creates iptable entries and a new proxy process per port-mapping, so this may take a very long time or even freeze your system!
… (Steps 2 and 3 like above) …

## References
https://kubernetes.io/docs/concepts/services-networking/service/#nodeport\

k3d https://k3d.io/usage/guides/exposing_services/



Book:
Designing Distributed Systems -David Burns
