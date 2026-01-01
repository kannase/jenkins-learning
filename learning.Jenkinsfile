pipeline {
    agent {
        label 'worker-1'
    }

    environment {
        // Using the internal Podman gateway to find the Mosquitto container
        MQTT_BROKER = 'host.containers.internal'
    }

    stages {
        stage('1. Environment Setup') {
            steps {
                echo "Creating Virtual Environment and installing dependencies..."
                // Creates a local 'venv' folder and installs requirements [cite: 5, 6]
                sh '''
            python3 -m venv venv
            # Point to the file inside the iot-testbed folder
            ./venv/bin/pip install -r iot-testbed/requirements.txt
        '''
            }
        }

        stage('2. Provision Broker') {
            steps {
                echo "Ensuring Mosquitto Broker is running..."
                // This starts the broker container using the podman installed in your agent
                sh 'podman run -d --name mqtt-broker -p 1883:1883 eclipse-mosquitto || true'
            }
        }

        stage('3. Integration Testing') {
            parallel {
                stage('Device Simulation') {
                    steps {
                        echo "Starting Device Simulator from subfolder..."
                        // Added 1-minute timeout to stop the 'while True' loop eventually 
                        timeout(time: 1, unit: 'MINUTES') {
                            sh "./venv/bin/python3 simulator/iot_device_simulator.py --broker ${env.MQTT_BROKER}"
                        }
                    }
                }
                stage('Pytest Logic') {
                    steps {
                        echo "Running Pytest from subfolder..."
                        // Points to the exact folder: iot-testbed/tests/pytest 
                        sh "./venv/bin/python3 -m pytest iot-testbed/tests/pytest --junitxml=results.xml"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Generating Test Reports..."
            // Looks for the results.xml generated in the workspace [cite: 12]
            junit 'results.xml'
        }
        cleanup {
            echo "Cleaning up containers..."
            sh 'podman stop mqtt-broker || true'
            sh 'podman rm mqtt-broker || true'
        }
    }
}