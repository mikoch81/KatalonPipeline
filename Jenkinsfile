pipeline {

    agent any

    environment {
        KS_VERSION = '6.3.3'
    }

    stages {
        stage ("DO NOT DISABLE THIS STAGE") {
            steps {
                sh '''
                    chmod u+x ./build/*.sh
                    ./build/prevent_overwrite_existing_tag.sh $KS_VERSION
                '''
            }
        }
        stage ("Build") {
            steps {
                lock('katalon_pipeline') {

                    sh 'chmod u+x ./build/*.sh'
                    echo 'idziemy po chmod'
                    sh './build/clean.sh $KS_VERSION'
                    sh './build/build.sh $KS_VERSION'
                    sh './build/tag.sh $KS_VERSION'
                    echo 'po clean, build, tag'

                    sh 'chmod u+x ./test/project/*.sh'
                    echo 'po test/project chmod'
                    sh 'cd ./test/project && ./run_chrome.sh $KS_VERSION'
                    sh 'cd ./test/project && ./run_chrome_advanced.sh $KS_VERSION'
                    sh 'cd ./test/project && ./run_firefox.sh $KS_VERSION'
                    archiveArtifacts '**/*.avi'
                    echo 'archiveArtifact'
                    withDockerRegistry([ credentialsId: "docker-hub", url: "https://registry.hub.docker.com" ]) {
                        sh '''
                            ./build/tag.sh $KS_VERSION
                            ./build/push.sh $KS_VERSION
                            ./build/tag.sh latest
                            ./build/push.sh latest
                        '''
                    }
                }
            }
        }
    }
}
