# Fabric8 online

This project generates the distribution of the [fabric8 online platform](https://fabric8.io/)

 <p align="center">
   <a href="http://fabric8.io/">
    <img src="https://raw.githubusercontent.com/fabric8io/fabric8/master/docs/images/cover/cover_small.png" alt="fabric8 logo"/>
   </a>
 </p>

## Deploying

Quickest way to get started is with `gofabric8`
http://fabric8.io/guide/getStarted/gofabric8.html#install-fabric8

Following the set up guild up until the step where you need to call `gofabric8 start` and instead:

Get the latest release tag from this fabric8-online repo, we will use 1.0.7 as an example until we automate updating this ReadMe.  Then download the release YAML which contains the deployment / deployment configs. configmaps etc and start the VM pointing to the released fabric8-online resources.

```
export ONLINE_VERSION=1.0.7
```

### Remote deploy

To deploy fabric8 online to a remote cluster ensure your oc or kubectl client is connnected to your remote cluster.

#### Kubernetes

```
curl  http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online/$ONLINE_VERSION/fabric8-online-$ONLINE_VERSION-kubernetes.yml > fabric8-online-$ONLINE_VERSION-kubernetes.yml
gofabric8 deploy --package=fabric8-online-$ONLINE_VERSION-kubernetes.yml
```

#### OpenShift

```
curl  http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online/1.0.7/fabric8-online-$ONLINE_VERSION-openshift.yml > fabric8-online-$ONLINE_VERSION-openshift.yml
gofabric8 deploy --package=fabric8-online-$ONLINE_VERSION-openshift.yml -d example.domain.io
```
One small manual step for now is you need to manually set the Che SCC, we'll sort this soon so you dont need to add it but once gofabric8 start has finished run..
```
oc scale --replicas=0 dc che
oc adm policy add-scc-to-user privileged -z che
oc scale --replicas=1 dc che
```
and the Che pod should start too.

### Local deploy

To run fabric8 online locally we recommend using minikube or minishift (v0.9.0)

#### Minikube

```
curl  http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online/$ONLINE_VERSION/fabric8-online-$ONLINE_VERSION-kubernetes.yml > fabric8-online-$ONLINE_VERSION-kubernetes.yml
gofabric8 start --package=fabric8-online-$ONLINE_VERSION-kubernetes.yml
```

#### Minishift

```
curl  http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online/1.0.7/fabric8-online-$ONLINE_VERSION-openshift.yml > fabric8-online-$ONLINE_VERSION-openshift.yml
gofabric8 start --minishift --package=fabric8-online-$ONLINE_VERSION-openshift.yml
```
One small manual step for now is you need to manually set the Che SCC, we'll sort this soon so you dont need to add it but once gofabric8 start has finished run..
```
oc scale --replicas=0 dc che
oc adm policy add-scc-to-user privileged -z che
oc scale --replicas=1 dc che
```
and the Che pod should start too.

##### Console

Until the new fabric8-ui is added you can open the current fabric8-console with `gofabric8 console`
