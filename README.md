# Check API

Get user

```
curl check.mycluster.club/users/1
```
Add user

```
curl -H "Content-Type: application/json" -X POST -d '{
  "firstName": "Linus",
  "lastName": "Torvalds",
  "emailAddress": "torvalds@transmeta.com",
  "profilePicture": "linus-torvalds.jpg"
}' http://check.mycluster.club/users
```

# Check cluster

```
kubectl cluster-info
Kubernetes master is running at https://api.mycluster.club
KubeDNS is running at https://api.mycluster.club/api/v1/proxy/namespaces/kube-system/services/kube-dns

kubectl get svc
NAME                  CLUSTER-IP       EXTERNAL-IP        PORT(S)        AGE
api-seed              100.67.103.172   ade6201e21925...   80:31442/TCP   55m
kubernetes            100.64.0.1       <none>             443/TCP        11h
logger                100.69.60.165    <none>             514/TCP        2h
postgres-postgresql   100.69.158.52    <none>             5432/TCP       10h
weave-scope-app       100.66.34.7      <none>             80/TCP         11h
```

# Forward Weave Scope

Forward Weave Scope

```
kubectl port-forward $(kubectl get pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}') 4040
```

http://localhost:4040


# Howto

Bootstrap a cluster on AWS using Kops:

```
aws configure
aws s3api create-bucket --bucket mycluster-state-store --region eu-west-1

export NAME=mycluster.club
export KOPS_STATE_STORE=s3://mycluster-state-store
export VPC_ID=<VPC_ID>
export DNS_ZONE_PRIVATE_ID=<ID_OF_PRIVATE_HOSTED_ZONE> 

kops create cluster --node-count 3 --zones eu-west-1a,eu-west-1b,eu-west-1c --master-zones eu-west-1a,eu-west-1b,eu-west-1c --dns-zone={$DNS_ZONE_PRIVATE_ID} --dns private --node-size t2.medium --master-size t2.medium --topology private --networking weave --vpc={$VPC_ID} --bastion --channel alpha {$NAME}

kops update cluster ${NAME} --yes
```

Setup PostgreSQL from a helm chart:

```
helm install --name postgres --set postgresUser=SpringBootUser,postgresPassword=SpringBootPass,postgresDatabase=SpringBootRestApi stable/postgresql
```

Create api-seed service:

```
kubectl create -f api-seed/deployment.yml 
kubectl create -f api-seed/svc.yml
```

Create logger service:

```
kubectl create -f logger/deployment.yml 
kubectl create -f logger/svc.yml
```

Setup DefaultDeny ingress policy:

```
kubectl create -f networkpolicy/defaultdeny.yml
```

Create Network Policies for each service:

```
kubectl create -f networkpolicy/db-ingress.yml
kubectl create -f networkpolicy/logger-ingress.yml
```
