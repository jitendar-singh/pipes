#!/usr/bin/env groovy

import java.text.SimpleDateFormat

pipeline {
  agent none
  options { timeout(time: 20, unit: 'MINUTES') }
  stages {
        stage('initialisation') {
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject() {
                            echo "Using project: ${openshift.project()}"
                        }
                    }
                }
            }
        }
  } 
}


