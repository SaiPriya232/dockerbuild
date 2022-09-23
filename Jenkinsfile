pipeline {
    agent any

    stages {
        stage('Source') {
            steps {
                git 'https://github.com/SushmithaNayak23/SpringMysqlDocker.git'
            }
        }
        
        stage('Build'){
            steps{
                echo 'mvn clean package'
            }
        }
        
        stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'LocalMaven'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=demo_project"
                        }
                    }
                }
            }
            
            stage('Packaging') {
                steps {
                    step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
                }
            }
            
            stage ("Artifactory Publish"){
                steps{
                    script{
                            def server = Artifactory.server 'artifactory_instanceid'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.deployer server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.tool = 'LocalMaven'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
    }
}

