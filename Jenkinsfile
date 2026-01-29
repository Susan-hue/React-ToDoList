pipeline {
    agent any

    stages {

        stage("Checkout code") {
            steps {
                git branch: 'peter-branch', url: 'https://github.com/bigcephas1/React-ToDoList.git'
            }
        }

        stage("Build image and push") {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'docker_creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')
                ]) {
                    sh '''
                        docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                        docker build -t $DOCKER_USERNAME/ci_backend_full_pipeline:v1 -f backend/Dockerfile backend
                        docker build -t $DOCKER_USERNAME/ci_frontend_full_pipeline:v1 -f dive-react-app/Dockerfile dive-react-app
                        docker push $DOCKER_USERNAME/ci_backend_full_pipeline:v1
                        docker push $DOCKER_USERNAME/ci_frontend_full_pipeline:v1
                    '''
                }
            }
        }

        stage("Implement Terraform") {
            
            steps {
                withAWS(credentials: 'AWS_CREEDS', region: 'us-east-1') {

                    withCredentials([
                        file(credentialsId: 'PUBKEY_FILE',  variable: 'PUBKEY_FILE'),
                        file(credentialsId: 'PRIVKEY_FILE', variable: 'PRIVKEY_FILE')
                    ]) {

                        dir("terraform/modules") {
                            sh """
                                echo "AWS credentials loaded into environment"
                               # Copy SSH keys for EC2 provisioning
                                 cp "${PUBKEY_FILE}" ec2-modules/my_key.pub
                                 cp "${PRIVKEY_FILE}" ec2-modules/my_key
                                 chmod 600 ec2-modules/my_key

                                terraform init
                                terraform apply --auto-approve
                                terraform destroy --auto-approve
                            """
                        }
                    }
                }
            }
        }

        stage("Deploy to EC2") {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'EC2_KEY', keyFileVariable: 'SSH_KEY'),
                    string(credentialsId: 'EC2_HOST', variable: 'EC2_HOST'),
                    usernamePassword(credentialsId: 'docker_creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')
                ]) {

                    sh '''
                        chmod 600 $SSH_KEY

                        ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@$EC2_HOST << EOF
                            echo "Connected to EC2"
                            export DOCKER_USERNAME="$DOCKER_USERNAME"
                            export DOCKER_PASSWORD="$DOCKER_PASSWORD"

                            cd /home/ubuntu/React-ToDoList
                            bash /home/ubuntu/React-ToDoList/deploy.sh
                        EOF
                    '''
                }
            }
        }

    }
}
