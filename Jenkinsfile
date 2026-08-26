pipeline {
    agent any
    tools {
        jdk 'jdk'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
      stage('Git Checkout') {
         steps {
            git branch: 'main',
            url: 'https://github.com/mohamedshahen/java-pipeline-demo'
             }
       }
        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('SonarQube Analysis') {
           steps {
             withSonarQubeEnv('SonarQube') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ci-cd \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=ci-cd
                '''
        }
    }
}
    stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
    stage('Code Build') {
            steps {
                sh "mvn clean install"
            }
        }
    stage('Docker Version') {
    steps {
        sh '''
            echo "PATH=$PATH"
            which docker
            docker version
            docker info
        '''
    }
}
   stage('Docker Build & Push') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-login') {
                sh '''
                    docker build -t java-app:${BUILD_NUMBER} .
                    docker tag java-app:${BUILD_NUMBER} mohamedshahen/java-app:${BUILD_NUMBER}
                    docker push mohamedshahen/java-app:${BUILD_NUMBER}
                '''
            }
        }
    }
}
stage('deploy to kubernetes'){
    steps {
        script {
    withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubeconfig']]) {
    sh '''
             export PATH="/var/jenkins_home/.local/bin:$PATH"
             which kubectl
             kubectl version --client
             kubectl get ns 
             kubectl apply -f k8s/deployment.yaml
    '''
}}}
}
    }
}
