pipeline {
    agent { 
        docker { 
            image 'argo.registry:5000/debian-jessie-argodoc' 
        }
    }
    options {
        checkoutToSubdirectory('argodoc')
        newContainerPerStage()
    }
    environment {
        DOC_PROJECT="argodoc"
        GIT_COMMITTER_NAME="newgrnetci"
        GIT_COMMITTER_EMAIL="<>"
        ARGOEU_URL=sh(script: "if [ \"$env.BRANCH_NAME\" == \"master\" ]; then echo \"git@github.com:ARGOeu/argoeu.github.io.git\"; else echo \"git@github.com:argoeu-devel/argoeu-devel.github.io.git\"; fi",returnStdout: true).trim()
    }
    stages {
        stage('Build') {
            //when { branch pattern: "master|devel", comparator: "REGEXP" }
            parallel {
                stage ('Build web-api docs'){
                    environment {
                        DOC_SOURCE="argo-web-api"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "devel",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v2
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/doc/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v2/site/ ${WORKSPACE}/${DOC_PROJECT}/api/v2
                            """
                            deleteDir()
                        }
                    }
                }
                stage ('Build messaging docs'){
                    environment {
                        DOC_SOURCE="argo-messaging"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "devel",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/doc/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/messaging/v1
                            """
                            deleteDir()
                        }
                    }
                }
                stage ('Build authn docs'){
                    environment {
                        DOC_SOURCE="argo-api-authn"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "devel",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/docs/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/docs/v1/docs/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/docs/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/authn/v1
                            """
                            deleteDir()
                        }
                    }
                }
                stage ('Build argo-ams-library docs'){
                    environment {
                        DOC_SOURCE="argo-ams-library"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "devel",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/documentation
                                make html
                            """
                        }
                    }
                }
            }
        }
        stage('Deploy mkdocs') {
            //when { branch pattern: "master|devel", comparator: "REGEXP" }
            steps {
                echo 'Deploying mkdocs...'
                dir ("${WORKSPACE}/argoeu") {
                    git branch: "master",
                        credentialsId: 'jenkins-rpm-repo',
                        url: "${ARGOEU_URL}"
                    sh """
                        cd ${WORKSPACE}/argodoc
                        git remote add kevangel79 git@github.com:kevangel79/argodoc.git
                        if [ -n "\$(git status --porcelain)" ]; then
                            git add -A
                            git commit -a --author="newgrnetci <>" -m "Update docs"
                            git push kevangel79 HEAD:${env.BRANCH_NAME}
                        fi
                        rm -rf ${WORKSPACE}/argoeu/api
                        rm -rf ${WORKSPACE}/argoeu/messaging
                        rm -rf ${WORKSPACE}/argoeu/authn
                        rm -rf ${WORKSPACE}/argoeu/ams-library/*
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/api ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/messaging ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/authn ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/argo-ams-library/documentation/_build/ ${WORKSPACE}/argoeu/ams-library
                        cd ${WORKSPACE}/argoeu
                        if [ -n "\$(git status --porcelain)" ]; then
                            git add -A
                            git commit -a --author="newgrnetci <>" -m "Update docs"
                            #git push origin master
                        fi
                    """
                }
            }
        } 
    }
    post {
        failure {
            sh "rm -rf ${WORKSPACE}"
        }
    }
}