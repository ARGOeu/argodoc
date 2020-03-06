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
            when { branch pattern: "master|devel", comparator: "REGEXP" }
            parallel {
                stage ('Build web-api docs'){
                    environment {
                        DOC_SOURCE="argo-web-api"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v2
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/doc/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v2/site/ ${WORKSPACE}/${DOC_PROJECT}/api/v2
                                cd ${WORKSPACE}/${DOC_PROJECT}
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update ${DOC_SOURCE} docs"
                                    #git push origin "${env.BRANCH_NAME}"
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
                stage ('Build messaging docs'){
                    environment {
                        DOC_SOURCE="argo-messaging"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/doc/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/messaging/v1
                                cd ${WORKSPACE}/${DOC_PROJECT}
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update ${DOC_SOURCE} docs"
                                    #git push origin "${env.BRANCH_NAME}"
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
                stage ('Build authn docs'){
                    environment {
                        DOC_SOURCE="argo-api-authn"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/docs/v1
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/docs/v1/docs/ ${WORKSPACE}/${DOC_PROJECT}/content/guides
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/docs/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/authn/v1
                                cd ${WORKSPACE}/${DOC_PROJECT}
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <>" -m "Update ${DOC_SOURCE} docs"
                                    #git push origin "${env.BRANCH_NAME}"
                                fi
                            """
                            //deleteDir()
                        }
                    }
                }
                stage ('Build argo-ams-library docs'){
                    environment {
                        DOC_SOURCE="argo-ams-library"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
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
            when { branch pattern: "master|devel", comparator: "REGEXP" }
            steps {
                echo 'Deploying mkdocs...'
                dir ("${WORKSPACE}/argoeu") {
                    git branch: "${env.BRANCH_NAME}",
                        credentialsId: 'jenkins-rpm-repo',
                        url: "${ARGOEU_URL}"
                    sh """
                        rm -rf ${WORKSPACE}/argoeu/api
                        rm -rf ${WORKSPACE}/argoeu/messaging
                        rm -rf ${WORKSPACE}/argoeu/authn
                        rm -rf ${WORKSPACE}/argoeu/ams-library/*
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/api ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/messaging ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/authn ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/argo-ams-library/_build/ ${WORKSPACE}/argoeu/ams-library
                        cd ${WORKSPACE}/argoeu
                        if [ -n "\$(git status --porcelain)" ]; then
                            git add -A
                            git commit -a --author="newgrnetci <>" -m "Update docs"
                            #git push origin master
                        fi
                    """
                    //deleteDir()
                }
            }
        } 
    }
}