pipeline {
    agent any
    tools { 
        maven 'maven3.6' 
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-jfrog-demo')
        ARTIFACTORY_CREDENTIALS = credentials('artifactory-jfrog-demo')
    }
    stages {	
        stage ('Artifactory configuration') {
            steps {
//                rtServer (
//                   id: "ARTIFACTORY_SERVER",
//                   url: "http://artifactory:8082/artifactory",
//		     credentialsId: 'artifactory-jfrog-demo'
//                )
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog-artifactory",
                    releaseRepo: "jfrog-demo-libs-release-local",
                    snapshotRepo: "jfrog-demo-libs-snapshot-local"
                )
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog-artifactory",
                    releaseRepo: "jfrog-demo-libs-release",
                    snapshotRepo: "jfrog-demo-libs-snapshot"
                )
            }
        }
        stage ('Build & Upload Artifact') {
            steps {
                rtMavenRun (
                    tool: "maven3.6",
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
            }
        }
        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog-artifactory"
                )
            }
        }
        stage ('Build latest Docker Image') {
            steps {
                sh "docker build -t $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER ."
                sh "docker tag $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:latest"
                sh "docker tag $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER artifactory:8082/jfrog-demo-docker-local/jfrog-demo:$BUILD_NUMBER"
                sh "docker tag $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER artifactory:8082/jfrog-demo-docker-local/jfrog-demo:latest"
            }
        }
        stage ('Push Docker Image to DockerHub') {
            steps {
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker push $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER"
                sh "docker push $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:latest"
                sh "docker logout"
            }
        }
        stage ('Push Docker Image to Artifactory') {
            steps {
                sh "echo $ARTIFACTORY_CREDENTIALS_PSW | docker login -u $ARTIFACTORY_CREDENTIALS_USR --password-stdin artifactory:8082"
                sh "docker push artifactory:8082/jfrog-demo-docker-local/jfrog-demo:$BUILD_NUMBER"
                sh "docker push artifactory:8082/jfrog-demo-docker-local/jfrog-demo:latest"
            }
        }
    }
    post {
        always {
            sh "docker logout"
        }
    }
}
