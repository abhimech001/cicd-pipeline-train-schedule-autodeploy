pipeline {
    agent any
    environment {
        // docker
        DOCKER_IMAGE_NAME = "abhimech001/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage ('Build and Push Docker Image') {
            steps {
                sh 'docker build -t abhimech001/train-schedule:$BUILD_NUMBER .'            
            withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'Docker_pwd', usernameVariable: 'Docker_ID')]) {
                sh "echo '${Docker_pwd}' | docker login -u ${Docker_ID} --password-stdin"
            }
                sh 'docker push abhimech001/train-schedule:$BUILD_NUMBER'
            }
        }

        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
