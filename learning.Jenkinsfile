pipeline {
    agent {
        label 'iot-runner' // This matches the label you just gave to the built-in node
    }
    environment {
        // Podman resolves 'mosquitto' to the container IP automatically
        MQTT_BROKER = 'mosquitto'
    }

    stages {
        stage('1. Environment Setup') {
            steps {
                echo "Creating Virtual Environment and installing dependencies..." 
                git url: 'https://github.com/kannase/iot-testbed.git', branch: 'main' 
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install -r requirements.txt 
                ''' 
            }
        }

        stage('2. Integration Testing') {
            parallel { 
                stage('Device Simulation') {
                    steps {
                        echo "Starting Device Simulator..." 
                        timeout(time: 1, unit: 'MINUTES') { 
                            sh "./venv/bin/python3 simulator/iot_device_simulator.py --broker ${env.MQTT_BROKER}" 
                        }
                    }
                }
                stage('Pytest Logic') {
                    steps {
                        echo "Running Pytest..." 
                        sh "./venv/bin/python3 -m pytest tests/pytest --junitxml=results.xml" 
                    }
                }
            }
        }
    } // End of stages

    post { // This block is now correctly placed inside the pipeline
        always {
            echo "Generating Test Reports..." 
            junit 'results.xml' 
        }
    }
} // End of pipeline
