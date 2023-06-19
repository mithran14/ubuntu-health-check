pipeline {
  agent any
  stages {
    stage('Dev Deployment') {
      steps {
        sh '''sshagent([\'dev-ssh-credentials\'])
                            {
                                   sh \'\'\'
                                    ssh -o StrictHostKeyChecking=no ubuntu@${DEV_SERVER_IP} \'
                                   
                                        sudo fuser -k 80/tcp
                                   
                                        git clone https://github.com/mithran14/jenkins-pipeline
                                        
                                        cd /home/ubuntu/jenkins-pipeline/leafhub
                                        
                                        git pull 
                                        
                                            // Get the commit id --> store it !
                                            // Alternate appach: Make your build with the build number 
                                            // And move to the nexus repo --> Ask your next server
                                        export COMMIT_ID = $(git rev-parse HEAD)
                                        nohup sudo mvn spring-boot:run > /dev/null 2>&1 & 
                                    \'
                                    \'\'\'
                            }'''
      }
    }

  }
}