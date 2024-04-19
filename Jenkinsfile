pipeline {
    agent any
    tools {
        nodejs 'node-14' // Adjust version number as per your Node.js setup
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/DevOps-Shack-3Tier.git'
            }
        }
        stage('Install Package Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Campground -Dsonar.projectName=Campground"
                }
            }
        }
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t adijaiswal/campa:latest ."
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html adijaiswal/campa:latest"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push adijaiswal/campa:latest"
                    }
                }
            }
        }
        stage('Deploy To EKS') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-5', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://0CB794B1E81785AF1D98888B0FB36B19.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f Manifests/"
                }
            }
        }
        stage('Verify the Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS-5', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://0CB794B1E81785AF1D98888B0FB36B19.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
