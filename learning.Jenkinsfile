// Methods are defined at the very top or very bottom of the file
def checkStatus(String component, int statusCode) {
    if (statusCode == 200) {
        return "${component} is HEALTHY"
    } else {
        return "${component} is FAILING"
    }
}

pipeline {
    agent { label 'linux-worker' } 
    
    // --- NEW: Global Environment Variables ---
    environment {
        APP_NAME    = "Groovy-Lab"
        API_VERSION = "v2.4.0"
        SECRET_KEY  = "super-secret-123" // In real life, use Credentials for this!
        MY_TOKEN = credentials('my-api-token')
    }
    
    parameters {
        choice(name: 'TARGET_ENV', choices: ['staging', 'production'], description: 'Which env to look up?')
        booleanParam(name: 'RUN_DIAGNOSTICS', defaultValue: true, description: 'Check system health?')
    }
    
    stages {
        stage('Parallel Lessons') {
            parallel {
                stage('Lesson 1: Loops') {
                    steps {
                        script {
                            def tools = ['Git', 'Podman', 'Groovy']
                            tools.each { echo "Learning: ${it}" }
                        }
                    }
                }
                stage('Lesson 4: Groovy Methods') {
                    steps {
                        script {
                            echo "System Report: ${checkStatus('Order-API', 200)}"
                            echo "System Report: ${checkStatus('Postgres-DB', 500)}"
                        }
                    }
                }
            }
        }
        
        // LESSON 2: Maps (Now Dynamic!)
        stage('Lesson 2: Dynamic Maps') {
            steps {
                script {
                    def deploymentConfig = [
                        'staging': [url: 'http://staging.internal', port: 8081],
                        'production': [url: 'http://myapp.com', port: 80]
                    ]
                    // Pulling data based on your UI selection
                    def selected = deploymentConfig[params.TARGET_ENV]
                    
                    echo "DEPLOYING TO ${params.TARGET_ENV}..."
                    echo "Target URL: ${selected.url}"
                    echo "Target Port: ${selected.port}"
                }
            }
        }

        // LESSON 3: Sandbox
        stage('Lesson 3: Sandbox') {
            steps {
                script {
                    def userHome = System.getProperty("user.home")
                    echo "Home path: ${userHome}"
                }
            }
        }
        // LESSON 5: Error Handling
        stage('Lesson 5: Error Handling') {
            steps {
                script {
                    try {
                        echo "Attempting a risky operation..."
                        error "The server connection timed out!" 
                    } catch (Exception e) {
                        echo "Caught the error: ${e.message}"
                    } finally {
                        echo "Cleanup complete on worker-1."
                    }
                }
            }
        }

        // LESSON 6: Shell Integration
        stage('Lesson 6: Shell Integration') {
            steps {
                script {
                    def kernelVersion = sh(script: 'uname -r', returnStdout: true).trim()
                    echo "The Podman container is running kernel: ${kernelVersion}"
                }
            }
        }

        // LESSON 7: Parameters (Visualizing the result)
        stage('Lesson 7: Parameters Output') {
            steps {
                script {
                    echo "User selected environment: ${params.TARGET_ENV}"
                    echo "Diagnostics enabled: ${params.RUN_DIAGNOSTICS}"
                }
            }
        }
        stage('Lesson 8: Environment Vars') {
            steps {
                script {
                    // Accessing environment variables in Groovy
                    echo "Starting ${env.APP_NAME} version ${env.API_VERSION}"
                    
                    // Accessing them in Shell (Notice the $ sign)
                    sh 'echo "The shell knows the APP_NAME is: $APP_NAME"'
                }
            }
        }
        stage('Lesson 9: Secure Credentials') {
            steps {
                script {
                    // Jenkins will MASK this output!
                    echo "My API Token is: ${env.MY_TOKEN}"
                    
                    // It also works in shell
                    sh 'echo "The shell secret is: $MY_TOKEN"'
                }
            }
        }
        stage('Lesson 10: Archiving') {
            steps {
                script {
                    // 1. Create a "fake" build report file
                    sh 'echo "Build version ${API_VERSION} was successful" > report.txt'
                    
                    // 2. Archive it so it appears in the Jenkins UI
                    archiveArtifacts artifacts: 'report.txt', fingerprint: true
                }
            }
        }
        stage('Lesson 10: Smart Archiving') {
            steps {
                script {
                    // We use double quotes "" so Groovy can inject variables
                    def reportContent = """
                        BUILD REPORT
                        ------------------
                        Application: ${env.APP_NAME}
                        Version:     ${env.API_VERSION}
                        Environment: ${params.TARGET_ENV}
                        Result:      SUCCESS
                        Node:        worker-1
                    """.stripIndent()
                    
                    // Write the variable to a file on the agent
                    writeFile file: 'report.txt', text: reportContent
                    
                    // Archive it to the controller
                    archiveArtifacts artifacts: 'report.txt'
                }
            }
        }
    }
    post {
        always {
            // Wrapping in node() ensures Jenkins finds the agent's disk to clean it
                cleanWs()
                echo "Cleanup attempt finished."
        }
        success {
            echo "Build Finished Successfully!"
        }
    }
}