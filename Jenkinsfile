// multistage
pipeline {
    agent any

        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/SaiPriya232/dockerbuild.git'
                }
            }
            
            stage('Docker bulid') {
                steps {
                    script {
                        bat 'docker build -t 1stdoc .'
                        bat 'docker images'
                    }
                }
            }
      
             stage('Build docker image') {
                steps {
                    script {
                        bat 'docker-compose up'
                    }
                }
            }
        
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        bat "${mvnHome}\\bin\\mvn -B verify"
                    }
                }
            }
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=dockerbuild"
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
                            def server = Artifactory.server 'JFROG'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.deployer server: server, releaseRepo: 'dockerbuild-1', snapshotRepo: 'dockerbuild-2'
                            rtMaven.tool = 'M3'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
          
        }
}
