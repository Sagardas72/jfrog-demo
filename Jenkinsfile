pipeline {
    agent any
    tools { 
        maven 'maven3.6' 
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-jfrog-demo')
    }
    stages {	
        stage ('Artifactory configuration') {
            steps {
//                rtServer (
//                   id: "ARTIFACTORY_SERVER",
//                   url: "http://artifactory:8082/artifactory",
//		     credentialsId: 'b7a1a8e8-1341-4366-989c-7e5f29a8fc82'
//                )
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog-artifactory",
                    releaseRepo: "test-libs-release-local",
                    snapshotRepo: "test-libs-snapshot-local"
                )
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog-artifactory",
                    releaseRepo: "test-libs-release",
                    snapshotRepo: "test-libs-snapshot"
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
            }
        }
        stage ('Push Dockerfile to Dockerhub') {
            steps {
                sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                sh "docker push $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:$BUILD_NUMBER"
                sh "docker push $DOCKERHUB_CREDENTIALS_USR/jfrog-demo:latest"
            }
        }
    }
    post {
        always {
            sh "docker logout"
        }
    }
}
