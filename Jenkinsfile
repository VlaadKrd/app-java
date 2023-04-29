pipeline {
    agent {label 'master'}

    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
        timestamps()
        ansiColor('xtrem')
    }

    environment {
        SERVICES_FILE = 'simple-java-project/apps/services.json'
    }

    stages {
        stage('Prepare') {
            steps {
                script {
                    func = load 'app-java/func'
                    apps = readJSON file: SERVICES_FILE
                    println (apps)
                    Integer PARALLEL_EXECUTE_COUNT = 2
                    buildStages = func.stagePrepare('build', apps, PARALLEL_EXECUTE_COUNT)
                    builtApps = [:]
                }
            }
        }

        stage ('Parallel build') {
            steps {
                script {
                    buildStages.each { stage ->
                        parallel stage
                    }
                }
            }
        }

        stage ('Parallel create image') {
            steps {
                script {
                    Integer PARALLEL_EXECUTE_COUNT = 2
                    createImageStages = func.stagePrepare('image', builtApps, PARALLEL_EXECUTE_COUNT)
                    createImageStages.each { stage ->
                        parallel stage
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}