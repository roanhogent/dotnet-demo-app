pipeline {
    agent any

    tools {
        dotnetsdk 'dotnetsdk'
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                

                sh '''
            if docker container inspect todoapp >/dev/null 2>&1; then
                echo "Removing existing todoapp container..."
                docker rm -f todoapp
            else
                echo "No existing todoapp container found."
            fi
        '''

            }
        }

        stage('Start Database') {
            steps {
                sh '''
                    docker compose -f "$COMPOSE_FILE" up -d todoappdb
                '''
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore dotnet-demo.slnx'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    dotnet build dotnet-demo.slnx \
                        --configuration Release \
                        --no-restore
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    dotnet test dotnet-demo.slnx \
                        --configuration Release \
                        --no-build \
                        --logger "trx;LogFileName=test-results.trx" \
                        --results-directory TestResults
                '''
            }
        }

        stage('Start Application') {
            steps {
                sh '''
                    docker compose -f "$COMPOSE_FILE" \
                        up -d --build todoapp
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts(
                artifacts: 'TestResults/**/*.trx',
                allowEmptyArchive: true
            )
        }

        success {
            echo 'Build and tests completed successfully.'
            echo 'Application should be available on host port 8081.'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
            sh '''
                docker compose -f "$COMPOSE_FILE" logs \
                    --no-color --tail=100 || true
            '''
        }
    }
}
