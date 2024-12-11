pipeline {
    agent any

    environment {
        ANSIBLE_HOST_KEY_CHECKING = "False"
    }

    options {
        // Ensures the workspace is cleaned before the job starts and after it ends
        cleanWs()
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

        stage('Clean Workspace') {
            steps {
                echo "Cleaning up workspace..."
                cleanWs() // Plugin for workspace cleanup
            }
        }

        stage('Build Application') {
            steps {
                echo "Building application on branch: ${BRANCH_NAME}"
                sh 'cd demo-app && mvn clean package'
            }
        }

        stage('Configure Servers') {
            steps {
                echo "Configuring ${DEPLOY_ENV} server with Tomcat..."
                sshagent(['tomcat-ssh']) {
                    ansiblePlaybook playbook: "ansible/install_tomcat_${DEPLOY_ENV}.yml", 
                                     inventory: 'ansible/inventory.ini',
                                     extraVars: [
                                         workspace_path: "${WORKSPACE}"
                                     ]
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo "Deploying application to ${DEPLOY_ENV} server on branch: ${BRANCH_NAME}"
                sshagent(['tomcat-ssh']) {
                    ansiblePlaybook playbook: "ansible/deploy_app_${DEPLOY_ENV}.yml", 
                                     inventory: 'ansible/inventory.ini',
                                     extraVars: [
                                         workspace_path: "${WORKSPACE}"
                                     ]
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed."
            cleanWs() // Ensures cleanup happens after pipeline execution
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
