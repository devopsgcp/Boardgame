pipeline {
    agent any
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner' // 'sonar-scanner' configured as a tool
        PROJECT_ID = 'cicd-2024'
        REGION = 'asia-south2'
        REPO_NAME = 'boardgame'
        IMAGE_NAME = 'boardgame'
        IMAGE_TAG = 'latest'
        ARTIFACT_REGISTRY = "${REGION}-docker.pkg.dev"
        FULL_IMAGE = "${ARTIFACT_REGISTRY}/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${IMAGE_TAG}"
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
                dependencyCheck additionalArguments: '--scan .', odcInstallation: 'DC'
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

        stage ('push_to_nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: '', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: 'cicd-template', traceability: true) {
                     sh 'mvn deploy'
    
                }
            }

        }

        stage('Set up GCP Auth') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        echo "Activating service account..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $PROJECT_ID
                        gcloud auth configure-docker $ARTIFACT_REGISTRY --quiet
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $ARTIFACT_REGISTRY/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }
        stage('Trivy Scan (Docker Image)') {
          steps {
            sh '''
              echo "Scanning Docker image with Trivy..."
              trivy image --severity HIGH,CRITICAL --format json -o trivy-image-report.json $FULL_IMAGE
            '''
    }
}


        stage('Push to Artifact Registry') {
            steps {
                sh '''
                    docker push $ARTIFACT_REGISTRY/$PROJECT_ID/$REPO_NAME/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    sh '''
                        echo "Authenticating to GKE..."
                        gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
                        gcloud config set project $PROJECT_ID
                        export PATH=$PATH:/usr/bin:/usr/local/bin
                        export USE_GKE_GCLOUD_AUTH_PLUGIN=True
                        gcloud container clusters get-credentials boardgame --region=$REGION --project=$PROJECT_ID
                        echo "Deploying to GKE..."
                        kubectl apply -f k8s/boardgame-deployment.yaml
                        kubectl apply -f k8s/boardgame-service.yaml
                    '''
                }
            }
        }

    }
}