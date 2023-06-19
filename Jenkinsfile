pipeline
        {
            agent any
            
                    environment
                    {
        
                        DEV_SERVER_IP = '13.233.79.155';
                        QA_SERVER_IP = '15.206.149.143';
                        Azure_IP = '4.240.82.63';
                    }
                stages
                {
                    stage('Dev Deployment')
                    {
                        steps
                        {
                            sshagent(['dev-ssh-credentials'])
                            {
                                   sh '''
                                    ssh -o StrictHostKeyChecking=no ubuntu@${DEV_SERVER_IP} '
                                   
                                        sudo fuser -k 80/tcp
                                   
                                        git clone https://github.com/mithran14/jenkins-pipeline
                                        
                                        cd /home/ubuntu/jenkins-pipeline/leafhub
                                        
                                        git pull 
                                        
                                            // Get the commit id --> store it !
                                            // Alternate appach: Make your build with the build number 
                                            // And move to the nexus repo --> Ask your next server
                                        export COMMIT_ID = $(git rev-parse HEAD)
                                        nohup sudo mvn spring-boot:run > /dev/null 2>&1 & 
                                    '
                                    '''
                            }
                   
                        }
                    }
                   stage('Run Health check')
                    {
                        steps
                        {
                            sshagent(['dev-ssh-credentials'])
                            {
                                   sh '''
                                    ssh -o StrictHostKeyChecking=no ubuntu@${DEV_SERVER_IP} '
                                    cd /home/ubuntu
                                    sh health-check_live.sh
                                    '
                                    '''
                            }
                   
                        }
                    }
                    
                    
                    stage('smoke against Dev')
                    {
                        steps
                        {
                            sshagent(['azure'])
                            {
                                   sh '''
                                    ssh -o StrictHostKeyChecking=no azureuser@${Azure_IP} '
                                    
                                     # Update the package lists for upgrades and new package installations
                                    sudo apt-get update

                                    # Install OpenJDK 8
                                    sudo apt-get install -y openjdk-8-jdk

                                    # Install Maven
                                    sudo apt-get install -y maven
                                    
                                    git clone https://github.com/mithran14/webdriver-leafhub
                                    
                                    cd webdriver-leafhub

                                    # pull the changes if any
                                    git pull

                                    # Run Maven tests
                                    mvn clean test -DsuiteXmlFile=smoke.xml -Dserver.ip=13.233.79.155
                                    
                                    
                                    
                                    '
                                    '''
                            }
                   
                        }
                    }
                 
        stage('QA Deployment')
        {
            steps
            {
               sshagent(['dev-ssh-credentials'])
               {
                   sh '''
                   ssh -o StrictHostKeyChecking=no ubuntu@${QA_SERVER_IP} '
                   
                        sudo fuser -k 80/tcp
                   
                        git clone https://github.com/mithran14/jenkins-pipeline
                            
                                # Move to the webdriver-tests folder
                                #cd webdriver-leafhub
                        
                        cd /home/ubuntu/jenkins-pipeline/leafhub
                        
                        git checkout $COMMIT_ID 
                        
                        nohup sudo mvn spring-boot:run > /dev/null 2>&1 & 
                   
                        '
                        '''
               }
               
            }
        }
        
        stage('Regression Tests on QA')
                    {
                        steps
                        {
                            sshagent(['azure'])
                            {
                                   sh '''
                                    ssh -o StrictHostKeyChecking=no azureuser@${Azure_IP} '
                                    
                                     # Update the package lists for upgrades and new package installations
                                    sudo apt-get update

                                    # Install OpenJDK 8
                                    sudo apt-get install -y openjdk-8-jdk

                                    # Install Maven
                                    sudo apt-get install -y maven
                                    
                                    git clone https://github.com/mithran14/webdriver-leafhub
                                    
                                    cd webdriver-leafhub

                                    # pull the changes if any
                                    git pull

                                    # Run Maven tests
                                    mvn clean test -DsuiteXmlFile=testng-parallel.xml -Dserver.ip=15.206.149.143
                                    
                                    yes | sudo apt install awscli                                                       << EOF
aws configure set aws_access_key_id AKIA2ZLFKTOAP62TKPOV
aws configure set aws_secret_access_key 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
aws configure set region ap-south-1
aws configure set output json

# Push the results to S3 (make sure to install and configure awsconfigure before)
#aws s3 rb s3://reports3004-html-selenium3004 --force
aws s3api create-bucket --bucket regressionqq23 --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1
aws s3 sync reports/ s3://regressionqq23
                                    
                                    '
                                    '''
                            }
                   
                        }
                    }
        
        
    }
    
}
                                        
