// multistage
pipeline {
    agent any

        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/kparunsagar/jenkins_sonar_artifactory.git'
                }
            }
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'maven'
                        bat "${mvnHome}\\bin\\mvn -B verify"
                    }
                }
            }
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'maven'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=sonardemo"
                        }
                    }
                }
            }
            
            stage('Test') {
                steps {
                    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
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
                            def server = Artifactory.server 'artifactory'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkinsdemo_repo', snapshotRepo: 'demopipeline'
                            rtMaven.deployer server: server, releaseRepo: 'jenkinsdemo_repo', snapshotRepo: 'demopipeline'
                            rtMaven.tool = 'maven'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
        }
}
