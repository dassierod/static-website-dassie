pipeline {
    agent { label 'docker-agent' }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality') {
            steps {
                sh '''
                    test -s index.html
                    test -s Dockerfile
                    grep -qi '<html' index.html
                '''
            }
        }

        stage('Build Docker') {
            steps {
                sh 'docker build -t static-website-dassie:ci .'
            }
        }

        stage('Tests') {
            steps {
                sh 'docker run --rm static-website-dassie:ci nginx -t'
            }
        }

        stage('Acceptance Tests') {
            steps {
                sh '''
                    docker rm -f dassie-acceptance 2>/dev/null || true

                    docker run -d \
                      --name dassie-acceptance \
                      static-website-dassie:ci

                    trap 'docker rm -f dassie-acceptance >/dev/null 2>&1 || true' EXIT

                    docker exec dassie-acceptance \
                      wget -q -O /dev/null http://127.0.0.1/

                    docker exec dassie-acceptance \
                      wget -q -O - http://127.0.0.1/ \
                      | grep -qi '<html'
                '''
            }
        }
stage('Artifact') {
    steps {
        sh '''
            mkdir -p artifacts

            echo "Application: static-website-dassie" > artifacts/build-info.txt
            echo "Build number: ${BUILD_NUMBER}" >> artifacts/build-info.txt
            echo "Git commit: ${GIT_COMMIT}" >> artifacts/build-info.txt
            echo "Docker image: static-website-dassie:ci" >> artifacts/build-info.txt

            docker image inspect static-website-dassie:ci \
                --format '{{.Id}}' > artifacts/image-id.txt
        '''

        archiveArtifacts artifacts: 'artifacts/*.txt', fingerprint: true
    }
}
    }
}
