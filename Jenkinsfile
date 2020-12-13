#!/usr/bin/env groovy
import java.text.SimpleDateFormat
podTemplate( name: 'openshift', cloud: 'openshift', label: 'openshift-agents', showRawYaml: false, envVars: [
    envVar(key: 'PATH', value: '/opt/rh/rh-nodejs10/root/usr/bin:/opt/rh/rh-maven35/root/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'),
    envVar(key: 'MAVEN_ARGS_APPENDX', value: '-Dcom.redhat.xpaas.repo.jbossorg')],
    containers: [
    containerTemplate(name: 'maven', image: 'registry.redhat.io/openshift4/ose-jenkins-agent-maven', ttyEnabled: true, command: 'cat', workingDir: '/tmp'),
    containerTemplate(name: 'nodejs', image: 'registry.redhat.io/openshift4/jenkins-agent-nodejs-10-rhel7', ttyEnabled: true, command: 'cat', workingDir: '/tmp')
    ]) {
  pipeline {
    agent { node { label 'nodejs' } }
    stages {
      stage('raw') {
        steps {
          node('nodejs') {
            script('Build a simple nodejs app') {
              sh """
                git clone https://github.com/akram/simple-nodejs-ex.git
                cd simple-nodejs-ex
                npm install
                """
            }
            script('Build nodejs s2i image') {
              openshift.withCluster() {
                try {
                  def created = openshift.newApp( 'nodejs~https://github.com/akram/simple-nodejs-ex.git' )
                    echo "new-app created ${created.count()} objects named: ${created.names()}"
                } catch ( e ) {
                  "Error encountered: ${e}"
                }
                //openshift.raw( "new-app nodejs~https://github.com/akram/simple-nodejs-ex.git " )
                def bc = openshift.selector( "bc", "simple-nodejs-ex");
                def buildSelector = bc.startBuild("--wait");
                openshift.withProject() {
                  currentProject = openshift.project();
                  def project = "test-" + new SimpleDateFormat("yyyy-MM-dd-HHmmss").format(new Date());
                  echo "To allow jenkins to create projects from a pipeline, the following command must be run";
                  echo "oc adm policy add-cluster-role-to-user self-provisioner system:serviceaccount:$currentProject:jenkins";
                  openshift.raw( "new-project $project" );
                  // echo "Context project is $openshift.project()"
                  // Project context has been set to the pipeline project
                  currentProject = project;
                  echo "openshift.raw() commands will specify $currentProject as project";
                  echo "end";
                }
              }
            }
          }

          script('Build, Install, deploy Maven project') {
            node('maven') {
              //git url: 'https://github.com/akram/simple-java-ex.git'
              sh """
                git clone https://github.com/akram/simple-java-ex.git
                cd simple-java-ex
                mvn -B test
                """
                script('Build JBoss EAP s2i image') {
                  openshift.withCluster() {
                    try {
                      openshift.raw( "new-app --build-env=MAVEN_ARGS_APPEND=-Dcom.redhat.xpaas.repo.jbossorg jboss-eap73-openshift:7.3~https://github.com/akram/simple-java-ex.git " );
                      def created = openshift.newApp( '--build-env=MAVEN_ARGS_APPEND=-Dcom.redhat.xpaas.repo.jbossorg jboss-eap73-openshift:7.3~https://github.com/akram/simple-java-ex.git' );
                      echo "new-app created ${created.count()} objects named: ${created.names()}";
                    } catch ( e ) {
                      echo "Error encountered: ${e}";
                    }
                    //openshift.raw( "new-app nodejs~https://github.com/akram/simple-nodejs-ex.git " )
                    //openshift.raw( "new-app --build-env=MAVEN_ARGS_APPEND=-Dcom.redhat.xpaas.repo.jbossorg jboss-eap73-openshift:7.3~https://github.com/akram/simple-java-ex.git " )
                  }
                }
              script('Build OpenShift Image') {
                echo "Building OpenShift container image example"
                  script {
                    openshift.withCluster() {
                      openshift.withProject() {
                        openshift.selector("bc", "simple-java-ex").startBuild("--follow=true");
                      }
                    }
                  }
              }
            }
          }
        }
      }
    }
  }
}

