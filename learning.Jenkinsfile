pipeline {
    agent any
    stages {
        stage('Setup') {
            steps {
                sh 'python3 -m venv venv'
                sh './venv/bin/pip install paho-mqtt pytest'
            }
        }
        stage('Integration Test') {
            steps {
                script {
                    // 1. Start simulator in background and save its Process ID (PID)
                    // The '&' is the magic that makes it run in the background
                    sh "python3 simulator/iot_device_simulator.py --broker mosquitto & echo \$! > sim.pid"
                    
                    try {
                        echo "Simulator started in background. Waiting for data..."
                        sleep 10
                        
                        // 2. Run the tests
                        sh "./venv/bin/python3 -m pytest tests/pytest --junitxml=results.xml"
                        
                    } finally {
                        // 3. THE CLEANUP: This runs even if tests fail!
                        echo "Killing the simulator process..."
                        sh "kill \$(cat sim.pid) || true"
                        sh "rm sim.pid"
                    }
                }
            }
        }
    }
    post {
        always {
            junit 'results.xml'
        }
    }
}