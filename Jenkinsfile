pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
    }

    stages {
        stage('Build Application') {
            steps {
                sh 'cd demo-app && mvn clean package'
            }
        }

        stage('Configure DEV and PROD Servers') {
            steps {
                ansiblePlaybook playbook: 'ansible/install_tomcat.yml', inventory: 'ansible/inventory.ini'
            }
        }

        stage('Deploy to DEV and PROD') {
            steps {
                ansiblePlaybook playbook: 'ansible/deploy_app.yml', inventory: 'ansible/inventory.ini'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
