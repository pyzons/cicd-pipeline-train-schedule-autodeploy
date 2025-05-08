pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "pyzons/train-schedule"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh '''
                    echo $PATH
                    export PATH=$PATH:/home/vsk2042/.nvm/versions/node/v9.11.1/bin
                    npm -v
                    node -v
                    JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64 ./gradlew build --no-daemon --refresh-dependencies
                '''
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            agent { label 'master' }
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh 'kubectl apply -f train-schedule-kube-canary.yml'
                //kubernetesDeploy(
                //    kubeconfigId: 'kube-config',
                 //   configs: 'train-schedule-kube-canary.yml',
                 //   enableConfigSubstitution: true
                //)
            }
        }
        stage('DeployToProduction') {
            agent { label 'master' }
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                //sh 'kubectl apply -f train-schedule-kube-canary.yml'
                //kubernetesDeploy(
                 //   kubeconfigId: 'kubeconfig',
                 //   configs: 'train-schedule-kube-canary.yml',
                  //  enableConfigSubstitution: true
                //)
                sh 'kubectl apply -f train-schedule-kube.yml'
                //kubernetesDeploy(
                 //   kubeconfigId: 'kubec-onfig',
                  //  configs: 'train-schedule-kube.yml',
                  //  enableConfigSubstitution: true
                //)
            }
        }
    }
}
