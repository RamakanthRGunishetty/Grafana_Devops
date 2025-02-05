pipeline {
    agent any
    environment {
        // Define the path to the local files and Docker image names
        LOCAL_SOURCE_PATH = '/path/to/your/local/files' // Replace with your actual path
        IMAGE_NAME = 'delivery_metrics'
    }
    stages {
        stage('Pre-check Docker') {
            steps {
                script {
                    try {
                        // Check if Docker is installed
                        def dockerVersion = sh(script: 'docker --version', returnStdout: true).trim()
                        if (!dockerVersion) {
                            error "Docker is not installed or not in the PATH. Please install Docker."
                        }

                        // Check if Docker daemon is running
                        def dockerInfo = sh(script: 'docker info', returnStatus: true)
                        if (dockerInfo != 0) {
                            error "Docker daemon is not running. Please start the Docker service."
                        }

                        echo "Docker is available and running: ${dockerVersion}"
                    } catch (Exception e) {
                        error "Pre-check failed: ${e.message}"
                    }
                }
            }
        }

        stage('Setup Workspace') {
            steps {
                script {
                    // Define the local source path where the files are located
                    def workspacePath = env.WORKSPACE

                    // Copy necessary files into the Jenkins workspace
                    sh """
                    cp ${LOCAL_SOURCE_PATH}/delivery_metrics.py ${workspacePath}/
                    cp ${LOCAL_SOURCE_PATH}/prometheus.yml ${workspacePath}/
                    cp ${LOCAL_SOURCE_PATH}/alert_rules.yml ${workspacePath}/
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image for delivery metrics
                    sh 'docker build -t ${IMAGE_NAME} .'
                }
            }
        }

        stage('Run Application') {
            steps {
                script {
                    // Run the application in a Docker container
                    sh 'docker run -d -p 8000:8000 --name ${IMAGE_NAME} ${IMAGE_NAME}'
                }
            }
        }

        stage('Run Prometheus & Grafana') {
            steps {
                script {
                    // Run Prometheus and Grafana using Docker containers
                    sh '''
                    docker run -d --name prometheus -p 9090:9090 \
                      -v $WORKSPACE/prometheus.yml:/etc/prometheus/prometheus.yml \
                      -v $WORKSPACE/alert_rules.yml:/etc/prometheus/alert_rules.yml \
                      prom/prometheus
                    docker run -d --name grafana -p 3000:3000 grafana/grafana
                    '''
                }
            }
        }
    }
    post {
        always {
            // Clean up Docker containers after the build
            echo 'Cleaning up Docker containers'
            sh '''
            docker rm -f delivery_metrics || true
            docker rm -f prometheus || true
            docker rm -f grafana || true
            '''
        }
    }
}
