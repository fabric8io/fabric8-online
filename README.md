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

## Running

### Remote 

To deploy fabric8 online to a remote cluster ensure your oc client is connnected to your remote cluster.

### Local 

To run fabric8 online locally we recommend using minishift
```
minishift start --vm-driver=xhyve --memory=6096 --cpus=2
```
If you know the IP address your minishift VM will use (you can get this after the VM has started with `minishift ip`) you can switch to use the more reliable nip.io for magic DNS
```
minishift start --vm-driver=xhyve --memory=6096 --cpus=2 --routing-suffix=192.168.64.151.nip.io
```
## Deploy
The fabric8 online distribution is versioned and released to maven central.  Using the scripts and commands below we will deploy the latest version in a new project called openshift-tennant, change the `$PRJ_NAME` value if you want a different project:
```
export PRJ_NAME=online-tenant
export ONLINE_VERSION=$(curl -L http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/maven-metadata.xml | grep '<latest' | cut -f2 -d">"|cut -f1 -d"<")

oc new-project $PRJ_NAME
oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/$ONLINE_VERSION/fabric8-online-team-$ONLINE_VERSION-openshift.yml
```
for now we have to update the Che configmap to use the che hostname so we can connect to the workspace:
```
oc get route che
```
edit the configmap
```
oc edit cm che
```
and replace `hostname-http:` value with the Che external hostname from the previous step

update the `externalName:` value for the Bayesian external SaaS service endpoint:
```
oc edit svc bayesian-link
```
Until we figure out the right roles for the pielines to create and edit environments run:
```
oc new-project $PRJ_NAME-staging
oc new-project $PRJ_NAME-production
oc project $PRJ_NAME
oc adm policy add-role-to-user edit system:serviceaccount:$PRJ_NAME:jenkins --namespace $PRJ_NAME-staging
oc adm policy add-role-to-user view system:serviceaccount:$PRJ_NAME:jenkins --namespace $PRJ_NAME-staging
oc adm policy add-role-to-user edit system:serviceaccount:$PRJ_NAME:jenkins --namespace $PRJ_NAME-production
oc adm policy add-role-to-user view system:serviceaccount:$PRJ_NAME:jenkins --namespace $PRJ_NAME-production
```
__minishift ONLY__

now use gofabric8 to change the PVCs to use the minishift VM host path to persist data and set extra permissions for Che
```
oc login -u system:admin
oc adm policy add-scc-to-user privileged -z che
gofabric8 volumes
oc login -u developer
```
retry the Che deployment if it is in failed state without the SCC added:
```
oc deploy --retry che
```

### Wait for applications to run running
wait for the pods to start:
```
oc get pods -w
```
get the URLs to access an application:
```
oc get route
```

### Run an example quickstart:

Add a sample quickstart build config using the piepline strategy and start the build:
```
oc apply -f https://gist.githubusercontent.com/rawlingsj/eff84421af56b6825c6ff38c1646382e/raw/49bcf50b6872268665e9fe9279e8888a7b1ab8ab/spring-boot-webmvc-build-config.yml
oc start-build spring-boot-webmvc-jr
```
View the pipeline in Jenkins or the OpenShift console


## Creating envionments

__NOTE__ a user or serviceaccount requires the self-provisioner role to request new openshift projects (environments).

```
export ONLINE_VERSION=$(curl -L http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team-environments/maven-metadata.xml | grep '<latest' | cut -f2 -d">"|cut -f1 -d"<")

oc process -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team-environments/$ONLINE_VERSION/fabric8-online-team-environments-$ONLINE_VERSION-openshift.yml -v PROJECT_NAME=test -v PROJECT_DISPLAYNAME=tester -v PROJECT_DESCRIPTION=testprj -v PROJECT_ADMIN_USER=developer -v PROJECT_REQUESTING_USER=system:admin  | oc apply -f -
```


## Using Minishift

Here are the instructions for using [minishift](https://github.com/minishift/minishift)

#### clone and build it
```
git clone https://github.com/fabric8io/fabric8-online.git
cd fabric8-online
mvn install
```

#### create a user/team set of environments and services

```
oc login -u system:admin
cd packages/fabric8-online-team
oc process -f target/classes/META-INF/fabric8/openshift.yml -v PROJECT_NAME=myproject  -v PROJECT_ADMIN_USER=`oc whoami`  -v PROJECT_REQUESTING_USER=`oc whoami` | oc apply -f -

cd ../fabric8-online-jenkins
oc project myproject-jenkins
oc process -f target/classes/META-INF/fabric8/openshift.yml -v PROJECT_USER=`oc whoami` -v PROJECT_NAMESPACE=myproject | oc apply -f -
gofabric8 volumes

cd ../fabric8-online-che
oc project myproject-che                   e
oc apply -f target/classes/META-INF/fabric8/openshift.yml
gofabric8 volumes
```

#### setup the shared services

```
oc new-project fabric8-saas
cd ../fabric8-online-platform-minimal
oc process -f target/classes/META-INF/fabric8/openshift.yml -v NAMESPACE=fabric8-saas  | oc apply -f -
```

#### adding the roles so the developer can use the new projects

```bash
export PROJECT_NAME=myproject
oc adm policy add-role-to-user view developer --namespace $PROJECT_NAME
oc adm policy add-role-to-user edit developer --namespace $PROJECT_NAME
oc adm policy add-role-to-user view developer --namespace $PROJECT_NAME-jenkins
oc adm policy add-role-to-user edit developer --namespace $PROJECT_NAME-run
oc adm policy add-role-to-user view developer --namespace $PROJECT_NAME-run
oc adm policy add-role-to-user edit developer --namespace $PROJECT_NAME-test
oc adm policy add-role-to-user view developer --namespace $PROJECT_NAME-test
oc adm policy add-role-to-user edit developer --namespace $PROJECT_NAME-stage
oc adm policy add-role-to-user view developer --namespace $PROJECT_NAME-stage
```
