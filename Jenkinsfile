pipeline {
    agent any

    environment {
        REPO_URL = 'git@github.com:soumyasow2407-eng/git--cmake.git'
        BRANCH = 'main'
        BUILD_DIR = 'build'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out branch ${BRANCH} from ${REPO_URL}"
                deleteDir()
                sshagent(credentials: ['gitHub-ssh-key']) {
                    sh "git clone -b ${BRANCH} ${REPO_URL} repo"
                }
            }
        }

        stage('Configure') {
            steps {
                sh """
                cd repo
                mkdir -p ${BUILD_DIR}
                cd ${BUILD_DIR}
                cmake ..
                """
            }
        }

        stage('Build') {
            steps {
                sh """
                cd repo/${BUILD_DIR}
                cmake --build .
                """
            }
        }

        stage('Run Executable') {
            steps {
                sh """
                cd repo/${BUILD_DIR}
                ./your_executable_name
                """
            }
        }
    }
}
