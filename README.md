# Fabric8 online

This project generates the distribution of the [fabric8 online platform](https://fabric8.io/)

 <p align="center">
   <a href="http://fabric8.io/">
    <img src="https://raw.githubusercontent.com/fabric8io/fabric8/master/docs/images/cover/cover_small.png" alt="fabric8 logo"/>
   </a>
 </p>

## What's included?

The distribution currently builds two packages

### fabric8-dsaas - SaaS tools 

  - fabric8-ui
  - planner
  - forge SaaS - not yet included
  - bayesian - not yet included
  - elasticsearch
  - kibana
  - exposecontroller
  - configmapcontroller

### fabric8-team - developer tools

  - jenkins
  - che
  - content repository

## Deploying

To deploy the developer tools get the latest release tag from this fabric8-online repo, we will use 1.0.27 as an example until we automate updating this ReadMe.  Then download the release YAML which contains the deployment / deployment configs. configmaps etc and start the VM pointing to the released fabric8-online resources.

```
export ONLINE_VERSION=1.0.28
```

### Remote deploy

To deploy fabric8 online to a remote cluster ensure your oc or kubectl client is connnected to your remote cluster.

#### Kubernetes

```
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-kubernetes.yml
```

#### OpenShift

```
oc new-project online-tennant
oc adm policy add-scc-to-user privileged -z che
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:online-tennant:che
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-openshift.yml
```

### Local deploy

To run fabric8 online locally we recommend using minikube or minishift

#### Minikube

```
kubectl apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-kubernetes.yml
kubectl apply -f http://central.maven.org/maven2/io/fabric8/devops/apps/exposecontroller/2.2.317/exposecontroller-2.2.317-kubernetes.yml
```
now use gofabric8 to change the PVCs to use the minishift VM host path to persist data

```
gofabric8 volumes
```
#### Minishift

```
oc new-project online-tennant
oc adm policy add-scc-to-user privileged -z che
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:online-tennant:che
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-openshift.yml
oc expose service che --hostname=che.$(minishift ip).nip.io
```
now use gofabric8 to change the PVCs to use the minishift VM host path to persist data

```
gofabric8 volumes
```

Get the URL to access Che on:
```
oc get route che
```