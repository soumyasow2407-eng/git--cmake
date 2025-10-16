pipeline {
    agent any

    environment {
        REPO_URL = 'git@github.com:soumyasow2407-eng/git--cmake.git'
        BRANCH = 'main'
        SONARQUBE_ENV = 'SonarCloud'
        SONAR_ORGANIZATION = 'SowmyaMV'
        SONAR_PROJECT_KEY = 'soumyasow2407-eng'
}

    stages {
        stage('Prepare Tools') {
            steps {
                echo 'Installing required tools...'
                sh '''
                    if ! command -v pip3 &>/dev/null; then
                        sudo yum install -y python3 python3-pip || true
                    fi

                    pip3 install --quiet cmakelint

                    if ! command -v dos2unix &>/dev/null; then
                        sudo yum install -y dos2unix || true
                    fi

                    if ! command -v cmake &>/dev/null; then
                        sudo yum install -y epel-release || true
                        sudo yum install -y cmake || true
                    fi

                    if ! command -v gcc &>/dev/null; then
                        sudo yum install -y gcc gcc-c++ || true
                    fi
                '''
            }
        }

        stage('Lint') {
            steps {
                echo 'Running lint checks on main.c...'
                sh '''
                    if [ -f src/main.c ]; then
                        cmakelint src/main.c > lint_report.txt
                    else
                        echo "main.c not found!"
                        exit 1
                    fi
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'lint_report.txt', fingerprint: true
                    fingerprint 'main.c'
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
                    if [ -d build ]; then
                        cd build
                        # Run all registered CTest tests
                        ctest --output-on-failure
                    else
                        echo "Build directory not found!"
                        exit 1
                    fi
                '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube (SonarCloud) analysis...'
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
            echo 'Build, lint, and SonarCloud analysis completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs or SonarCloud dashboard.'
        }
    }
}
