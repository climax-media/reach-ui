#!groovy
@Library('jenkins-libs@master') _

def call() {
    pipeline {
        agent any
        stages {
            stage("Check build conditions") {
                environment {
                    PACKAGE_NAME = getPackageJsonName()
                    PACKAGE_VERSION = getPackageJsonVersion()
                    CI = true
                }
                when {
                    expression {
                        !isLastCommitRelease()
                    }
                }
                stages {
                    stage("Debug output") {
                        steps {
                            echo "Branch: ${env.BRANCH_NAME}"
                            echo "Version: ${env.PACKAGE_VERSION}"
                            echo "Tag: ${getDistTagFromVersion(env.PACKAGE_VERSION)}"
                        }
                    }
                    stage("Slack: Build started") {
                        steps {
                            slackSend color: '#5DADE2', message: "${env.PACKAGE_NAME}@${env.PACKAGE_VERSION} CI started in branch ${env.BRANCH_NAME}"
                        }
                    }
                    stage("Install npm packages") {
                        steps {
                            exec("npm install")
                        }
                    }
                    stage("Tests") {
                        steps {
                            exec("npm run bootstrap")
                        }
                    }
                    stage("Tests") {
                        steps {
                            exec("npm test -- --ci")
                        }
                    }
                    stage("Slack: Build succeeded") {
                        steps {
                            slackSend color: '#2ECC71', message: "${env.PACKAGE_NAME}@${PACKAGE_VERSION} in branch ${env.BRANCH_NAME} build passed!"
                        }
                    }
                    stage("Check release conditions") {
                        when {
                            expression {
                                isReleaseBranch()
                            }
                        }
                        stages {
                            stage("Slack: Release started") {
                                steps {
                                    slackSend color: '#5DADE2', message: "${env.PACKAGE_NAME} in branch ${env.BRANCH_NAME} publish started."
                                }
                            }
                            stage("Build") {
                                steps {
                                    exec("npm run build")
                                }
                            }
                            stage("Auto-version") {
                                steps {
                                    autoVersion()
                                }
                            }
                            stage("Push changes") {
                                steps {
                                    gitPush()
                                }
                            }
                            stage("Publish") {
                                steps {
                                    npmPublish()
                                }
                            }
                            stage("Slack: Release succeeded") {
                                steps {
                                    slackSend color: '#2ECC71', message: "${env.PACKAGE_NAME}@${getPackageJsonVersion()} in branch ${env.BRANCH_NAME} publish succeeded!"
                                }
                            }
                        }
                    }
                }
            }
        }
        post {
            failure {
                slackSend color: '#E74C3C', message: "${getPackageJsonName()}@${getPackageJsonVersion()} in branch ${env.BRANCH_NAME} CI failed."
            }
            cleanup {
                deleteDir()
            }
        }
    }
}
