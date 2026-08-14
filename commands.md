Resource Type	Use Case	Example Command
Create Object	Create from a definition file	kubectl create -f <filename>
View ReplicaSets/RC	List replication controllers	kubectl get replicaset or kubectl get replicationcontroller
Delete ReplicaSet/RC	Remove a replication controller	kubectl delete replicaset <replicaset-name>
Update Definition	Replace object using YAML file	kubectl replace -f <filename>
Scale ReplicaSet/RC	Change number of replicas	kubectl scale --replicas=<number> -f <filename

kubectl create -f deployment-definition.yml

kubectl get deployments
kubectl get replicasets
kubectl get pods

kubectl get pods --namespace=kube-system
 kubectl create -f pod-definition.yml --namespace=dev
 kubectl create namespace dev

kubectl get pods --all-namespaces