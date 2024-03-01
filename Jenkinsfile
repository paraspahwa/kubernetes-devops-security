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
        stage('Sonarqube - SAST') {
            steps {
                withSonarQubeEnv('Sonarscanner') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=numeric-appication -Dsonar.host.url=http://3.109.144.253:9000"
                }
                timeout(time: 2, unit: 'MINUTES') {
                    script {
                        waitForQualityGate abortPipeline: true
                    }
                } 
            }
        }
        stage('Vulnerability Scan - Docker') {
            steps {
                sh "mvn dependency-check:check"
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


