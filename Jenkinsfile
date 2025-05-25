pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose-deploy.yml'
        DEPLOY_SERVER="jjej02280@34.29.89.164"
    }

    stages {
        stage('Checkout') {
            steps {
		            deleteDir()
                git branch: 'main', url: 'https://github.com/Capstone-AdCanvas/versioning.git'
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker --version"
                    sh "docker compose --version"
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    sshagent(['deploy-server-access']) {
                        sh """
                        rm -rf .git
                        scp -r -o StrictHostKeyChecking=no . ${DEPLOY_SERVER}:~/directory
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        cd ~/directory &&
                        ls -al &&
                        docker-compose -f ${DOCKER_COMPOSE_FILE} down --remove-orphans
                        docker-compose -f ${DOCKER_COMPOSE_FILE} pull &&
                        docker-compose -f ${DOCKER_COMPOSE_FILE} up -d'
                        """
                    }
                }
            }
        }
    }

    post {
		    always {
                cleanWs(cleanWhenNotBuilt: false,
                        deleteDirs: true,
                        disableDeferredWipeout: true,
                        notFailBuild: true,
                        patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                                [pattern: '.propsfile', type: 'EXCLUDE']])
        }
        success {
            echo 'Build and deployment successful!'
        }
        failure {
            echo 'Build or deployment failed.'
        }
    }
}
