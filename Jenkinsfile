pipeline {
    agent { 
        docker { 
            image 'argo.registry:5000/debian-jessie-argodoc'
            label 'slave01'
        }
    }
    options {
        checkoutToSubdirectory('argodoc')
        newContainerPerStage()
    }
    environment {
        DOC_PROJECT="argodoc"
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
                                credentialsId: 'jenkins-master',
                                url: "git@github.com:kevangel79/argodoc.git"
                            sh '''
                                whoami
                                touch aaaaa
                                ls -l
                                git branch
                                if [ -n "$(git status --porcelain)" ]; then
                                    git checkout devel
                                    git add -A
                                    git config --global user.email "argo@grnet.gr"
                                    git config --global user.name "newgrnetci"
                                    git commit -a --author="newgrnetci <argo@grnet.gr>" -m "Update docs" 
                                fi
                                echo "pushed"
                                echo ssh -i $SSH_KEY -l git -o StrictHostKeyChecking=no \\"\\$@\\" > local_ssh.sh
                                chmod +x local_ssh.sh
                                export GIT_SSH=./local_ssh.sh
                                git push origin devel
                            '''
                            withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-master', keyFileVariable: 'SSH_KEY')]) {
                                sh 'echo ssh -i $SSH_KEY -l git -o StrictHostKeyChecking=no \\"\\$@\\" > local_ssh.sh'
                                sh 'chmod +x local_ssh.sh'
                                withEnv(['GIT_SSH=./local_ssh.sh']) {
                                    sh 'git push origin devel'
                                }
                            }
                            sshagent(credentials:['jenkins-master']) {
                                sh ('''
                                    export GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no"
                                    git push origin devel
                                ''')
                            }
                            withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-master', keyFileVariable: 'GITHUB_KEY')]) {
                                //withEnv(["GIT_SSH_COMMAND=ssh -i $GITHUB_KEY -o StrictHostKeyChecking=no"]) {
                                    sh '''
                                        export GIT_SSH_COMMAND="ssh -i $GITHUB_KEY -o StrictHostKeyChecking=no"
                                        echo $GIT_SSH_COMMAND
                                        git remote -v
                                        pwd
                                        git branch
                                        git push origin devel
                                    '''
                                //}
                            }
                            sshagent(credentials:['jenkins-master']) {
                                sh ('''
                                    find ~/
                                    git push origin devel
                                ''')
                            }
                        }
                    }
                }
            }
        }
    }
}