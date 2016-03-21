# Run Mesos in GCE

## Project prepare

Create your GCP project and export to environment.

```
export PROJECT_ID=sunny-573
```

## Set up firewall rule

```
gcloud compute firewall-rules delete default-swarm

gcloud compute firewall-rules create default-swarm \
  --allow tcp:2376,tcp:3376,tcp:8500,tcp:5050,tcp:8080 \
  --source-range 0.0.0.0/0
```

## Create Consoul

Here use consoul as cluster store, and we create one server as consoul host.

```
docker-machine create --driver google --google-project $PROJECT_ID \
  --google-zone asia-east1-c \
  --google-machine-type f1-micro mh-keystore

docker $(docker-machine config mh-keystore) run \
  -d -p "8500:8500" \
  -h "consul" progrium/consul \
  -server -bootstrap
```

## Swarm Cluster

Create the swarm cluster master node:

```
docker-machine create \
  --driver google \
  --google-project $PROJECT_ID \
  --google-zone asia-east1-c \
  --google-machine-type n1-standard-1 \
  --swarm --swarm-master \
  --swarm-discovery="consul://$(docker-machine ip mh-keystore):8500" \
  --engine-opt="cluster-store=consul://$(docker-machine ip mh-keystore):8500" \
  --engine-opt="cluster-advertise=eth0:2376" \
  mhs-demo0
```


## Create swarm slave

Create the swarm cluster slave node.

```
docker-machine create \
  --driver google \
  --google-project $PROJECT_ID \
  --google-zone asia-east1-c \
  --google-machine-type n1-standard-1 \
  --swarm \
  --swarm-discovery="consul://$(docker-machine ip mh-keystore):8500" \
  --engine-opt="cluster-store=consul://$(docker-machine ip mh-keystore):8500" \
  --engine-opt="cluster-advertise=eth0:2376" \
  mhs-demo1
```

## Run the mesos/marathon start compose

Run the compose file to group start mesos server and marathon server.

```
docker-compose -f mesos-marathon-swarm.yml up -d
```






