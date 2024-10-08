pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
        // docker 'docker-latest'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        // DOCKERHUB_CREDENTIALS = credentials('jenkins-docker-id')
    }

    stages {
        stage('Git-Checkout') {
            steps {
                checkout changelog: false, poll: false, scm: scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/MohammadRafi44/DevopsCICD--project-2__productionlevel_cicd_Task-Master-Pro.git']])
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Unit-Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SONARQUBE-ANALYSIS'){
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=taskmaster \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=taskmaster '''
                }
            }
        }
        stage('Trivy-FS-Scan'){
            steps {
                // sh 'trivy fs --security-checks vuln,config /var/lib/jenkins/workspace/cicd-devops-pipiline'
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('Build'){
            steps {
                sh " mvn package"
            }
        }

        stage('publish-artifact'){
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                   sh 'mvn deploy'
                }
            }
        }

        stage('DOCKER-BUILD'){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t mohammadrafi44/taskmaster:latest ."
                    }
                }
            }
        }

        stage('Trivy-image-scan'){
            steps {
                sh 'trivy image --format table -o image.html mohammadrafi44/taskmaster:latest'
                }
        }

        stage('Docker-Publish'){
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push mohammadrafi44/taskmaster:latest"
                    }
                }
            }
        }
        stage("Docker-Image-Cleanup"){
            steps {
                script {
                    echo 'docker images cleanup started'
                    sh 'docker system prune -af'
                    echo 'docker images cleanup finished'
                }
            }
        }
        stage("K8s-Deploy"){
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://41BFF322BA9DC0AC6E2B601200EBB29D.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f deployment-service.yml'
                    sleep 30 
                }
            }
        }
        
        stage("K8s-Verify-Deploy"){
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://41BFF322BA9DC0AC6E2B601200EBB29D.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
    }
}