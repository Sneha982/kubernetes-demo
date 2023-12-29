jenkinsfile for this project:

node{
    
    stage('Git checkout'){
        git branch: 'main', url: 'https://github.com/Sneha982/kubernetes-demo.git'
    }
    stage('sending dockerfile to ansible server over ssh'){
        sshagent(['ansible_demo']) {
          sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111'
          sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.3.111:/home/ubuntu '
      }
    }
    stage('Docker image build'){
        sshagent(['ansible_demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
            
        }
        
    }
    stage('Docker image tagging'){
        sshagent(['ansible_demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image tag $JOB_NAME:v1.$BUILD_ID snehagunda1/$JOB_NAME:v1.$BUILD_ID'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image tag $JOB_NAME:v1.$BUILD_ID snehagunda1/$JOB_NAME:latest'
            
        }
        
    }
    stage('Push docker images to dockerhub'){
        sshagent(['ansible_demo']) {
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker login -u snehagunda1 -p ${dockerhub_passwd}"
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image push snehagunda1/$JOB_NAME:v1.$BUILD_ID'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image push snehagunda1/$JOB_NAME:latest'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 docker image rm snehagunda1/$JOB_NAME:v1.$BUILD_ID snehagunda1/$JOB_NAME:latest $JOB_NAME:v1.$BUILD_ID'
                
             }
        }
        
    }
    stage('Copy files from jenkins server to kubernetes server'){
        sshagent(['kubernetes_server']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.44.79'
            sh 'scp /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.44.79:/home/ubuntu '
            
      }
        
    }
    stage('kubernetes deployment using ansible'){
        sshagent(['ansible_demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 cd /home/ubuntu'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.3.111 sudo ansible-playbook Ansible.yml'
        }
        
    }
    
    
}
