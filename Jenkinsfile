pipeline {
    environment {
        ANSIBLE_SERVER = "20.111.8.195"
    }
    agent any
    stages {

        stage('provision ec2 instances') {
            environment {
                AWS_ACCESS_KEY_ID = credentials("jenkins_aws_access_key_id")
                AWS_SECRET_ACCESS_KEY = credentials("jenkins_aws_secret_access_key")
            }
            steps {
                script {
                    dir('terraform') {
                        sh "terraform init"
                        sh "terraform apply --auto-approve"
                        NGINX_PUBLIC_IP = sh(script: "terraform output nginx_public_ip",returnStdout: true).trim()
                        NODE1_PUBLIC_IP = sh(script: "terraform output server_one_public_ip",returnStdout: true).trim()
                        NODE2_PUBLIC_IP = sh(script: "terraform output server_two_public_ip",returnStdout: true).trim()
                    }
                }
            }
        }

        stage('manage the servers') {
            steps {
                script {
                    echo "copying the files to ansible server"
                    sh "envsubst < conf-nginx > nginx-config"
                    sshagent(['server-ssh-key']) {
                       sh "scp -o StrictHostKeyChecking=no nginx-config ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                       sh "scp -o StrictHostKeyChecking=no deploy-nginx.yaml ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                       sh "scp -o StrictHostKeyChecking=no deploy-nodeapp.yaml ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                       sh "scp -o StrictHostKeyChecking=no inventory-aws_ec2.yaml ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                       sh "scp -o StrictHostKeyChecking=no ansible.cfg ${ANSIBLE_SERVER}:/home/zikou/ansible-nginx"
                    }

                    echo "calling ansible playbook to configure nginx server"
                    def remote= [:]
                    remote.name = "ansible-server"
                    remote.host = "20.111.8.195"
                    remote.allowAnyHosts = true
                    withCredentials([sshUserPrivateKey(credentialsId:'ansible-server-key',keyFileVariable: 'keyfile',usernameVariable:'user')]){
                        remote.user = user
                        remote.identityFile = keyfile
                        sshCommand remote: remote, command: "cd ansible-nginx && sudo ansible-playbook deploy-nodeapp.yaml && sudo ansible-playbook deploy-nginx.yaml "
                    }
                }
            }
        }

    }
}