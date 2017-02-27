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
export ONLINE_VERSION=1.0.30
```

### Remote deploy

To deploy fabric8 online to a remote cluster ensure your oc or kubectl client is connnected to your remote cluster.

#### OpenShift

```
oc new-project online-tennant
oc adm policy add-scc-to-user privileged -z che
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:online-tennant:che
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-openshift.yml
```

### Local deploy

To run fabric8 online locally we recommend using minishift

#### Minishift

```
export ONLINE_VERSION=1.0.30
minishift start --vm-driver=xhyve --memory=6096 --cpus=2
oc new-project online-tennant
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-openshift.yml
```
For now we have to update the Che configmap to use the che hostname so we can connect to the workspace:
```
oc get route Che
```
edit the configmap
```
oc edit cm che
```
and replace `hostname-http:` value with the Che external hostname from the previous step

now use gofabric8 to change the PVCs to use the minishift VM host path to persist data and set extra permissions for Che
```
oc login -u system:admin
oc adm policy add-scc-to-user privileged -z che
gofabric8 volumes
oc login -u developer
```
Get the URLs to access an application:
```
oc get route
```
