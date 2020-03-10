pipeline {
    agent { 
        docker { 
            image 'argo.registry:5000/debian-jessie-argodoc'
            args '-u root'
        }
    }
    options {
        checkoutToSubdirectory('argodoc')
        newContainerPerStage()
    }
    environment {
        DOC_PROJECT="argodoc"
        GIT_COMMITTER_NAME="newgrnetci"
        GIT_COMMITTER_EMAIL="<argo@grnet.gr>"
        ARGOEU_URL=sh(script: "if [ \"$env.BRANCH_NAME\" == \"master\" ]; then echo \"git@github.com:ARGOeu/argoeu.github.io.git\"; else echo \"git@github.com:argoeu-devel/argoeu-devel.github.io.git\"; fi",returnStdout: true).trim()
    }
    stages {
        stage('Build') {
            //when { branch pattern: "master|devel", comparator: "REGEXP" }
            parallel {
                stage ('Build argo docs'){
                    steps {
                        dir ("${WORKSPACE}/kevangel_argodoc") {
                            git branch: "devel",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:kevangel79/argodoc.git"
                            sh """
                                cd ${WORKSPACE}/kevangel_argodoc
                                touch aaaaa
                                git branch 
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <argo@grnet.gr>" -m \"Update docs\"
                                fi
                            """
                            script {
                                sshagent (credentials: ['jenkins-rpm-repo']) {
                                    sh """
                                        cd ${WORKSPACE}/kevangel_argodoc
                                        export GIT_SSH_COMMAND=\"ssh -oStrictHostKeyChecking=no\"
                                        git push origin devel
                                    """
                                }
                            }
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
                        export GIT_SSH_COMMAND="ssh -oStrictHostKeyChecking=no"
                        if [ -n "\$(git status --porcelain)" ]; then
                            git add -A
                            git commit -a --author="newgrnetci <argo@grnet.gr>" -m "Update docs"
                            git push -f kevangel79 devel
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
                            git commit -a --author="newgrnetci <argo@grnet.gr>" -m "Update docs"
                            #git push origin master
                        fi
                    """
                }
            }
        } 
    }
}