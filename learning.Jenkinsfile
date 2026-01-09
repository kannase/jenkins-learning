pipeline {
    agent any

    environment {
        // Global variables for the whole pipeline 
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
                // Remote control your laptop's Podman engine 
                sh 'podman --url tcp://host.containers.internal:2375 rm -f mosquitto || true'
                sh 'podman --url tcp://host.containers.internal:2375 run -d --name mosquitto -p 1883:1883 docker.io/library/eclipse-mosquitto mosquitto --allow-anonymous --listener 1883' 
                sh 'sleep 3'
            }
        }

        stage('3. Integration Testing') {
            parallel { 
                stage('Device Simulation') {
                    steps {
                        echo "Starting Device Simulator from subfolder..." 
                        timeout(time: 1, unit: 'MINUTES') { 
                            sh "./venv/bin/python3 simulator/iot_device_simulator.py --broker ${env.MQTT_BROKER}"
                        }
                    }
                }
                stage('Pytest Logic') {
                    steps {
                        echo "Running Pytest from subfolder..."
                        sh "./venv/bin/python3 -m pytest tests/pytest --junitxml=results.xml" 
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Generating Test Reports..."
            junit 'results.xml' 
        }
        cleanup {
            echo "Cleaning up containers..."
            sh 'podman --url tcp://host.containers.internal:2375 rm -f mosquitto || true' 
        }
    }
}