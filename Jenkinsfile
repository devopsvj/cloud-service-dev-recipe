pipeline {
    agent { 
        kubernetes{
            label 'jenkins-slave'
        }
        
    }
    options {
        skipStagesAfterUnstable()
    }
    environment{
        DOCKER_USERNAME = credentials('DOCKER_USERNAME')
        DOCKER_PASSWORD = credentials('DOCKER_PASSWORD')
    }
    stages {
        stage('docker login') {
            steps{
                sh(script: """
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                """, returnStdout: true) 
            }
        }

        stage('git clone') {
            steps{
                sh(script: """
                    git clone https://github.com/manug2/cloud-service-dev-recipe.git
                """, returnStdout: true) 
            }
        }

        stage('python UI docker build') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/cloud-service-dev-recipe/python
                docker build . --network host -t manug2018/python:${BUILD_NUMBER}
                '''
            }
        }

        stage('python UI docker push') {
            steps{
                sh(script: """
                    docker push manug2018/python:${BUILD_NUMBER}
                """)
            }
        }

        stage('calc-service build') {
            steps{
                withMaven(
                    // Maven installation declared in the Jenkins "Global Tool Configuration"
                    maven: 'Maven_3.6.3', // (1)
                    // Use `$WORKSPACE/.repository` for local repository folder to avoid shared repositories
                    mavenLocalRepo: '.repository', // (2)
                    // Maven settings.xml file defined with the Jenkins Config File Provider Plugin
                    // We recommend to define Maven settings.xml globally at the folder level using
                    // navigating to the folder configuration in the section "Pipeline Maven Configuration / Override global Maven configuration"
                    // or globally to the entire master navigating to  "Manage Jenkins / Global Tools Configuration"
                    // mavenSettingsConfig: 'my-maven-settings' // (3)
                ){
                  sh script: '''
                  #!/bin/bash
                  cd $WORKSPACE/cloud-service-dev-recipe/calc-service
                  mvn clean install
                  '''
                }
            }
        }

        stage('calc-service docker build') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/cloud-service-dev-recipe/calc-service
                docker build . --network host -t manug2018/calc-service:${BUILD_NUMBER}
                '''
            }
        }

        stage('calc-service docker push') {
            steps{
                sh(script: """
                    docker push manug2018/calc-service:${BUILD_NUMBER}
                """)
            }
        }
        stage('deploy') {
            steps{
                sh script: '''
                #!/bin/bash
                cd $WORKSPACE/cloud-service-dev-recipe/
                #get kubectl for this demo
                curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
                chmod +x ./kubectl
                ./kubectl -n jenkins apply -f ./deploy/configmap.yaml
                ./kubectl -n jenkins apply -f ./deploy/secret.yaml
                ./kubectl -n jenkins apply -f ./deploy/service-ui.yaml
                cat ./deploy/deployment-ui.yaml | sed s/1.0.0/${BUILD_NUMBER}/g | ./kubectl apply -f -
                ./kubectl -n jenkins apply -f ./deploy/service-calc.yaml
                cat ./deploy/deployment-calc.yaml | sed s/1.0.0/${BUILD_NUMBER}/g | ./kubectl apply -f -
                '''
        }
    }
}
}