pipeline {
    agent any
    stages {
        stage('Setup & Checkout') {
            steps {
                // MUST checkout the code first!
                git url: 'https://github.com/kannase/iot-testbed.git', branch: 'main'
                sh 'python3 -m venv venv'
                sh './venv/bin/pip install paho-mqtt pytest'
            }
        }
        stage('Integration Test') {
            steps {
                script {
                    // Start simulator in background and save PID
                     sh "./venv/bin/python3 simulator/iot_device_simulator.py --broker mosquitto & echo \$! > sim.pid"
                    
                    try {
                        echo "Simulator started in background. Waiting for data..."
                        sleep 10
                        
                        // Run the tests
                        sh "./venv/bin/python3 -m pytest tests/pytest --junitxml=results.xml"
                        
                    } finally {
                        // Cleanup: Kill the process using the saved PID
                        echo "Cleaning up simulator..."
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