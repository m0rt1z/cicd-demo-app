pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/m0rt1z/cicd-demo-app.git', description: 'HTTPS-ссылка на Fork репозитория')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Ветка, которую нужно собрать')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Выполнять unit-тесты')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    def scmVars = checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${params.GIT_BRANCH}"]],
                        userRemoteConfigs: [[url: params.REPO_URL]]
                    ])
                    env.GIT_BRANCH_NAME = scmVars.GIT_BRANCH
                    env.GIT_COMMIT_FULL = scmVars.GIT_COMMIT
                    echo "Branch: ${params.GIT_BRANCH} (${scmVars.GIT_BRANCH})"
                    echo "Commit SHA: ${scmVars.GIT_COMMIT}"
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    ['README.md', 'requirements.txt', 'src', 'tests'].each { path ->
                        if (!fileExists(path)) {
                            error "Required file or directory missing: ${path}"
                        }
                    }
                    echo 'Project structure OK: README.md, requirements.txt, src/, tests/ present'
                }
            }
        }

        stage('Test') {
            when { expression { params.RUN_TESTS } }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    printf 'Build number: %s\\nCommit SHA: %s\\n' "$BUILD_NUMBER" "$GIT_COMMIT_FULL" > build-info.txt
                    tar -czf app.tar.gz src tests README.md requirements.txt build-info.txt
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESS: all stages completed, artifact app.tar.gz archived'
        }
        failure {
            echo 'BUILD FAILURE: Build and Archive stages were not executed'
        }
    }
}