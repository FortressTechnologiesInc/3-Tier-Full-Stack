pipeline {
    agent any
    tools {
        nodejs 'node' // Adjust version number as per your Node.js setup
    }
    environment {
        SCANNER_HOME = tool 'scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/FortressTechnologiesInc/3-Tier-Full-Stack.git'
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
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t limkel/campa:2.0  ."
                    }                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html limkel/campa:2.0"
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker push limkel/campa:2.0"
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
