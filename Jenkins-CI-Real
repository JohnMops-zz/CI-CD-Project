pipeline {
    agent{
        label 'slave'
    }
    
    stages {
        stage ('checkout'){
            steps {
                script{
                dir('development'){
                deleteDir()
                checkout([$class: 'GitSCM', branches: [[name: '*/Dev']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'b6218c54-9fe9-4052-8da9-58322b94e248', url: 'https://github.com/JohnMops/CI-CD-Project']]])
                sh "git fetch --all"
                CurrentVersion = sh script: "git branch -r | cut -d'/' -f2 | grep -v -e master -e HEAD -e Dev | sort -r | head -1", returnStdout: true
                CurrentVersion = CurrentVersion.trim()
                nextVersion = CurrentVersion + 1
                commitIDshort = sh script:"git rev-parse HEAD | cut -c1-10", returnStdout: true
                commitIDshort = commitIDshort.trim()
                BuildVersion = "${CurrentVersion}_${commitIDshort}"
                }
                }
            }
        }
        stage ('Build Image and Sanity Test'){
            steps {
                script{
                    dir ('development') {
                    myImage = docker.build("johnmops/python-app:${nextVersion}", "./app")
                    myImage.withRun('-p 80:80 --name python') {c ->
                      try{
                          sh 'sleep 1'
                          sh 'curl -i -m 1 http://localhost'
                      } catch (err) {
                          currentBuild.result = 'FAILED'
                          
                      }
                    }
                }
                }
            }
        }
        stage ('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'b76ccfeb-b32d-41c0-b116-c85c72168576'){
                        myImage.push()
                }
                }
            }
        }
        stage ('Push to a new branch'){
            steps {
                script {
                    dir ('development'){
                        withCredentials([usernamePassword(credentialsId: 'b6218c54-9fe9-4052-8da9-58322b94e248', 
                        passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                        sh "git branch ${nextVersion}"
                        sh "git checkout ${nextVersion}"
                        sh 'git push https://${GIT_USER}:${GIT_PASS}@github.com/JohnMops/CI-CD-Project'
                    }
                    }
                }
            }
        }
    }
}
