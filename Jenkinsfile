def imageName="ppaciekq/frontend"
def dockerTag=""
def dockerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    agent {
        label 'agent'
    }
 
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarScanner'
    }
 
 
    stages {
        stage('Get Code') {
            steps {
                checkout scm // Get some code from a GitHub repository
            }
        }
 
 
        stage('Unit tests') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
 
 
        stage('Run Sonarqube analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        } 

        stage('Build application image') {
            steps {
                script {
                  dockerTag = "RC-${env.BUILD_ID}"
                  applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }

        stage ('Pushing image to Artifactory') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        } 


    }

    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success {
            build job: 'app_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
        }
    }

}
 
