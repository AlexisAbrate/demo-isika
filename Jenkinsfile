 pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS=credentials('dockerhub-credential')
    }
    
    stages {
        stage('Check out') {
            steps {
                echo '-=- Checkout project -=-'
                git branch: 'master', url:'https://github.com/AlexisAbrate/demo-isika.git'
            }
        }
         stage('Compile') {
            steps {
                echo '-=- Compile project -=-'
                sh 'mvn clean compile'
            }
        }
         stage('Test') {
            steps {
                echo '-=- Test project -=-'
                sh 'mvn test'
            }
            post{
                success{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
         stage('Package') {
            steps {
                echo '-=- Package project -=-'
                sh 'mvn package -DskipTests'
            }
            post{
                always{
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        
        stage('Cleaning project image and container') {
            steps {
                echo '-=- Docker build -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 docker stop demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 docker rm demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 docker rmi dockerabrate/demo-isika || true'
                sh 'docker rmi dockerabrate/demo-isika || true'
            }
        }
        
        stage('Construct image') {
            steps {
                echo '-=- Docker build -=-'
                sh 'docker build -t demo-isika .'
            }
        }
        
        stage('Tag') {
            steps {
                echo '-=- Docker run -=-'
                sh 'docker tag demo-isika dockerabrate/demo-isika'
            }
        }
        
        stage('Login to Docker Hub') {
            steps {
                echo '-=- Login DH -=-'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo '-=- Login DH -=-'
                sh 'docker push dockerabrate/demo-isika'
            }
        }
        
        stage('Run container to local') {
            steps {
                echo '-=- Run Container -=-'
                sh 'ssh -v -o StrictHostKeyChecking=no vagrant@192.168.33.20 sudo docker run -d --name demo-isika -p 8080:8080 dockerabrate/demo-isika'
            }
        }
        
        stage ('Deploy To AWS'){
              input{
                message "Do you want to proceed for production deployment?"
              }
            steps {
                sh 'echo "Deploy into Prod"'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@54.172.8.225 sudo docker stop demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@54.172.8.225 sudo docker rmi demo-isika || true'
                sh 'ssh -v -o StrictHostKeyChecking=no ubuntu@54.172.8.225 sudo docker run -d --name demo-isika -p 8080:8080 dockerabrate/demo-isika'
                

            }
        }

        
    }
    
    post {
        always {
            sh 'docker logout'
        }
    }
        
}