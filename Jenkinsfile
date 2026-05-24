pipeline {
    agent {
        node {
            label 'prod'
        }
    }

    tools {
        maven 'mymaven'
    }

    stages {

        stage('Git-Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/saluvalasaichandu/3-Tier-Docker-Application.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=3-Tier-java-app'
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'cp -r target Docker-app'
            }
        }

        stage('Artifact-Uploader') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'vprofile',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war'
                    ]],
                    credentialsId: 'nexus-creds',
                    groupId: 'com.visualpathit',
                    nexusUrl: '3.90.163.133:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'myrepo',
                    version: 'v2'
                )
            }
        }

        stage('Docker-Image-Build') {
            steps {
                sh 'docker build -t saichandu27/automation:db Docker-db'
                sh 'docker build -t saichandu27/automation:app Docker-app'
            }
        }

        stage('Trivy-Image-Scan') {
            steps {
                sh 'trivy image saichandu27/automation:db'
                sh 'trivy image saichandu27/automation:app'
            }
        }

        stage('Docker-Push') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'registry-docker-hub-password'
                    ) {
                        sh 'docker push saichandu27/automation:db'
                        sh 'docker push saichandu27/automation:app'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker stack deploy -c compose.yml flm'
            }
        }
    }
}