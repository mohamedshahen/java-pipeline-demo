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
                sh "mvn -e clean compile"
            }
        }

        stage('Unit testing') {
            steps {
                sh "mvn test -Dmaven.test.skip=true"
            }
        }
        
        stage('SonarQube Analysis') {
            when {
        expression {
	   true
        }
    }
           steps {
             withSonarQubeEnv('SonarQube') {
                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=demo \
                -Dsonar.java.binaries=. \
                -Dsonar.projectKey=demo
                '''
        }
    }
}

stage('OWASP Dependency') {
when {
        expression {
            params.RUN_SONAR
        }
    }

            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

stage('OWASP Dependency') {
    steps {
        dependencyCheck(
            additionalArguments: '--scan ./',
            odcInstallation: 'dependency-check'
        )

        dependencyCheckPublisher(
            pattern: '**/dependency-check-report.xml'
        )

        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: '.',
            reportFiles: 'dependency-check-report.html',
            reportName: 'OWASP Dependency-Check Report'
        ])
    }
}

    stage('Quality Gate') {
        when {
        expression {
            params.RUN_SONAR
        }
    }
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
    stage('Code Build') {
        when {
        expression {
            params.RUN_SONAR
        }
    }
            steps {
                sh "mvn clean install"
            }
        }
    stage('Docker Version') {
        when {
        expression {
            params.RUN_SONAR
        }
    }
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
       when {
        expression {
            params.RUN_SONAR
        }
    }
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
    when {
        expression {
            params.RUN_SONAR
        }
    }
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
