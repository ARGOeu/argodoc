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
        GIT_COMMITTER_EMAIL="<argo@grnet.gr>"
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
                            git branch: "${env.BRANCH_NAME}",
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
                            git branch: "${env.BRANCH_NAME}",
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
                stage ('Build poem-2 docs'){
                    environment {
                        DOC_SOURCE="poem-2"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v1/
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/poem/v1
                            """
                            deleteDir()
                        }
                    }
                }
                stage ('Build monitoring-probes docs'){
                    environment {
                        DOC_SOURCE="monitoring-probes"
                    }
                    steps {
                        dir ("${WORKSPACE}/${DOC_SOURCE}") {
                            git branch: "${env.BRANCH_NAME}",
                                credentialsId: 'jenkins-rpm-repo',
                                url: "git@github.com:ARGOeu/${DOC_SOURCE}.git"
                            sh """
                                cd ${WORKSPACE}/${DOC_SOURCE}/doc/v1/
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/${DOC_SOURCE}/doc/v1/site/* ${WORKSPACE}/${DOC_PROJECT}/monitoring-probes/v1
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
        stage('Push changes to argodoc') {
            when { branch pattern: "master|devel", comparator: "REGEXP" }
            steps {
                dir ("${WORKSPACE}/argodoc2") {
                    git branch: "${env.BRANCH_NAME}",
                        credentialsId: 'jenkins-rpm-repo',
                        url: "git@github.com:ARGOeu/argodoc.git"
                    sh """
                        rm -rf $WORKSPACE/argodoc2/content/guides/*
                        rm -rf $WORKSPACE/argodoc2/api/v2/*
                        rm -rf $WORKSPACE/argodoc2/messaging/v1/*
                        rm -rf $WORKSPACE/argodoc2/authn/v1/*
                        rm -rf $WORKSPACE/argodoc2/poem/v1/*
                        rm -rf $WORKSPACE/argodoc2/monitoring-probes/v1/*
                        cp -R $WORKSPACE/argodoc/content/guides/* $WORKSPACE/argodoc2/content/guides/
                        cp -R $WORKSPACE/argodoc/api/v2/* $WORKSPACE/argodoc2/api/v2/
                        cp -R $WORKSPACE/argodoc/messaging/v1/* $WORKSPACE/argodoc2/messaging/v1/
                        cp -R $WORKSPACE/argodoc/authn/v1/* $WORKSPACE/argodoc2/authn/v1/
                        cp -R $WORKSPACE/argodoc/poem/v1/* $WORKSPACE/argodoc2/poem/v1/
                        cp -R $WORKSPACE/argodoc/monitoring-probes/v1/* $WORKSPACE/argodoc2/monitoring-probes/v1/
                        cd $WORKSPACE/argodoc2
                    """
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-master', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            echo ssh -i $SSH_KEY -l git -o StrictHostKeyChecking=no \\"\\$@\\" > ../local_ssh.sh
                            chmod +x ../local_ssh.sh
                        '''
                        withEnv(['GIT_SSH=../local_ssh.sh']) {
                            sh """
                                GIT_COMMIT_TIME=$(git log -1 --format=%ct)
                                CURR_TIME=$(date +%s)
                                SUB="$(($CURR_TIME - $GIT_COMMIT_TIME))"
                                if [[ $SUB -lt 600 ]]; then
                                    echo ">>> Trigger loop abort push commit"
                                elif [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <argo@grnet.gr>" -m "Update docs"
                                    git push origin ${env.BRANCH_NAME}
                                fi
                                rm ../local_ssh.sh
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
                    git branch: "master",
                        credentialsId: 'jenkins-rpm-repo',
                        url: "${ARGOEU_URL}"
                    sh """
                        rm -rf ${WORKSPACE}/argoeu/api
                        rm -rf ${WORKSPACE}/argoeu/messaging
                        rm -rf ${WORKSPACE}/argoeu/authn
                        rm -rf ${WORKSPACE}/argoeu/poem
                        rm -rf ${WORKSPACE}/argoeu/monitoring-probes
                        rm -rf ${WORKSPACE}/argoeu/ams-library/*
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/api ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/messaging ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/authn ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/poem ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/${DOC_PROJECT}/monitoring-probes ${WORKSPACE}/argoeu
                        cp -R ${WORKSPACE}/argo-ams-library/documentation/_build/html/* ${WORKSPACE}/argoeu/ams-library
                        cd ${WORKSPACE}/argoeu
                        git checkout master
                    """
                    withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-master', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                            echo ssh -i $SSH_KEY -l git -o StrictHostKeyChecking=no \\"\\$@\\" > ../local_ssh.sh
                            chmod +x ../local_ssh.sh
                        '''
                        withEnv(['GIT_SSH=../local_ssh.sh']) {
                            sh """
                                if [ -n "\$(git status --porcelain)" ]; then
                                    git add -A
                                    git commit -a --author="newgrnetci <argo@grnet.gr>" -m "Update docs"
                                    git push origin master
                                fi
                                rm ../local_ssh.sh
                            """
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}