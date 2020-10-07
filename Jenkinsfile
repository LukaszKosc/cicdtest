def WORKSPACE_PATH = ""
def pods_created = ""
environment {

}
def run_cmd_via_ssh(user, host, cmd){
    echo "working"
}

pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                script {
                    def timestamp = sh (script: "date +'%Y%m%d_%H%M_%s'", returnStdout: true).trim()
                    WORKSPACE_PATH = "${WORKSPACE}/${timestamp}"
                }
                echo "Workspace: ${WORKSPACE_PATH}"
            }
        }
        stage('Prepare Workspace') {
            steps {
                script{
                    sh (script: "mkdir -p ${WORKSPACE_PATH}", returnStdout: true).trim()
                    // sh (script: "scp jenkins@192.168.2.15:/home/jenkins/repos/tools/deploy.yml ${WORKSPACE_PATH}")
                    echo "with timestamp: ${WORKSPACE_PATH}"
                }
            }
        }
        stage('Setup Pods') {
            steps {
                script {
                    sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl delete -f /home/jenkins/repos/tools/deploy.yml --ignore-not-found'")
                    sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl create -f /home/jenkins/repos/tools/deploy.yml'")
                    sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl wait --for=condition=ready pod -l app=nginx-pod-app'")
                    pods_created = sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl get all | grep Running | grep nginx'", returnStdout: true).trim()
                    echo "${pods_created}"
                }
            }
        }
        stage('Verify if service is up') {
            steps {

                script {
                    // sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl wait --for=condition=ready pod -l app=nginx-pod-app'")
                    def service_response = sh (script: "curl -s http://192.168.2.15:32333", returnStdout: true).trim()
                    echo "${service_response}"
                    if (service_response == "Hello from Kubernetes storage"){
                        currentBuild.result = 'SUCCESS'
                    } else {
                        currentBuild.result = 'FAIL'
                    }
                }
            }
        }
        stage('Teardown Pods') {
            steps {
                script {
                    echo "Teardown: "
                    sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl delete -f /home/jenkins/repos/tools/deploy.yml --ignore-not-found'")
                    pods_created = sh (script: "ssh jenkins@192.168.2.15 '/usr/bin/kubectl get deployments'", returnStdout: true).trim()
                    echo "${pods_created}"
                }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    sh (script: "rm -rf ${WORKSPACE_PATH}", returnStdout: true).trim()
                    echo "in cleanup: ${WORKSPACE_PATH}"
                }
            }
        }

    }
}
