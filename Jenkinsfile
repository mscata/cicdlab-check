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
    stage('Check DB Migration Capabilities') {
        parallel(
            'Liquibase': {
                stage('Liquibase') {
                    def toolLocation = "$toolsDir/liquibase-4.29.2"
                    echo "Testing Postgres connection"
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'POSTGRES_CREDENTIALS', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        wrap([$class: 'MaskPasswordsBuildWrapper', varPasswordPairs: [[password: env.PASSWORD]]]) {
                            retry(3) {
                                sh "$toolLocation/liquibase execute-sql --username $USERNAME --password $PASSWORD --sql='SELECT NOW()' --url=jdbc:postgresql://dbserver:5432/postgres"
                                sleep(time: 10, unit: 'SECONDS')
                            }
                        }
                    }
                }
            }
        )
    }
    stage('Check Artifact Publishing Capabilities') {
        parallel(
            'Nexus': {
                stage('Nexus Maven') {
                    def toolLocation = tool 'Maven'
                    echo "Testing connection to Maven Central"
                    withEnv(["MVN_HOME=$toolLocation"]) {
                        def v=3.8.1
                        sh '"$MVN_HOME/bin/mvn" dependency:$v:get -Dartifact=org.apache.maven.plugins:maven-dependency-plugin:$v -Drepo.user=$USERNAME -Drepo.password=$PASSWORD'
                    }
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
                    echo 'Trivy is used to find vulnerabilities in code, dependencies, and container images'
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
