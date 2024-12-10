pipeline {
    agent {
        docker {
            image 'maven:3.8.5-eclipse-temurin-17' // Maven with JDK 17
        }
    }
    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
        DEPLOY_ENV = BRANCH_NAME == 'main' ? 'PROD' : 'DEV'
    }

    stages {
        stage('Build Application') {
            steps {
                echo "Building application on branch: ${BRANCH_NAME}"
                sh 'cd demo-app && mvn clean package'
            }
        }

        stage('Configure Servers') {
            steps {
                echo "Configuring ${DEPLOY_ENV} server with Tomcat..."
                ansiblePlaybook playbook: "ansible/install_tomcat_${DEPLOY_ENV}.yml", inventory: 'ansible/inventory.ini'
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying application to ${DEPLOY_ENV} server on branch: ${BRANCH_NAME}"
                ansiblePlaybook playbook: "ansible/deploy_app_${DEPLOY_ENV}.yml", inventory: 'ansible/inventory.ini'
            }
        }
    }

    post {
        always {
            echo "Pipeline for branch ${BRANCH_NAME} completed."
        }
    }
}
