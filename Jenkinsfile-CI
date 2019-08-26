pipeline {
    options {
        timeout(time: 10, unit: 'MINUTES')
    }
    agent {
        label 'master'
    }
    stages {
        stage('Checkout') {
            steps {
                script {
                    dir('release') {
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: '*/development']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'DevOpsINT', url: 'https://github.com/DevOpsINT/release.git']]])    
                    }
                    dir('experiment') {
                        deleteDir()
                        checkout([$class: 'GitSCM', branches: [[name: '*/development']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'DevOpsINT', url: 'https://github.com/DevOpsINT/experiment.git']]])                    
                        sh "git fetch --all"
                        CurrentVersion = sh script: "git branch -r | cut -d'/' -f2 | grep -v -e master -e HEAD -e dev | sort -r | head -1", returnStdout: true
                        CurrentVersion = CurrentVersion.trim()
                        // to fix next version
                        nextVersion = CurrentVersion + 1
                        commitIDshort = sh script:"git rev-parse HEAD | cut -c1-10", returnStdout: true
                        commitIDshort = commitIDshort.trim()
                        BuildVersion = "${CurrentVersion}_${commitIDshort}"
                        println("Current version is ${CurrentVersion}")
                        println("nextVersion is ${nextVersion}")
                        print("BuildVersion is ${BuildVersion}")
                    }
                }
            }
        }
        stage('Unit Test') {
            steps {
                script {
                    dir('experiment') {
                        sh "python ExperimentTests.py"
                        stash includes: '*', name: 'files', useDefaultExcludes: false
                    }
                }
            }
        }
        stage('Sanity Test') {
        agent {
            label 'docker'
        }      
            steps {
                script {
                    unstash 'files'
                    try {
                        sh "docker build . -t experiment:${nextVersion} &>/dev/null"
                        sh "docker run -d --name experiment experiment:${nextVersion}"
                        sh "docker logs experiment | grep Hello"
                    } catch(err) {
                        println("[ERROR] Failed on previous step ${err}")
                        sh "if docker ps -a | grep experiment; then docker rm experiment -f; fi"
                       currentBuild.result = 'FAILED'
                    }
                    sh "if docker ps -a | grep experiment; then docker rm experiment -f; fi"
                }
            }
        }
        stage('Tag and Upload') {
        agent {
            label 'docker'
        }      
            steps {
                script {   
                    sh "mkdir -p /mnt/artifacts/experiment/${nextVersion}"
                    sh "docker save experiment:${nextVersion} -o /mnt/artifacts/experiment/${nextVersion}/experiment.tar"
                }
            }
        }
        stage('Create new Git Branch') {
            steps {
                script {
                    //withCredentials([usernamePassword(credentialsId: 'DevOpsINT', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    dir('experiment') {
                        sh "git branch ${nextVersion}"
                        sh "git checkout ${nextVersion}"
                        sh "git push https://DevOpsINT:\\!Devopsshoam2019@github.com/DevOpsINT/experiment.git"
                    }
                    dir('release') {
                        sh "sed -i 's/${CurrentVersion}/${nextVersion}/g' dev.json"
                        sh "git config user.email 'devopsint@gmail.com'"
                        sh "git checkout development"
                        sh "git config user.name DevOpsINT"
                        sh "git add dev.json"
                        sh "git commit -m 'CI approved ${nextVersion}'"
                        sh "git push https://DevOpsINT:\\!Devopsshoam2019@github.com/DevOpsINT/release.git"
                    }
                }
            }
        }
    }
}
