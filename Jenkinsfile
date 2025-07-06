pipeline {
    agent any
	tools {
	  maven : maven3
	  jdk : jdk17
	}
    stages {
        stage('clone repo') {
            steps {
                git branch: 'main', url: 'https://github.com/devopsgcp/Boardgame.git'
            }
        }
		
		stage('compile') {
            steps {
               sh 'mvn compile'
            }
        }
		stage('test') {
            steps {
               sh 'mvn test'
            }
        }
		stage('package') {
            steps {
               sh 'mvn package'
            }
        }
    }
}