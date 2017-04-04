# Solution details

I'm running latest Kubernetes 1.6 cluster bootstrapped with latest Kops 1.6.0-alpha
on AWS in an existing VPC with both public and private subnets per each AZ.
Kubernetes master and worker nodes are created as ASGs in private subnets per each
of three AZs in the region making such setup secure and highly available.
And also making me proud of this thing :)

Custom docker images are used to implement the requirements. I've pushed them to
[DockerHub](https://hub.docker.com/r/adubeniuk/) for simplicity. Separate private Docker registry or ECS registry
should be used for private images in production.

Latest Weave Net 1.6 addon is used for networking and Weave Scope can be used
to visualize and interact with the nodes, pods, containers and their processes.

API service is based on [SpringBootRestApiSeed](https://github.com/thoersch/spring-boot-rest-api-seed) and connects to PostgreSQL via [DNS service name](https://kubernetes.io/docs/concepts/services-networking/service/#dns)
API service is exposed to the public by external AWS ELB forwarding to service NodePort
in private network.

PostgreSQL service has been installed from a [helm chart](https://github.com/kubernetes/charts/tree/master/stable/postgresql) with the required user/pass/db
config.
PGDATA is stored on a PersistentVolume and can be claimed by new pods.

Logger service is based on rsyslog and listens for the logs on tcp/514 only.

NetworkPolicies have been used to ensure that each machine can only connect to
the machine(s) specified in the network rules. There are two ingress network
policies for API and Logger services allowing traffic only from the allowed pods.
Also DefaultDeny ingress policy is set for all the pods in the default namespace.

Serverspec has been used to test the configuration. You can run it from Bastion
(jump server) however the tests are really simple as there's no Kubernetes support.

# Check API

Get user

```
http://check.mycluster.club/users/1
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

kubectl logs api-seed-97405333-30659 | grep GET | wc -l
    3748
```

# Access Weave Scope

Forward Weave Scope to your localhost

```
kubectl port-forward $(kubectl get pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}') 4040
```

http://localhost:4040

# Do it yourself

Setup AWS VPC, private and public subnets, NAT gateways and routing tables.
You will need your VPC ID and a private DNS hosted zone resolving the name of the cluster.

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

# Things to discuss

• using socat for log redirection (implementation, encryption, performance)

• using tools like serverspec to test docker containers on kubernetes
(lack of native support for kubernetes, overall simplicity of the tools)
