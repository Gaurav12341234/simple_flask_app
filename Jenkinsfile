pipeline {
    agent any

    environment {
        VENV_PATH = 'venv'
        PORT = '8000'
    }

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git branch: 'main',
                    url: 'https://github.com/Gaurav12341234/simple_flask_app.git'
            }
        }

        stage('Set Up Python Environment') {
            steps {
                script {
                    // Create virtual environment and upgrade pip
                    sh 'python3 -m venv $VENV_PATH'
                    sh './$VENV_PATH/bin/python -m pip install --upgrade pip'
                    
                    // Install requirements
                    sh '''
                        source $VENV_PATH/bin/activate && \
                        pip install -r requirements.txt --verbose
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Activate virtual environment and run tests
                    sh 'source $VENV_PATH/bin/activate && python -m pytest'
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    // Create a shell script to run the server
                    sh '''
                        echo "#!/bin/bash" > start_server.sh
                        echo "export FLASK_APP=app.py" >> start_server.sh
                        echo "export FLASK_ENV=production" >> start_server.sh
                        echo "source $VENV_PATH/bin/activate" >> start_server.sh
                        echo "python -m flask run --host=127.0.0.1 --port=$PORT" >> start_server.sh
                        chmod +x start_server.sh
                    '''
                    
                    // Run the server in the background
                    sh './start_server.sh &'
                    
                    // Wait for the server to start
                    sh 'sleep 15'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Check if the application is running using curl
                    sh '''
                        if curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:$PORT | grep -q "200"; then
                            echo "Application is running successfully!"
                        else
                            echo "Failed to connect to application!"
                            exit 1
                        fi
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                script {
                    // Perform a health check using curl and jq
                    sh '''
                        response=$(curl -s http://127.0.0.1:$PORT/health)
                        status=$(echo $response | jq -r .status)
                        if [ "$status" == "healthy" ]; then
                            echo "Health check passed!"
                        else
                            echo "Health check failed: $status"
                            exit 1
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                // Clean up processes
                sh '''
                    pkill -f "python -m flask" || true
                '''
            }
        }
        failure {
            script {
                echo 'Pipeline failed! Checking virtual environment status...'
                sh '''
                    echo "Listing installed packages:"
                    source $VENV_PATH/bin/activate && pip list
                '''
            }
        }
    }
}
