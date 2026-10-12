pipeline {
    agent any

    parameters {
        string(name: 'VM_IP', defaultValue: '', description: 'IP of the target VM')
    }

    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Syntax Check') {
            steps { sh 'ansible-playbook nginx.yml --syntax-check -i "localhost,"' }
        }

        stage('Ping VM') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'vm-ssh',
                                                  usernameVariable: 'VM_USER',
                                                  passwordVariable: 'VM_PASS')]) {
                    sh 'ansible all -i "$VM_IP," -u "$VM_USER" -e ansible_password="$VM_PASS" -m ping'
                }
            }
        }

        stage('Install nginx') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'vm-ssh',
                                                  usernameVariable: 'VM_USER',
                                                  passwordVariable: 'VM_PASS')]) {
                    sh '''ansible-playbook nginx.yml -i "$VM_IP," -u "$VM_USER" \
                          -e ansible_password="$VM_PASS" \
                          -e ansible_become_password="$VM_PASS"'''
                }
            }
        }
    }

    post {
        success { echo "nginx installed and running on ${params.VM_IP}" }
        failure { echo "Pipeline failed, check Console Output" }
    }
}
