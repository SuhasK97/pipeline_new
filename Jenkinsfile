pipeline {
    agent { label 'git_Pipeline' }

    environment {
        GIT_REPO = 'https://github.com/SuhasK97/pipeline_new.git'
        BRANCH   = 'main'
    }

    stages {

        // -----------------------------
        stage('Checkout Repo') {
            steps {
                git branch: BRANCH,
                    url: GIT_REPO,
                    credentialsId: 'git_pipeline'
            }
        }

        // -----------------------------
        stage('Prepare Tools') {
            steps {
                sh '''
                    set -e

                    sudo apt-get update -y
                    sudo apt-get install -y \
                        build-essential \
                        python3 \
                        python3-pip \
                        pipx \
                        cmake \
                        curl

                    pipx ensurepath
                    pipx install cmakelint
                '''
            }
        }

        // -----------------------------
        stage('Lint') {
            steps {
                sh '''
                    if [ -f CMakeLists.txt ]; then
                        ~/.local/bin/cmakelint CMakeLists.txt > lint_report.txt || true
                    else
                        echo "CMakeLists.txt not found" > lint_report.txt
                    fi
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'lint_report.txt', fingerprint: true
                }
            }
        }

        // -----------------------------
        stage('Build') {
            steps {
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

        // -----------------------------
        stage('Unit Tests') {
            steps {
                sh '''
                    if [ -d build ]; then
                        cd build
                        ctest --output-on-failure --test-output-junit test-results.xml || true
                    else
                        echo "Build folder not found, skipping tests"
                    fi
                '''
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'build/test-results.xml'
                }
            }
        }
    }
}
