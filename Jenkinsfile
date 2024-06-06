pipeline {
    agent any

    environment {
        DEPLOY_USER = 'vagrant'
        DEPLOY_HOST = '192.168.33.10'
        SSH_KEY = credentials('Fastapi_Vagrant')
        GITHUB_TOKEN = credentials('github_token')
        PLINK_PATH = 'C:\\Program Files\\PuTTY\\plink.exe'
        PSCP_PATH = 'C:\\Program Files\\PuTTY\\pscp.exe'
        DEPLOY_PASSWORD = 'vagrant'
        REPO_URL = 'https://github.com/ritesh1603/ReactDeploy.git'
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        env.DEPLOY_PATH = '/home/vagrant/dev'
                        env.BRANCH_TO_BUILD = 'uat'
                    } else if (env.BRANCH_NAME == 'uat') {
                        env.DEPLOY_PATH = '/home/vagrant/uat'
                        env.BRANCH_TO_BUILD = 'staging'
                    } else if (env.BRANCH_NAME == 'staging') {
                        env.DEPLOY_PATH = '/home/vagrant/staging'
                        env.BRANCH_TO_BUILD = null
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    git(
                        branch: env.BRANCH_NAME,
                        url: "${REPO_URL}",
                        credentialsId: 'github_token'
                    )
                }
            }
        }

        stage('Cache SSH Host Key') {
            steps {
                bat """
                echo y | "$PLINK_PATH" -pw "$DEPLOY_PASSWORD" "$DEPLOY_USER@$DEPLOY_HOST" exit
                """
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                bat """
                "$PLINK_PATH" -pw "$DEPLOY_PASSWORD" "$DEPLOY_USER@$DEPLOY_HOST" "ansible-playbook /home/vagrant/ansible_project/react/${env.BRANCH_NAME}-deployment-playbook.yml"
                """
            }
        }

        // stage('Notify and Trigger Next Build') {
        //     when {
        //         expression { env.BRANCH_TO_BUILD != null }
        //     }
        //     steps {
        //         script {
        //             build job: 'MultiBranchDeployment_React', parameters: [[$class: 'StringParameterValue', name: 'BRANCH', value: env.BRANCH_TO_BUILD]]
        //         }
        //     }
        // }
    }

    post {
        success {
            echo "Build and deployment for ${env.BRANCH_NAME} completed successfully."
        }
        failure {
            echo "Build and deployment for ${env.BRANCH_NAME} failed."
        }
    }
}

