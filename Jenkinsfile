pipeline {
    agent any
    environment {
        DOCKERHUB = credentials('dockerhub-rahali')
        SONAR_TOKEN = credentials('SONAR_TOKEN')
        IMAGE = 'mohamedalrahali/devvops'
    }
    stages {
        stage('Checkout') { steps { git branch: 'main', url: 'https://github.com/MohamedalRahali/devvops.git' } }
        stage('Maven Build & Test') { steps { sh 'mvn -B clean verify' } }
        stage('SonarCloud') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh 'mvn -B sonar:sonar -Dsonar.projectKey=mohamedalrahali_devvops -Dsonar.organization=mohamedrahali -Dsonar.login=$SONAR_TOKEN'
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                sh '''
                docker build -t ${IMAGE}:${BUILD_NUMBER} .
                docker tag ${IMAGE}:${BUILD_NUMBER} ${IMAGE}:latest
                echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                docker push ${IMAGE}:${BUILD_NUMBER}
                docker push ${IMAGE}:latest
                '''
            }
        }
        stage('Deploy to Minikube') {
            steps {
                sh '''
                sed -i "s|mohamedalrahali/devvops:latest|${IMAGE}:${BUILD_NUMBER}|g" k8s/deployment.yaml
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
    post { success { echo 'Pipeline complète terminée avec succès !' } }
}
