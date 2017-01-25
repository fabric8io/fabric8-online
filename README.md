# Fabric8 online

This project generates the distribution of the [fabric8 online platform](https://fabric8.io/)

 <p align="center">
   <a href="http://fabric8.io/">
    <img src="https://raw.githubusercontent.com/fabric8io/fabric8/master/docs/images/cover/cover_small.png" alt="fabric8 logo"/>
   </a>
 </p>

TODO - add getting started docs and description of the different parts of the platform

To run fabric8 online locally we recommend using minishift although for now we have fixed to use version 0.9.0 until fabric8 has been verified to work on it.

Quickest way to get started is with `gofabric8`
http://fabric8.io/guide/getStarted/gofabric8.html#install-fabric8

Following the set up guild up until the step where you need to call `gofabric8 start` and instead:

Get the latest release tag from this repo, we will use 1.0.7 as an example until we automate updating these docs, then..
```
curl  http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online/1.0.7/fabric8-online-1.0.7-openshift.yml > fabric8-online-1.0.7-openshift.yml
gofabric8 start --minishift --package=fabric8-online-1.0.7-openshift.yml
```
That yaml contains the deployment configs, services, configmaps etc for the fabric8-online which can run on minishift locally.

One small manual step for now is you need to manually set the Che SCC, we'll sort this soon so you dont need to add it but once gofabric8 start has finished run..
```
oc scale --replicas=0 dc che
oc adm policy add-scc-to-user privileged -z che
oc scale --replicas=1 dc che
```
and the Che pod should start too.

Until the new fabric8-ui is added you can open the current fabric8-console with `gofabric8 console`
