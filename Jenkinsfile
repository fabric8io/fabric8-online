#!/usr/bin/groovy
@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def releaseVersion
deployOpenShiftTemplate(openshiftConfigSecretName: 'devshift-config'){
  mavenNode {
    ws{
      try {
        checkout scm
        readTrusted 'release.groovy'
        sh "git remote set-url origin git@github.com:fabric8io/fabric8-online.git"

        def pipeline = load 'release.groovy'

        stage 'Stage'
        def stagedProject = pipeline.stage()
        releaseVersion = stagedProject[1]

        stage 'Deploy to OpenShift'
        def prj = "cd-tenant"
        def user = "rhn-support-jrawling"
        echo "using project: ${prj} and user: ${user}"

        container(name: 'clients') {
          stage "Applying ${releaseVersion} updates"
          sh """

          # commmented out until we can create projects 
          #oc new-project ${prj}
          oc process -f https://oss.sonatype.org/content/repositories/staging/io/fabric8/online/packages/fabric8-online-team/${releaseVersion}/fabric8-online-team-${releaseVersion}-openshift.yml -v PROJECT_NAME=${prj}  -v PROJECT_ADMIN_USER=${user}  -v PROJECT_REQUESTING_USER=${user} | oc apply -f -

          echo "now populating the che namespace: ${prj}-che"
          oc project ${prj}-che
          oc apply -f https://oss.sonatype.org/content/repositories/staging/io/fabric8/online/packages/fabric8-online-che/${releaseVersion}/fabric8-online-che-${releaseVersion}-openshift.yml
                   
          echo "now populating the jenkins namespace: ${prj}-jenkins"
          oc project ${prj}-jenkins
          oc process -f https://oss.sonatype.org/content/repositories/staging/io/fabric8/online/packages/fabric8-online-jenkins/${releaseVersion}/fabric8-online-jenkins-${releaseVersion}-openshift.yml -v PROJECT_USER=${user} | oc apply -f -
                   

          """
          sh " -n ${prj}"

          sleep 20 // ok bad bad but there's a delay between DC's being applied and new pods being started.  lets find a better way to do this looking at teh new DC perhaps?

          waitUntil{
            // wait until the pods are running has been deleted
            try{
              sh "oc get pod -l project=jenkins-openshift,provider=fabric8 -n ${prj} | grep Running"
              sh "oc get pod -l project=content-repository,provider=fabric8 -n ${prj} | grep Running"
              sh "oc get pod -l project=che,provider=fabric8 -n ${prj} | grep Running"
              echo "Jenkins, Che and Content Repository pods Running for v ${releaseVersion}"
              return true
            } catch (err) {
              echo "waiting for Jenkins, Che and Content Repository to be ready..."
              return false
            }
          }
          def routes = sh(script: "oc get routes -n ${prj}", returnStdout: true).toString().trim()
          def msg = """${env.JOB_NAME} v${releaseVersion} Deployed and ready for QA:
          ${routes}
          """
          hubot room: 'release', message: msg

          stage "Trigger sample build"
          sh "oc start-build spring-boot-webmvc-jr --wait -n ${prj}"
            
        }

        stage 'Approve'
        pipeline.approve(stagedProject)

        stage 'Promote'
        pipeline.release(stagedProject)

        stage 'Tear down Test'
        sh """
          oc delete project ${prj} ${prj}-jenkins ${prj}-che ${prj}-test ${prj}-stage ${prj}-run 
        """
      } catch (err){
          hubot room: 'release', message: "${env.JOB_NAME} failed: ${err}"
          error "${err}"
      }
    }
  }
}
