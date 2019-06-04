pipeline {
    agent any
    options { disableConcurrentBuilds() }
    environment {
        committerEmail = ""
    }
    parameters {
        booleanParam(name: "RELEASE", description: "Build a release from current commit.", defaultValue: false)
    }
    stages {
        stage('Setup') {
            agent any
            steps {
                sh '''echo ${BUILD_NUMBER}
                echo ${BUILD_TAG}
                echo ${BUILD_URL}
                echo ${JENKINS_URL}
                echo ${gitlabSourceBranch}
                echo ${gitlabActionType}
                echo ${gitlabUserName}
                echo ${gitlabSourceRepoURL}
                echo ${gitlabMergeRequestState}
                echo ${gitlabBefore}
                echo ${gitlabBefore}
                echo ${gitlabAfter}
                echo ${gitlabAfter}
                echo ${gitlabAfter}

                echo ${gitlabTriggerPhrase}'''
                updateGitlabCommitStatus(name: "${gitlabActionType}", state: 'running')
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'maven-openjdk8:2.0.0'
                    args '-v /root/.m2:/root/.m2"'
                }
            }
            when {
                beforeAgent true
                expression { 
                !params.RELEASE
                env.jenkins == "jenkins_new"
                }
            }
            steps {
                updateGitlabCommitStatus(name: 'Build', state: 'running')
                sh "mvn compile"
            }
            post {
                success {
                    updateGitlabCommitStatus(name: 'Build', state: 'success')
                }
                failure {
                    updateGitlabCommitStatus(name: 'Build', state: 'failed')
                }
            }
        }
        stage('Unit Test') {
            agent any
            when {
                beforeAgent true
                not {
                    anyOf {
                        environment name: 'gitlabSourceBranch', value: 'development'
                        environment name: 'gitlabSourceBranch', value: 'release'
                        environment name: 'gitlabSourceBranch', value: 'hotfix'
                        environment name: 'gitlabSourceBranch', value: 'master'
                    }
                }
            }
            steps {
                sleep(5)
                updateGitlabCommitStatus(name: 'Unit Test', state: 'running')
                sh "echo feature27"
            }
            post {
                success {
                    updateGitlabCommitStatus(name: 'Unit Test', state: 'success')
                }
                failure {
                    updateGitlabCommitStatus(name: 'Unit Test', state: 'failed')
                }
            }
        }
        stage('Checkmarx scan') {
            agent any
            when {
                beforeAgent true
                not {
                    anyOf {
                        environment name: 'gitlabSourceBranch', value: 'development'
                        environment name: 'gitlabSourceBranch', value: 'release'
                        environment name: 'gitlabSourceBranch', value: 'hotfix'
                        environment name: 'gitlabSourceBranch', value: 'master'
                    }
                }
            }
            steps {
                sleep(5)
            }
            post {
                success {
                    updateGitlabCommitStatus(name: 'Checkmarx scan', state: 'success')
                }
                failure {
                    updateGitlabCommitStatus(name: 'Checkmarx scan', state: 'failed')
                }
            }
        }
        /*stage('SonarQube analysis') {
            agent {
                docker {
                    image 'sonnar-scanner3.3.0:1.0.0'
                    args '-v /root/.m2:/root/.m2'
                }
            }
            when {
                beforeAgent true
                allOf {
                    expression { 
                        !params.RELEASE
                        env.jenkins == "jenkins_new"
                    }
                    not {
                        anyOf {
                            environment name: 'gitlabSourceBranch', value: 'development'
                            environment name: 'gitlabSourceBranch', value: 'release'
                            environment name: 'gitlabSourceBranch', value: 'hotfix'
                            environment name: 'gitlabSourceBranch', value: 'master'
                        }
                    }
                }
            }
            steps {
                updateGitlabCommitStatus(name: 'Sonarqube scan', state: 'running')
                withSonarQubeEnv('SonarQubeScannerServer') {
                    sh
                }
            }
            post{
                success {
                    addGitLabMRComment comment: 'Sonarqube scan succesfully done!'
                    updateGitlabCommitStatus(name: 'Sonarqube scan', state: 'success')
                }
                failure {
                    addGitLabMRComment comment: 'Sonarqube scan failed!'
                    updateGitlabCommitStatus(name: 'Sonarqube scan', state: 'failed')
                }
            }
        }*/
        /*stage("Quality Gate") {
            agent {
                docker {
                    image 'sonnar-scanner3.3.0:1.0.0'
                    args '-v /root/.m2:/root/.m2'
                }
            }

            when {
                beforeAgent true
                allOf {
                    expression { 
                        !params.RELEASE
                        env.jenkins == "jenkins_new"
                    }
                    not {
                        anyOf {
                            environment name: 'gitlabSourceBranch', value: 'development'
                            environment name: 'gitlabSourceBranch', value: 'release'
                            environment name: 'gitlabSourceBranch', value: 'hotfix'
                            environment name: 'gitlabSourceBranch', value: 'master'
                        }
                    }
                }
            }
            steps {
                updateGitlabCommitStatus(name: 'Quality Gate', state: 'running')
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post{
                success {
                    addGitLabMRComment comment: 'Quality Gate succesfully passed!'
                    updateGitlabCommitStatus(name: 'Quality Gate', state: 'success')
                }
                failure {
                    addGitLabMRComment comment: 'Quality Gate failed!'
                    updateGitlabCommitStatus(name: 'Quality Gate', state: 'failed')
                }
            }
        }*/
        stage('Archive Artefacts') {
            agent {
                docker {
                    image 'maven-openjdk8:2.0.0'
                    args '-v /root/.m2:/root/.m2"'
                }
            }
            when {
                beforeAgent true
                allOf {
                    expression { 
                        !params.RELEASE
                        env.jenkins == "jenkins_new"
                    }
                    allOf {
                        environment name: 'gitlabActionType', value: 'PUSH'
                        anyOf {
                            environment name: 'gitlabSourceBranch', value: 'development'
                            environment name: 'gitlabSourceBranch', value: 'release'
                            environment name: 'gitlabSourceBranch', value: 'hotfix'
                        }
                    }
                }
            }
            steps {
                updateGitlabCommitStatus(name: 'Archive Artefacts', state: 'running')
                sh "mvn package -DskipTests"
                archiveArtifacts artifacts: '**/target/*.jar'
            }
            post {
                success {
                    updateGitlabCommitStatus(name: 'Archive Artefacts', state: 'success')
                }
                failure {
                    updateGitlabCommitStatus(name: 'Archive Artefacts', state: 'failed')
                }
            }
        }
        stage('Send Artifacts'){
            agent {
                docker {
                    image 'maven-openjdk8:2.0.0'
                    args '-v /root/.m2:/root/.m2"'
                }
            }
            when {
                beforeAgent true
                allOf {
                    expression { 
                        !params.RELEASE
                        env.jenkins == "jenkins_new"
                    }
                    allOf {
                        environment name: 'gitlabActionType', value: 'PUSH'
                        anyOf {
                            environment name: 'gitlabSourceBranch', value: 'development'
                            environment name: 'gitlabSourceBranch', value: 'release'
                            environment name: 'gitlabSourceBranch', value: 'hotfix'
                        }
                    }
                }
            }
            steps {
                updateGitlabCommitStatus(name: 'Sending Artefacts', state: 'running')
                sh "mvn clean deploy -DskipTests"
            }
            post {
                success {
                    updateGitlabCommitStatus(name: 'Sending Artefacts', state: 'success')
                }
                failure {
                    updateGitlabCommitStatus(name: 'Sending Artefacts', state: 'failed')
                }
            }
        } 

    }
    post { 
	    success {
            updateGitlabCommitStatus(name: "${gitlabActionType}", state: 'success')
  	    }
  	    unsuccessful {
            updateGitlabCommitStatus(name: "${gitlabActionType}", state: 'failed')
  	    }
  	}
}
