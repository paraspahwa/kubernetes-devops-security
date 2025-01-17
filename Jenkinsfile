pipeline {
    agent any

    stages {
        stage('Build Artifact - Maven') {
            steps {
                sh "mvn clean package -Dskiptests=true"
                archive 'target/*.jar'
            }
        }
        stage('Unit Tests - JUnit and Jacoco') {
            steps {
                sh "mvn test"
            }
        }
        stage('Mutation tests - PIT') {
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
        }
        
        stage('Vulnerability Scan - Docker') {
            steps {
                parallel(
                    "Dependency Scan": {
                        sh "mvn dependency-check:check"
                    },
                    "Trivy Scan": {
                        sh "bash trivy-scan.sh"
                    }
                )
                }
             post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
             }
        }
        stage('Docker Build and Push') {
            steps {
                withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
                    sh 'printenv'
                    sh 'docker build -t paraspahwa/numeric-app:""$GIT_COMMIT"" .'
                    sh 'docker push paraspahwa/numeric-app:""$GIT_COMMIT""'
                }
            }
        }
        stage('Kubernetes Deployment - DEV') {
            steps {
                withKubeConfig([credentialsId: "kubeconfig"]) {
                    sh "sed -i 's#replace#paraspahwa/numeric-app:$GIT_COMMIT#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}   


