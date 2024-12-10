
pipeline {
    agent {
        docker {
          //  image 'maven:3.8.5-jdk-11' Maven with JDK 11
            image 'maven:3.8.5-eclipse-temurin-17' // Maven with JDK 17
        }
    }
    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
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
                echo "Configuring DEV and PROD servers with Tomcat..."
                ansiblePlaybook playbook: 'ansible/install_tomcat.yml', inventory: 'ansible/inventory.ini'
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying application to servers on branch: ${BRANCH_NAME}"
                ansiblePlaybook playbook: 'ansible/deploy_app.yml', inventory: 'ansible/inventory.ini'
            }
        }
    }

    post {
        always {
            echo "Pipeline for branch ${BRANCH_NAME} completed."
        }
    }
}
