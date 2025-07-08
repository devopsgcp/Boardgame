// parametrize jenkinsfile  

pipeline {
    agent any
    tools {
        maven 'maven3'
        java 'jdk17'

    }

    stages {
        stage ('git clone') {
            steps {
                git branch: "${params.branch_name}", url: 'https://github.com/devopsgcp/Boardgame.git'
            }
        }

        stage ('git compile') {
            sh 'mvn compile'
        }
    }


}