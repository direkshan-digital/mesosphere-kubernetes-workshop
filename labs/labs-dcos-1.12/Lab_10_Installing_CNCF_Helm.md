# Lab 10 - The DC/OS + Kubernetes + Helm - cool beer demo

![Model](https://github.com/dcos/demos/raw/master/dcos-k8s-beer-demo/1.10/images/components.png)

Are you wondering how [Java](http://www.oracle.com/technetwork/java/index.html), [Spring Boot](https://projects.spring.io/spring-boot/), [MySQL](https://www.mysql.com), [Neo4j](https://neo4j.com), [Apache Zeppelin](https://zeppelin.apache.org/), [Apache Spark](https://spark.apache.org/), [Elasticsearch](https://www.elastic.co),  [Apache Mesos](https://mesos.apache.org/), [DC/OS](https://dcos.io), [Kubernetes](https://kubernetes.io/) and [Helm](https://helm.sh) can all fit in one demo? Well, we'll show you! This is a cool demo, so grab your favourite beer and enjoy. 🍺


### Prerequisite

This lab assumes you have a Kubernetes cluster and kubectl with API Server access from the previous labs. 

### Helm setup

To deploy `beer-service` we will need [Helm](https://helm.sh).

To download and install Helm cli run:
```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
```

Then we need to install Helm's server side `Tiller`:
```bash
helm init
```

Once Tiller is installed, running `helm version` should show you both the client and server version:
```bash
helm version
Client: &version.Version{SemVer:"v2.8.0", GitCommit:"14af25f1de6832228539259b821949d20069a222", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.8.0", GitCommit:"14af25f1de6832228539259b821949d20069a222", GitTreeState:"clean"}
```

Now we need to add DC/OS Labs Helm Charts repo (where this demo Helm charts are published) to Helm:
```bash
helm repo add dlc https://dcos-labs.github.io/charts/
helm repo update
```

## Deploy Backend on DC/OS

Let's deploy Backend using the `dcos` cli:
```bash
dcos marathon group add https://raw.githubusercontent.com/dcos/demos/master/dcos-k8s-beer-demo/1.10/marathon-apps/marathon-configuration.json
```

Wait till it gets installed, you can check it's progress in DC/OS Dashboard/Services/beer.

## Deploy Frontend App on Kubernetes and expose it locally

### Frontend App

To deploy Frontend App [chart](https://github.com/dcos-labs/charts/tree/master/stable/beer-service-web) run:
```bash
helm install --name beer --namespace beer dlc/beer-service-web
```

Check that pods are running:
```bash
kubectl -n beer get pods
NAME                                                    READY     STATUS    RESTARTS   AGE
beer-beer-service-web-1676235277-s9ptv                  1/1       Running   0          2m
beer-beer-service-web-1676235277-tskrp                  1/1       Running   0          2m
```

### Accessing Frontend App locally

To access app locally run: (replace beer-beer-service-web-1676235277-tskrp with the pod name from the previous command)
```bash
kubectl port-forward -n beer beer-beer-service-web-1676235277-tskrp 8080
```

Now you should be able to check beer at `http://127.0.0.1:8080`

## Deploy Frontend App and expose it via Cloudflare Warp on Kubernetes

### Frontend App

To deploy Frontend App [chart](https://github.com/dcos-labs/charts/tree/master/stable/beer-service-web) (do not forget to replace there with your `domain_name`) run:
```bash
helm upgrade --install beer --namespace beer dlc/beer-service-web \
  --set ingress.enabled="true",ingress.host="beer.mydomain.com"
```

Check that pods are running:
```bash
kubectl -n beer get pods
NAME                                                    READY     STATUS    RESTARTS   AGE
beer-beer-service-web-854fb8dc65-76bk4                  2/2       Running   0          2m
beer-beer-service-web-854fb8dc65-d8tm9                  2/2       Running   0          2m
```

### Cloudflare Warp

The Cloudflare Warp Ingress Controller makes connections between a Kubernetes service and the Cloudflare edge, exposing an application in your cluster to the internet at a hostname of your choice. A quick description of the details can be found at https://warp.cloudflare.com/quickstart/.

**Note:** Before installing Cloudflare Warp you need to obtain Cloudflare credentials for your domain zone.
The credentials are obtained by logging in to https://www.cloudflare.com/a/warp, selecting the zone where you will be publishing your services, and saving the file locally to `dcos-k8s-beer-demo` folder.

To deploy Cloudflare Warp Ingress Controller [chart](https://github.com/dcos-labs/charts/tree/master/stable/cloudflare-warp-ingress) run:
```bash
helm install --name beer-ingress --namespace beer dlc/cloudflare-warp-ingress --set cert=$(cat cloudflare-warp.pem | base64)
```

Check that pods are running:
```bash
kubectl -n beer get pods
NAME                                                    READY     STATUS    RESTARTS   AGE
beer-beer-service-web-57f9bc955c-c9v4q                  2/2       Running   0          3m
beer-beer-service-web-57f9bc955c-k4mps                  2/2       Running   0          3m
beer-ingress-cloudflare-warp-ingress-775b5965fd-qd4rk   1/1       Running   0          16s
```

### Testing external access

Now you should be able to check beer at https://beer.mydomain.com/
And if you noticed Cloudflare Warp creates `https` connection by default :-)

## Conclusion

Cheers 🍺
