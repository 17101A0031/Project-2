pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        NVD_API_KEY = credentials('nvd-api-key')  // Jenkins secret text credential
    }

    tools {
        maven 'maven3'
        jdk 'jdk-17'
    }

    stages {
        stage('git checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/17101A0031/Project-2.git'
            }
        }

        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('unit tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "${env.SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=EKART \
                        -Dsonar.projectName=EKART \
                        -Dsonar.java.binaries=target/classes"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                  withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_API_KEY')]) {
                    dependencyCheck additionalArguments: "--nvdApiKey=$NVD_API_KEY",
                                    odcInstallation: 'DC'
             }
        }
        }

        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }

         stage('deploy to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'b1088c4c-08d9-4504-8871-5a1cf7fc8717',
                          jdk: 'jdk-17',
                          maven: 'maven3') {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        

        stage('build and Tag docker image') {
            steps {
                script {
                        sh "docker build -t sidk27/ekart:latest -f docker/Dockerfile ."
                    }
            }
        }

        stage('Push image to Hub'){
            steps{
                script{
                   withCredentials([string(credentialsId: 'dockerhub-pwd', variable: 'dockerhubpwd')]) {
                   sh 'docker login -u sidk27 -p ${dockerhubpwd}'}
                   sh 'docker push sidk27/ekart:latest'
                }
            }
        }
        stage('EKS and Kubectl configuration'){
            steps{
                script{
                    sh 'aws eks update-kubeconfig --region ap-south-1 --name project-cluster'
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                script{
                    sh 'kubectl apply -f deploymentservice.yml'
                }
            }
        }
    }

}
