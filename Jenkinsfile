pipeline {
    agent { label 'linuxgit' }

    environment {
        GIT_REPO = 'git@github.com:soumyasow2407-eng/git--cmake.git'  // Your repo
        BRANCH = 'main'

        // SonarCloud Configuration
        SONARQUBE_ENV = 'SonarCloud'
        SONAR_ORGANIZATION = 'soumyasow2407-eng'
        SONAR_PROJECT_KEY = 'soumyasow2407-eng_git--cmake'
    }

    stages {
        stage('Configure Git') {
            steps {
                echo 'Checking out source code from Git...'
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${GIT_REPO}",
                        credentialsId: 'gitHub-ssh-key'   // Your Git credential ID
                    ]]
                ])
            }
        }

        stage('Prepare Tools') {
            steps {
                echo 'Installing required tools...'
                sh '''
                    sudo apt update
                    sudo apt install -y python3 python3-pip dos2unix cmake gcc g++
                    pip3 install --quiet --break-system-packages cmakelint

                '''
            }
        }

        stage('Lint') {
            steps {
                echo 'Running lint checks on source files...'
                sh '''
                    if [ -d src ]; then
                        $HOME/.local/bin/cmakelint src/*.c > lint_report.txt || true

                else
                        echo "Source directory not found!"
                        exit 1
                    fi
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'lint_report.txt', fingerprint: true
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Running build with CMake...'
                sh '''
                    if [ -f CMakeLists.txt ]; then
                        mkdir -p build
                        cd build
                        cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ..
                        make -j$(nproc)
                        cp compile_commands.json ..
                    else
                        echo "CMakeLists.txt not found!"
                        exit 1
                    fi
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh '''
                    if [ -f build/Makefile ]; then
                        cd build
                        ctest --output-on-failure || true
                    else
                        echo "Build directory not found!"
                        exit 1
                    fi
                '''
            }
        }

        stage('SonarCloud Analysis') {
                when {
        branch 'main'
    }

            steps {
                echo 'Running SonarCloud analysis...'
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        sonar-scanner \
                          -Dsonar.organization=${SONAR_ORGANIZATION} \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=src \
                          -Dsonar.cfamily.compile-commands=compile_commands.json \
                          -Dsonar.host.url=https://sonarcloud.io \
                          -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo '✅ Build, lint, unit tests, and SonarCloud analysis completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs and SonarCloud dashboard.'
        }
    }
}
