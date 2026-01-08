pipeline {
    agent any

    environment {
        // Using the internal Podman gateway to find the Mosquitto container
		DOCKER_HOST = 'tcp://host.containers.internal:2375'
        MQTT_BROKER = 'host.containers.internal'
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

        stage('2. Provision Broker') {
            steps {
			       // 1. Remove any old container with this name (even if it's already stopped)
                   sh 'podman rm -f mosquitto || true'
                   echo "Ensuring Mosquitto Broker is running..."
                
				  // 2. Start the new broker with the correct flags and full image path
                  sh 'podman run -d --name mosquitto -p 1883:1883 docker.io/library/eclipse-mosquitto mosquitto --allow-anonymous --listener 1883'
        
                  // 3. Give the broker a moment to initialize before the tests start
                  sh 'sleep 3'
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
                        sh "./venv/bin/python3 -m pytest tests/pytest --junitxml=results.xml"
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