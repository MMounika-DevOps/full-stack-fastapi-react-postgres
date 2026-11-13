pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker compose -p full-stack-fastapi-react-postgres --env-file /etc/fullstack-app.env build'
            }
        }

        stage('Deploy Application') {
            steps {
                sh 'docker compose -p full-stack-fastapi-react-postgres --env-file /etc/fullstack-app.env up -d --build --remove-orphans'
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                    docker compose -p full-stack-fastapi-react-postgres --env-file /etc/fullstack-app.env ps
                    docker inspect fullstack_postgres \
                      --format='PostgreSQL status: {{.State.Status}}'
                    docker inspect fullstack_backend \
                      --format='Backend status: {{.State.Status}}'
                    docker inspect fullstack_frontend \
                      --format='Frontend status: {{.State.Status}}'
                '''
            }
        }

        stage('Application Test') {
            steps {
                sh '''
                    set -eu
                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost/api/openapi.json \
                            -o /tmp/fullstack-openapi.json; then
                            echo "API test passed."
                            break
                        fi
                        if [ "$i" -eq 30 ]; then
                            echo "API test failed."
                            docker compose -p full-stack-fastapi-react-postgres --env-file /etc/fullstack-app.env logs --tail=100
                            exit 1
                        fi
                        sleep 5
                    done
                    test -s /tmp/fullstack-openapi.json
                    curl -fsS http://localhost/ -o /dev/null
                    echo "Frontend HTTP test passed."
                '''
            }
        }
    }

    post {
        success {
            echo 'Build, deployment, and HTTP checks completed successfully.'
        }
        failure {
            sh 'docker compose -p full-stack-fastapi-react-postgres --env-file /etc/fullstack-app.env logs --tail=100 || true'
        }
    }
}
