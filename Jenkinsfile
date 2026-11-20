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
                    echo "Branch: ${scmVars.GIT_BRANCH}"
                    echo "Commit SHA: ${scmVars.GIT_COMMIT}"
                }
            }
        }

        stage('Validate') {
            steps {
                script {
                    ['README.md', 'requirements.txt', 'src', 'tests'].each { path ->
                        if (!fileExists(path)) {
                            error "Обязательный файл или каталог отсутствует: ${path}"
                        }
                    }
                    echo 'Структура проекта корректна: README.md, requirements.txt, src/, tests/ на месте'
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
                    tar -czf app.tar.gz --exclude=./app.tar.gz --exclude=./.git .
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
            echo 'Сборка завершена успешно: все стадии выполнены, артефакт app.tar.gz сохранён'
        }
        failure {
            echo 'Сборка завершена с ошибкой: стадии Build и Archive не выполнялись'
        }
    }
}