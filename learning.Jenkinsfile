pipeline {
    agent any

    environment {
        // Using the internal Podman gateway to find the Mosquitto container
		DOCKER_HOST = 'tcp://192.168.1.206:2375'
        MQTT_BROKER = '192.168.1.206'
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
		     environment {
                        // Force Podman to talk to your laptop's service instead of trying to run locally
                        DOCKER_HOST = "tcp://192.168.1.206:2375" 
    }
            steps {
                     // Use the --url flag to be 100% sure it hits your laptop
                     sh 'podman --url tcp://192.168.1.206:2375 rm -f mosquitto || true'
                     sh 'podman --url tcp://192.168.1.206:2375 run -d --name mosquitto -p 1883:1883 docker.io/library/eclipse-mosquitto mosquitto --allow-anonymous --listener 1883'
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
           sh 'podman --url tcp://192.168.1.206:2375 rm -f mosquitto || true'
        }
    }
}pipeline {
    agent any

    environment {
        // Using the internal Podman gateway to find the Mosquitto container
		DOCKER_HOST = 'tcp://192.168.1.206:2375'
        MQTT_BROKER = '192.168.1.206'
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
		     environment {
                        // Force Podman to talk to your laptop's service instead of trying to run locally
                        DOCKER_HOST = "tcp://192.168.1.206:2375" 
    }
            steps {
                     // Use the --url flag to be 100% sure it hits your laptop
                     sh 'podman --url tcp://192.168.1.206:2375 rm -f mosquitto || true'
                     sh 'podman --url tcp://192.168.1.206:2375 run -d --name mosquitto -p 1883:1883 docker.io/library/eclipse-mosquitto mosquitto --allow-anonymous --listener 1883'
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
           sh 'podman --url tcp://192.168.1.206:2375 rm -f mosquitto || true'
        }
    }
}