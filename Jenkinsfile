node {
    def toolsDir = '/home/jenkins/tools'
    def discordWebHookUrl = 'https://discord.com/api/webhooks/1233451596676071527/iomVt3QPH4WLnWAO2hLmdKmW_QT-HgQpPyiQpxAWGic3wmztObLis33tHmPygCPbDX-_'
    sh 'printenv | sort -h'
    stage('Check Git') {
        sh 'git --version'
    }
    stage('Check Docker') {
        sh 'docker --version'
    }
    stage('Check Build Tools') {
        parallel(
            'Java': {
                stage('Java') {
                    sh 'java --version'
                }
            },
            'Maven': {
                stage('Maven') {
                    def toolLocation = tool 'Maven'
                    withEnv(["MVN_HOME=$toolLocation"]) {
                        sh '"$MVN_HOME/bin/mvn" --version'
                    }
                }
            },
            'Gradle': {
                stage('Check Gradle') {
                    def toolLocation = tool 'Gradle'
                    withEnv(["GRADLE_HOME=$toolLocation"]) {
                        sh '"$GRADLE_HOME/bin/gradle" --version'
                    }
                }
            },
            'NPM': {
                stage('NPM') {
                    sh '. ~/.profile && npm --version'
                }
            }
        )
    }
    stage('Check Publishing Capabilities') {
        parallel(
            'Liquibase': {
                stage('Liquibase') {
                    def toolLocation = "$toolsDir/liquibase-4.26.0"
                    sh "$toolLocation/liquibase --version"
                }
            },
            'Nexus': {
                stage('Nexus Maven') {
                    def toolLocation = tool 'Maven'
                    sh 'echo test > test.jar'
                    withEnv(["MVN_HOME=$toolLocation"]) {
                        sh '"$MVN_HOME/bin/mvn" deploy:deploy-file -DgroupId=org.cicdlabs -DartifactId=test -Dversion=0.0.0 -Dfile=test.jar -Dpackaging=jar -DgeneratePom=true -DrepositoryId=cicdlab -Durl=http://artifactsrepo:8081/nexus/repository/maven-public/'
                    }
                    sh 'curl -X DELETE http://artifactsrepo:8081/nexus/service/rest/v1/components/org.cicdlabs%3Atest%3A0.0.0 -H "accept: application/json"'
                }
            }
        )
    }
    stage('Check Scanning Tools') {
        parallel(
            'Dependency Check': {
                stage('Dependency Check') {
                    echo 'Dependency Check is used to find vulnerabilities in open source dependencies'
                    def toolLocation = tool 'Dependency Check'
                    sh "$toolLocation/bin/dependency-check.sh --version"
                }
            },
            'Trivy': {
                stage('Trivy') {
                    echo 'Trivy is used to find vulnerabilities in container images'
                    sh 'trivy --version'
                }
            },
            'Gitleaks': {
                stage('Gitleaks') {
                    echo 'Gitleaks is used to find secrets that should not be committed to the code repository'
                    sh 'gitleaks version'
                }
            }
        )
    }
    stage('Check Notification Capabilities') {
        parallel(
            'Discord': {
                stage('Discord') {
                    discordSend description: "Build Notification Testing _(Builder ${env.HOSTNAME})_",
                        footer: "Marco's CI/CD Lab",
                        link: "",
                        result: currentBuild.currentResult,
                        title: currentBuild.fullDisplayName,
                        webhookURL: discordWebHookUrl
                }
            }
        )
    }
}
