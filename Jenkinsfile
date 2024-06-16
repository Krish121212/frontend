pipeline {
    agent {
        label 'Agent-1' //config agent in jenkins after creatin server and add the agent name here
    }
    options{
        //how much time does a snapshot need to run? max time? that we will configure here
        timeout(time: 30, unit: 'MINUTES') //we can give mints, hours, seconds etc.. 
        disableConcurrentBuilds() //Next build will wait for the previous build to get completed.
        ansiColor('xterm')
    }
    environment {
        def appVersion = '' //declared variable at env level, this can be used in any stage.
        nexusUrl = 'nexus.devopskk.online:8081' //nexus runs on port 8081 jenkins on 8080
    }
    stages {
        stage('Read Version') {
            steps {
                script{ //grovy script so code need to be in script {}
                def packageJson = readJSON file: 'package.json'
                appVersion = packageJson.version
                echo "application version: $appVersion"
                }
            }
        }
        stage('Build'){
            steps{
                sh """
                    zip -q -r frontend.${appVersion}.zip * -x Jenkinsfile -x frontend.${appVersion}.zip
                    ls -ltr
                """
            }
        }
        stage('Nexus Artifact Upload'){
            steps{ //while variables using below give "" not ''
                script{ // in Jenkins for push we need credantials line 52. pull no need. As we are pushing to nexus credantials are mandatory
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "frontend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "frontend" ,
                            classifier: '',
                            file: "frontend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy') {
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}") //if wait is true this frontend will wait until downstream 'frontend-deploy' is completed
                    ]   
                    build job: 'frontend-deploy', parameters: params, wait: false
                }
            }
        }    
    }
    post {//we have many posts,below are 3 among them. so posts run after build.used for trigging mails about status etc
        always { 
            echo 'the steps we write here will always run after any build'
            deleteDir()    //this deletes the build files after build in directory. otherwise memory waste
        }
        success { 
            echo 'the steps we write here will run after only success build'
        }
        failure { 
            echo 'the steps we write here will run after only failure build'
        }
    }
}