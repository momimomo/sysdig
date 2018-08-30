pipeline {
    environment {
        // credential UUID of github repo ssh key. Need to refresh if the github ssh key is changed.
        SSHAGENT_CRED_ID='4399087a-3e99-41e5-9dbe-a70a554672c8'
    }

    parameters {
        string(name: 'PROBE_TYPE', description: 'probe to build; could be sysdig, sysdigcloud-probe, falco', defaultValue: 'sysdig-probe')
        string(name: 'TARGET_TAG', description: 'tag of git repo', defaultValue: '0.23.1')
    }

    agent { label 'probe-builder-test' }
    stages {
        stage ('preparation') {
            steps {
                sh 'hostname'
                sh 'uname -a'
                sh 'gcc --version -v'
                sh 'g++ --version -v'
                sh 'pwd -P'
                sh 'df -h'
                dir('sysdig') { checkout scm }
                sh 'pwd -P; df -h'
                sh 'ls -l'
                sh 'echo build dokcer images of various builders ...'
                sh 'sysdig/scripts/build-builder-containers.sh'
                sh 'docker images'
                sh 'docker ps -a'
                sh 'docker ps -aq -f "name=fedora-atomic-build|amazon-linux-build|ol6-buildol7-build" | xargs --no-run-if-empty docker rm -f'
                sh 'docker ps -a'
                sh 'mkdir -p probe/output'
                sh 'echo ready for probe build: ${PROBE_TYPE}, code tag ${TARGET_TAG}'
            }
        }

        stage ('compilation') {
            steps {
                parallel (
                    "info" : { sh 'echo "git repo branch: ${BRANCH_NAME}" && pwd -P && df -h' },
                    "ubuntu" 		    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/ubuntu        && cd probe/ubuntu        && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Ubuntu && cp -u output/*${TARGET_TAG}* ../output/ && echo ubuntu finished'
                        }
                    },
                    "debian" 		    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/debian        && cd probe/debian        && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Debian && cp -u output/*${TARGET_TAG}* ../output/ && echo debian finished'
                        }
                    },
                    "rhel"   		    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/rhel          && cd probe/rhel          && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable RHEL && cp -u output/*${TARGET_TAG}* ../output/ && echo rhel finished'
                        }
                    },
                    "fedora & fedora-atomic" 	        : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/fedora && cd probe/fedora && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Fedora && cp -u output/*${TARGET_TAG}* ../output/ && echo fedora finished'
                            sh 'mkdir -p probe/fedora && cd probe/fedora && docker run -i --rm --name fedora-atomic-build -v ${PWD}:/build/probe fedora-builder ${PROBE_TYPE} ${TARGET_TAG} stable Fedora-Atomic && cp -u output/*${TARGET_TAG}* ../output/ && echo fedora-atomic finished'
                        }
                    },
                    "coreos" 		    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/coreos        && cd probe/coreos        && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable CoreOS && cp -u output/*${TARGET_TAG}* ../output/ && echo coreos finished'
                        }
                    },
                    "boot2docker" 	    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/boot2docker   && cd probe/boot2docker   && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable boot2docker && cp -u output/*${TARGET_TAG}* ../output/ && echo boot2docker finished'
                        }
                    },
                    "oracle_rhck" 	    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/oracle_rhck   && cd probe/oracle_rhck   && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Oracle_RHCK && cp -u output/*${TARGET_TAG}* ../output/ && echo oracle_rhck finished'
                        }
                    },
                    "oracle_linux6_uek" : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/oracle_linux6_uek && cd probe/oracle_linux6_uek && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Oracle_Linux_6_UEK && cp -u output/*${TARGET_TAG}* ../output/ && echo oracle_linux6_uek finished'
                        }
                    },
                    "oracle_linux7_uek" : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/oracle_linux7_uek && cd probe/oracle_linux7_uek && bash -x ../../sysdig/scripts/build-probe-binaries ${PROBE_TYPE} ${TARGET_TAG} stable Oracle_Linux_7_UEK && cp -u output/*${TARGET_TAG}* ../output/ && echo oracle_linux7_uek finished'
                        }
                    },
                    "amazon_linux" 	    : {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/amazon_linux  && cd probe/amazon_linux  && docker run -i --rm --name amazon-linux-build -v ${PWD}:/build/probe fedora-builder ${PROBE_TYPE} ${TARGET_TAG} stable AmazonLinux && cp -u output/*${TARGET_TAG}* ../output/ && echo amazon_linux finished'
                        }
                    },
                    "amazon_linux2" 	: {
                        sshagent(["$SSHAGENT_CRED_ID"]) {
                            sh 'mkdir -p probe/amazon_linux2 && cd probe/amazon_linux2 && docker run -i --rm --name amazon-linux2-build -v ${PWD}:/build/probe fedora-builder ${PROBE_TYPE} ${TARGET_TAG} stable AmazonLinux2 && cp -u output/*${TARGET_TAG}* ../output/ && echo amazon_linux2 finished'
                        }
                    },
                )
            }
        }
    
        stage ('s3 publishing') {
	    when { branch "${BRANCH_NAME}" }
            steps {
                sh 'hostname'
                sh 'uname -a'
                sh 'pwd -P'
                sh 'echo workspace = $WORKSPACE'
                sh 'df -h'
                sh 'ls -l probe/output/'
                sh 'echo "number of total files = $(ls -l probe/output/ | wc -l)" '
                // in test env, only one executor is available, hence impossible to wait.
                build job: "test-publish-probe-modules", propagate: false, wait: false, parameters: [ string(name: 'WDIR', value: "${WORKSPACE}") ]
                sh ' echo probe modules publish job started' 
            }
        }
    }
}
