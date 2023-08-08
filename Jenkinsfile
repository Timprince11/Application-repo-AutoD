pipeline {
    agent any
    environment {
        DOCKER_USER = credentials('DockerUsername')
        DOCKER_PASSWORD = credentials('DockerPassword')
    }
    stages {
        stage('Code Analysis') {
            steps {
               withSonarQubeEnv('SonarQube') {
                  sh "mvn sonar:sonar"  
               }
            }   
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Build Artifact') {
            steps {
                sh 'mvn -f pom.xml clean package'
            }
        }
        stage('Log into Docker Repo') {
            steps {
               sh 'docker login --username $DOCKER_USER --password $DOCKER_PASSWORD' 
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t cloudhight/usteam-app:latest .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                sh 'docker push cloudhight/usteam-app:latest'
            }
        }
        stage('Deploy into Staging') {
            steps {
                sshagent (['ansible-key']) {
                    sh 'ssh -t -t ec2-user@10.0.2.247 -o strictHostKeyChecking=no "cd /etc/ansible && ansible-playbook Staging-MyPlaybook.yml"'
                }
            }
        }
        stage ('New deployment notification') {
            steps {
                slackSend channel: 'cloudhight', message: 'New deployment awaiting approval'
            }
        }
        stage('Deploy to Production') {
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sshagent (['ansible-key']) {
                    sh 'ssh -t -t ec2-user@10.0.2.247 -o strictHostKeyChecking=no "cd /etc/ansible && ansible-playbook -i prod-hosts Prod-MyPlaybook.yml"'
                }
            }
        }
        stage ('Notify for new realease') {
            steps {
                slackSend channel: 'cloudhight', message: 'New realease deployed into Production'
            }
        }
    }                   
}
