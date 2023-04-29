def stagePrepare(flag, apps, parallel_count) {
    stageList = []
    stageMap = [:]
    apps.eachWithIndex { app, path, i ->
        Integer lock_id = i % parallel_count
        if (flag == 'build') {
            stageMap.put(app, stageBuildCreate(app, path, lock_id))
        } else {
            stageMap.put(app, stageImageCreate(app, path, lock_id))
        }
    }
    stageList.add(stageMap)
    return stageList
}

def stageBuildCreate(app, path, lock_id) {
    return {
        stage(app) {
            lock("Build-lock-${lock_id}") {
                dir(path) {
                    sh """
                        [ -d target] || mkdir target
                        docker build -t ${app} -f Dockerfile-build .
                        docker run --name ${app} ${app} mvn test &&
                        docker cp ${app}:/app/target/ target/
                        docker rm -f ${app}
                    """
                    builtApps.put(app, path)
                }
            }
        }
    }
}

def stageImageCreate(app, path, lock_id) {
    return {
        stage(app) {
            lock("Image-create-lock-${lock_id}") {
                dir (path) {
                    sh "docker build -t ${app} -f Dockerfile-create ."
                }
            }
        }
    }
}


pipeline {
    agent {Label 'master'}

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
                    apps = readJSON file: SERVICES_FILE
                    println (apps)
                    Integer PARALLEL_EXECUTE_COUNT = 2
                    buildStages = stagePrepare('build', apps, PARALLEL_EXECUTE_COUNT)
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
                    createImageStages = stagePrepare('image', builtApps, PARALLEL_EXECUTE_COUNT)
                    createImageStages.each { stage ->
                        parallel stage
                    }
                }
            }
        }


        // // stage('Build') {
        // //     steps {
        // //         script {
        // //             apps.each { app, path ->
        // //                 dir(path) {
        // //                     sh """
        // //                         docker rm -f ${app}
        // //                         [ -d target] || mkdir target
        // //                         docker build -t ${app} -f Dockerfile-build .
        // //                         docker run --name ${app} ${app} mvn test &&
        // //                         docker cp ${app}:/app/target/ target/
        // //                         docker rm -f ${app}
        // //                     """
        // //                 }
        // //             }
        // //         }
        // //     }
        // //     post {
        // //         always {
        // //             script {
        // //                 apps.each { app, path ->
        // //                     dir (path) {
        // //                         junit 'target/target/surefire-reports/*.xml'
        // //                     }
        // //                 }
        // //             }
        // //         }
        // //     }
        // // }
        
        // stage('Image create') {
        //     steps {
        //         script {
        //             apps.each { app, path ->
        //                 dir (path) {
        //                     sh "docker build -t ${app} -f Dockerfile-create ."
        //                 }
        //             }
        //         }
        //     }
        // }
    }
    post {
        always {
            cleanWs()
        }
    }
}