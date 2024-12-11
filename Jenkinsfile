pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"  // Disable host key checking for Ansible
    }

    stages {
        stage('Determine Environment') {
            steps {
                script {
                    // Set the DEPLOY_ENV dynamically based on the branch name
                    env.DEPLOY_ENV = BRANCH_NAME == 'main' ? 'PROD' : 'DEV'
                }
                echo "Deploy environment set to: ${DEPLOY_ENV}"
            }
        }

        stage('Build Application') {
            steps {
                echo "Building application on branch: ${BRANCH_NAME}"
                sh 'cd demo-app && mvn clean package'
                archiveArtifacts artifacts: 'demo-app/target/*.war', fingerprint: true
            }
        }

        stage('Configure Servers') {
            steps {
                echo "Configuring ${DEPLOY_ENV} server with Tomcat..."
                ansiblePlaybook credentialsId: 'ec2-devops-key',  // Use Jenkins credential for SSH
                                 playbook: "ansible/install_tomcat.yml", 
                                 inventory: 'ansible/inventory.ini'
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying application to ${DEPLOY_ENV} server on branch: ${BRANCH_NAME}"
                ansiblePlaybook credentialsId: 'ec2-devops-key',  // Use Jenkins credential for SSH
                                 playbook: "ansible/deploy_app.yml", 
                                 inventory: 'ansible/inventory.ini'
            }
        }
    }

    post {
        always {
            echo "Pipeline for branch ${BRANCH_NAME} completed."
        }
    }
}
