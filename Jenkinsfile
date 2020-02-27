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
        RPOJECT_NAME="argodoc"
    }
    stages {
        stage('Build') {
            parallel {
                stage ('Get web-api docs'){
                    steps {
                        dir ("${WORKSPACE}/argo-web-api") {
                            git branch: 'devel',
                                credentialsId: 'jenkins-rpm-repo',
                                url: 'git@github.com:ARGOeu/argo-web-api.git'
                            sh """
                                cd ${WORKSPACE}/argo-web-api/doc/v2
                                mkdocs build --clean
                                cp -R ${WORKSPACE}/argo-web-api/doc/doc/ ${WORKSPACE}/argodoc/content/guides
                                cp -R ${WORKSPACE}/argo-web-api/doc/v2/site ${WORKSPACE}/argodoc/api/v2
                                cd ${WORKSPACE}/argodoc
                                git status
                                ls -l content/guides
                                ls -l api/v2
                                #git status | grep -qF 'working directory clean' || (git add -A &&  git commit -a --author="GRNET CI <>" -m "Update argo-web-api docs" && git push -f origin devel)
                            """
                            deleteDir()
                        }
                    }
                }
                // stage ('Get messaging docs'){
                //     steps {
                        
                //     }
                // }
            }
        }
        // stage('Deploy mkdocs') {
        //     steps {
        //         when {
        //             branch "master"
        //         }
        //         echo 'Deploying mkdocs...'
        //         sh """
        //         cd ${WORKSPACE}/${PROJECT_DIR} && make sources
        //         cp ${WORKSPACE}/${PROJECT_DIR}/argo-web-api*.tar.gz /home/jenkins/rpmbuild/SOURCES/
        //         if [ "$env.BRANCH_NAME" != "master" ]; then
        //             sed -i 's/^Release.*/Release: %(echo $GIT_COMMIT_DATE).%(echo $GIT_COMMIT_HASH)%{?dist}/' ${WORKSPACE}/${PROJECT_DIR}/${PROJECT_DIR}.spec
        //         fi
        //         cd /home/jenkins/rpmbuild/SOURCES && tar -xzvf ${PROJECT_DIR}*.tar.gz
        //         cp ${WORKSPACE}/${PROJECT_DIR}/${PROJECT_DIR}.spec /home/jenkins/rpmbuild/SPECS/
        //         rpmbuild -bb /home/jenkins/rpmbuild/SPECS/*.spec
        //         rm -f ${WORKSPACE}/*.rpm
        //         cp /home/jenkins/rpmbuild/RPMS/**/*.rpm ${WORKSPACE}/
        //         """
        //         archiveArtifacts artifacts: '**/*.rpm', fingerprint: true
        //         script {
        //             if ( env.BRANCH_NAME == 'master' ) {
        //                 echo 'Uploading rpm for devel...'
        //                 withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-repo', usernameVariable: 'REPOUSER', \
        //                                                         keyFileVariable: 'REPOKEY')]) {
        //                     sh  '''
        //                         scp -i ${REPOKEY} -o StrictHostKeyChecking=no ${WORKSPACE}/*.rpm ${REPOUSER}@rpm-repo.argo.grnet.gr:/repos/ARGO/prod/centos7/
        //                         ssh  -i ${REPOKEY} -o StrictHostKeyChecking=no ${REPOUSER}@rpm-repo.argo.grnet.gr createrepo --update /repos/ARGO/prod/centos7/
        //                         '''
        //                 }
        //             }
        //             else if ( env.BRANCH_NAME == 'devel' ) {
        //                 echo 'Uploading rpm for devel...'
        //                 withCredentials(bindings: [sshUserPrivateKey(credentialsId: 'jenkins-repo', usernameVariable: 'REPOUSER', \
        //                                                             keyFileVariable: 'REPOKEY')]) {
        //                     sh  '''
        //                         scp -i ${REPOKEY} -o StrictHostKeyChecking=no ${WORKSPACE}/*.rpm ${REPOUSER}@rpm-repo.argo.grnet.gr:/repos/ARGO/devel/centos7/
        //                         ssh -i ${REPOKEY} -o StrictHostKeyChecking=no ${REPOUSER}@rpm-repo.argo.grnet.gr createrepo --update /repos/ARGO/devel/centos7/
        //                         '''
        //                 }
        //             }
        //         }
        //     }
        // } 
    }
}