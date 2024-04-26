node {
    def toolsDir = '/home/jenkins/tools'
    def discordWebHookUrl = 'https://discord.com/api/webhooks/1233451596676071527/iomVt3QPH4WLnWAO2hLmdKmW_QT-HgQpPyiQpxAWGic3wmztObLis33tHmPygCPbDX-_'
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
                stage('Nexus') {
                    echo "!!! TODO !!!"
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
            stage('Discord') {
                discordSend description: currentBuild.fullDisplayName,
                    footer: "Marco's CI/CD Labs",
                    link: "",
                    result: currentBuild.currentResult,
                    title: env.JOB_NAME,
                    webhookURL: discordWebHookUrl
            }
        )
    }
}
