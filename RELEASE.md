# fabric8 online release process

fabric8 online is made up from a number of projects.  We dont yet have a single end to end pipeline so until then these are the steps we need to perform, depending from which project you start.


## jenkins-openshift

__NOTE__ we currently have an issue where new plugin versions are not automatically applied during a rolling upgrade of Jenkins when PVs are used.  See https://github.com/fabric8io/fabric8-online/issues/80

We layer the fabric8 dependent plugins on top of the OpenShift Jenkins image using the s2i tool.  If one of the custom plugins are changed these are the steps to include it a new release.

__using jenkins-sync-plugin as an example:__

```
git clone git@github.com:fabric8io/jenkins-sync-plugin.git
cd jenkins-sync-plugin
git checkout job-to-bc
mvn clean install
cd ..
git clone git@github.com:fabric8io/openshift-jenkins-s2i-config.git
cd openshift-jenkins-s2i-config
cp ../jenkins-sync-plugin/target/openshift-sync.hpi plugins/openshift-sync.jpi
git commit -a -m 'new openshift sync plugin'
git push  origin master
```

- trigger http://jenkins.cd.k8s.fabric8.io/job/fabric8-cd/job/openshift-jenkins-s2i-config/job/master/

- you can see the new version here https://hub.docker.com/r/fabric8/jenkins-openshift/tags/

- trigger http://jenkins.cd.k8s.fabric8.io/job/fabric8-cd/job/fabric8-devops/
fabric8-devops will have already been updated with the new tag

- trigger http://jenkins.cd.k8s.fabric8.io/job/fabric8-cd/job/fabric8-online/job/master/
new fabric8-online yaml now includes the new jenkins 


