pipeline {
    agent any
    
    triggers {
        bitbucketPush()
    }

    tools {
        jdk 'OpenJDK_1_8_0'
    }

    stages {
        stage('Artifactory config') {
            steps {
                rtServer(
                    id: "server_tutorial",
                    url: "http://3.134.8.172:8081/artifactory",
                    credentialsId: "artifactory-credentials"
                )
                rtMavenDeployer(
                    id: "MAVEN_DEPLOYER",
                    serverId: "server_tutorial",
                    releaseRepo: "libs-release",
                    snapshotRepo: "libs-release"
                )
                //with the resolver it doesn't work, I suppose it's because Artifactory doesn't support Maven3
                //I'll manage the resolver by changing the settings.xml
                /*rtMavenResolver(id:"MAVEN_RESOLVER",serverId:"server_tutorial",releaseRepo:"libs-release",snapshotRepo:"libs-release")*/
            }
        }

        stage('Build Maven') {
            steps {
                configFileProvider([configFile(fileId: 'MavenArtifactorySettingId', variable: 'MAVEN_SETTINGS_XML')]) {
                    retry(count: 3) {
                        rtMavenRun(
                            tool: "Maven 3.6.2",//Tool name from Jenkins configuration
                            pom: 'pom.xml',
                            goals: '-U -s $MAVEN_SETTINGS_XML clean install',
                            deployerId: "MAVEN_DEPLOYER"/*,
                            resolverId: "MAVEN_RESOLVER"*/
                        )
                    }
                }
            }
        }
        stage('Publish to Artifactory') {
            steps {
                rtPublishBuildInfo(serverId: "server_tutorial")
            }
        }
    }
}
