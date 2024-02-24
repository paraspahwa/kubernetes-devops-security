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
            post {
                always {
                    JUnit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

    }
}   
