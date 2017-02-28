#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def releaseVersion
deployOpenShiftTemplate{
  mavenNode {
    ws{
      checkout scm
      readTrusted 'release.groovy'
      sh "git remote set-url origin git@github.com:fabric8io/fabric8-online.git"

      def pipeline = load 'release.groovy'

      stage 'Stage'
      def stagedProject = pipeline.stage()
      releaseVersion = stagedProject[1]

      stage 'Deploy to openshift.io'
      
      def token = readFile file: "/root/.oc/token"
      def prj = 'f8'

      container(name: 'clients') {
        sh "oc login ${config.url} --token=${token} --insecure-skip-tls-verify=true"

        stage "Applying ${releaseVersion} updates"
        sh "oc apply -f oc apply -f http://central.maven.org/maven2/io/fabric8/online/packages/fabric8-online-team/${releaseVersion}/fabric8-online-team-${releaseVersion}-openshift.yml"

        waitUntil{
          // wait until the pods are running has been deleted
          try{
            //sh "oc get pods -l provider=fabric8 | cut -f 1 -d ' ' | grep fabric8-test"
            sh "oc get pods \$(oc get pods | grep jenkins | cut -f 1 -d ' ') | grep Running"
            sh "oc get pods \$(oc get pods | grep content-repo | cut -f 1 -d ' ') | grep Running"
            echo "Jenkins and Content Repository pods Running for v ${releaseVersion}"
            return true
          } catch (err) {
            echo "waiting for Jenkins and Content Repository to be ready..."
            return false
          }
        }
        def routes = sh(script: 'oc get routes', returnStdout: true).toString().trim()
        echo """Applications are accessible on: 
        ${routes}
        """
        stage "Trigger sample build"
        sh "oc start-build spring-boot-webmvc-jr --wait"
          
      }

      stage 'Approve'
      pipeline.approve(stagedProject)

      stage 'Promote'
      pipeline.release(stagedProject)
    }
  }
}
