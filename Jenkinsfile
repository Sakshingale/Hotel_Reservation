pipeline {
    agent any

    environment {
        VENV_DIR = 'venv'
        GCP_PROJECT = "gen-lang-client-0208573681"
        IMAGE_NAME = "ml-project"
        TAG = "latest"
    }

    stages {

        stage('Cloning Github repo to Jenkins') {
            steps {
                echo 'Cloning Github repo to Jenkins............'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        credentialsId: 'github-token',
                        url: 'https://github.com/Sakshingale/Hotel_Reservation.git'
                    ]]
                )
            }
        }

        stage('Setting up Virtual Environment & Installing dependencies') {
            steps {
                echo 'Setting up our Virtual Environment and Installing dependencies............'
                sh '''
                    python -m venv ${VENV_DIR}
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -e .
                '''
            }
        }

        stage('Build & Push Docker Image to GCR') {
            steps {
                withCredentials([file(credentialsId: 'gcp-key', variable: 'GCP_KEY')]) {
                    sh '''
                      docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v $PWD:/workspace \
                        -v $GCP_KEY:/key.json \
                        -w /workspace \
                        -e GOOGLE_APPLICATION_CREDENTIALS=/key.json \
                        google/cloud-sdk:slim \
                        sh -c "
                          gcloud auth activate-service-account --key-file=/key.json &&
                          gcloud config set project ${GCP_PROJECT} &&
                          gcloud auth configure-docker --quiet &&
                          docker build -t gcr.io/${GCP_PROJECT}/${IMAGE_NAME}:${TAG} . &&
                          docker push gcr.io/${GCP_PROJECT}/${IMAGE_NAME}:${TAG}
                        "
                    '''
                }
            }
        }
    }
}
