// parametrize jenkinsfile  

pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'

    }

    stages {
        stage ('git clone') {
            steps {
                git branch: "${params.branch_name}", url: 'https://github.com/devopsgcp/Boardgame.git'
            }
        }

        stage ('git compile') {
            steps {
                sh 'mvn compile'

            }
            
        }

        stage ('git package') {
            steps {
                sh 'mvn package'
            }

        }
    }


}