pipeline {
    agent any
    tools { 
	// Global tools to be used by the pipeline
        maven 'maven3.6' 
        jdk 'jdk8' 
    }
    stages {	
        stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "ARTIFACTORY_SERVER",
                    url: "http://artifactory:8082/artifactory",
					credentialsId: 'b7a1a8e8-1341-4366-989c-7e5f29a8fc82'
                )
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "ARTIFACTORY_SERVER",
                    releaseRepo: "test-libs-release-local",
                    snapshotRepo: "test-libs-snapshot-local"
                )
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "ARTIFACTORY_SERVER",
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
                    serverId: "ARTIFACTORY_SERVER"
                )
            }
        }
    }
}