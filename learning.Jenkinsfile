pipeline {
    agent { label 'linux-worker' }

    environment {
        // Since we map Podman ports to the host, we use localhost/127.0.0.1
        MQTT_BROKER = "127.0.0.1"
        APP_NAME    = "IoT-MQTT-Testbed"
    }

    stages {
        stage('1. Environment Setup') {
            steps {
                echo "Installing Python dependencies from requirements.txt..."
                // Ensures your worker has paho-mqtt and pytest
                sh '''
            python3 -m venv venv
            ./venv/bin/pip install -r requirements.txt
        '''
            }
        }

        stage('2. Provision Broker') {
            steps {
                script {
                    echo "Starting Mosquitto Broker via Podman..."
                    // Stop and remove any existing broker to ensure a clean slate
                    sh "podman stop mqtt-broker || true"
                    sh "podman rm mqtt-broker || true"
                    
                    // Run the official Mosquitto image in the background
                    sh "podman run -d --name mqtt-broker -p 1883:1883 eclipse-mosquitto"
                    
                    // Give the broker 2 seconds to initialize
                    sh "sleep 2"
                }
            }
        }

        stage('3. Integration Testing') {
            parallel {
                stage('Device Simulation') {
                    steps {
                        echo "Starting IoT Device Simulator..."
                        // We run the simulator. If it's a loop, it runs until the test finishes
                        // Using timeout to prevent the pipeline from hanging forever
                        timeout(time: 5, unit: 'MINUTES') {
                            sh "./venv/bin/python3 iot_device_simulator.py --broker ${env.MQTT_BROKER}"
                        }
                    }
                }

                stage('Pytest Logic') {
                    steps {
                        script {
                            try {
                                echo "Running IoT Logic Validation..."
                                // Runs your pytest and generates the XML results file
                                sh './venv/bin/python3 -m pytest --junitxml=results.xml'
                            } catch (Exception e) {
                                echo "Tests failed: ${e.message}"
                                currentBuild.result = 'UNSTABLE'
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Generating Test Reports..."
            // This 'slurps' the results.xml into the Jenkins UI
            junit 'results.xml'

            echo "Cleaning up Podman containers..."
            sh "podman stop mqtt-broker || true"
            sh "podman rm mqtt-broker || true"
            
            // Wipe the workspace to keep the worker clean
            cleanWs()
        }
        success {
            echo "Pipeline Completed Successfully! MQTT data verified."
        }
        failure {
            echo "Pipeline Failed. Check the Console Output and JUnit report for details."
        }
    }
}