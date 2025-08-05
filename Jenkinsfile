pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'   //   'sonar-scanner'  this is toll name which we have configured in tool section in Jenkins
        PROJECT_ID = 'cicd-2024 '
        CLUSTER_NAME = 'boardgame'
        CLUSTER_REGION = 'asia-south2 ' // e.g. us-central1
        GOOGLE_CREDENTIALS = credentials('gke-service-account')
        REPOSITORY = 'boardgame '
        IMAGE_NAME = 'boardgame'
        IMAGE_TAG = 'latest'
        FULL_IMAGE = "${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}:${IMAGE_TAG}"


    }

    stages {
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --format json -o trivy-scan-report.json .'
            }
        }

        stage('OS Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'DD'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Boardgame \
                        -Dsonar.projectName=Boardgame \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

       stage('Nexus_push') {
         steps {
           withMaven(globalMavenSettingsConfig: 'cicd-nexus', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
              sh 'mvn deploy'
           }
        }
     }

       stage('Authenticate with GCP') {
            steps {
                sh """
                echo "${GOOGLE_CREDENTIALS}" > gcp-key.json
                gcloud auth activate-service-account --key-file=cicd-2024-f0fba99f0bbd.json
                gcloud config set project ${PROJECT_ID}
                gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${CLUSTER_REGION}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('Push to Artifact Registry') {
            steps {
                sh "docker push ${FULL_IMAGE}"
            }
        }


/* 
        stage('Deploy to GKE') {
            steps {
                sh """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                """
            }
        }

*/
   }
}