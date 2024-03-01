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
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-appication -Dsonar.projectName='numeric-appication' -Dsonar.host.url=http://3.109.144.253:9000 -Dsonar.login=sqp_7bf09e9d1e3847379ce00a717586bff07f480e35"
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


