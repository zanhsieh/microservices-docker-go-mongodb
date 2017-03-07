Deliverable
===========
* This github repo
* Google Cloud Container Engine cluster page (http://104.199.209.96/#/deployment?namespace=_all)
* Travis build page (https://travis-ci.org/zanhsieh/microservices-docker-go-mongodb)
* GKE logging page (https://console.cloud.google.com/logs/viewer?_ga=1.229840984.519146318.1488762114&project=neon-notch-160406&minLogLevel=0&expandAll=false&resource=container%2Fcluster_name%2Fseekercap-cluster%2Fnamespace_id%2Fdev-hk&logName=projects%2Fneon-notch-160406%2Flogs%2Fcinema-bookings&timestamp=2017-03-07T06:01:28.000000000Z)

Highlight
=========

1/ Choose an application

Cinema

2/ Implement one or multiple DevOps tasks

a/ Run the whole application from source

In GKE cloud (see above URL), with some modification, such as VIRTUAL_HOST assignment (e.g. bookings.local -> bookings) since internal DNS cannot resolve *.local.

Implment ingress controller to replace nginx proxy.

Convert docker-compose.yml to Helm chart (https://github.com/kubernetes/helm).

```
cd k8s-charts
kubectl create ns dev-hk
helm init
helm install . --namespace dev-hk
helm install . --namespace uat-hk
helm install . --namespace prod-hk
```

Note that ingress controller in GKE must come with NodePort, and free trial only have 5 backends could be used (this project use 4, and plus the default 1, so quota exceeded).

b/ Run the whole application from images or other build artefacts

See .travis.yml. You need to generate your own docker hub login credential, kubernetes credentials, etc.

c/ Setup a log management solution and/or log analysis

GKE comes with fluentd integrated. To search log, go to GKE logging page above.

d/ Setup a monitoring dashboard

Kubernetes dashboard should be sufficient; in case you need long time archive of statistics, then please request Grafana + Prometheus.

Kubernetes dashboard (http://104.199.209.96/#/deployment?namespace=_all)

e/ Setup a DevOps tool to manage environments staging.

Travis build already capable to have dev / uat / prod via kubernets namespacing (e.g. dev-hl, uat-hk, prod-hk). Either from developer prospect to change .travis.yml environment variables (e.g. REGION, ENVIRONMENT) to have different build, or from system operator prospect that we could eliminate those just like GCLOUD_KEY as Travis environment variables.

If all requied docker image has been built, then we could take advance from .travis.yml:

```
# Promotion
docker pull zanhsieh/cinema-users:dev-hk-d680540
docker tag  zanhsieh/cinema-users:dev-hk-d680540 zanhsieh/cinema-users:uat-hk-d680540
docker push zanhsieh/cinema-users
kubectl patch deployment users -p '{"spec":{"template":{"spec":{"containers":[{"name":"'"cinema-users"'","image":"'"zanhsieh/cinema-users:uat-hk-d680540"'"}]}}}}' -n uat-hk
```

```
# Demotion
kubectl patch deployment users -p '{"spec":{"template":{"spec":{"containers":[{"name":"'"cinema-users"'","image":"'"zanhsieh/cinema-users:uat-hk-0a5ba1e"'"}]}}}}' -n uat-hk
```

```
# Rollback
kubectl rollout undo deployment/users -n prod-hk --to-revision=2
```

Helm chart base upgrade / rollback

```
# List available release
helm list

# Upgrade
cd to/where/helm/chart/located
helm upgrade <release_name> . --namespace dev-hk

# Rollback
helm rollback <release_name> <revision>
```