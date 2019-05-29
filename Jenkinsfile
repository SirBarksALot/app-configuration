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
        stage('Print current vars and set pipeline in gitlab to running') {
            agent any
            steps {
                sh '''echo ${BUILD_NUMBER}
                echo ${BUILD_TAG}
                echo ${BUILD_URL}
                echo ${JENKINS_URL}
                echo ${BUILD_URL}
                echo ${gitlabSourceBranch}
                echo ${gitlabActionType}
                echo ${gitlabUserName}
                echo ${gitlabSourceRepoURL}
                echo ${gitlabMergeRequestState}
                echo ${gitlabBefore}
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
                sh "mvn compile"
            }
        }
        stage('SonarQube analysis-new') {
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
                sh "sonar-scanner -Dsonar.host.url=http://172.17.0.3:9000 -Dsonar.java.binaries=**/target/classes -Dsonar.projectKey=jenkins-scanner -Dsonar.projectName=jenkins-scanner"
                
            }
            post{
                success {
                    addGitLabMRComment comment: 'Sonar succesfully done!'
                }
            }
        }
        stage('Archive Artifacts'){
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
                sh "mvn package -DskipTests"
                archiveArtifacts artifacts: '**/target/*.jar'
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
                sh "mvn clean deploy -DskipTests"
            }
        } 
        stage('Sonar scan test') {
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
                sh "echo feature2"
            }
        }
        stage('Send Artifact test') {
            agent any
            when {
                beforeAgent true
                allOf {
                    environment name: 'gitlabActionType', value: 'PUSH'
                    anyOf {
                        environment name: 'gitlabSourceBranch', value: 'development'
                        environment name: 'gitlabSourceBranch', value: 'release'
                        environment name: 'gitlabSourceBranch', value: 'hotfix'
                    }
                }
            }
            steps {
                sh "echo master"
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
