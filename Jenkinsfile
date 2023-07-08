pipeline{
    agent{
        label "jenkins-slave-1"
    }
    stages{
        stage("Installing puppet-agent"){
            steps{
                sh "sudo apt-get install puppet-agent -y"
            }
        }
        stage("Docker installation on slave node"){
            agent{
                label 'master'
            }
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'jenkins-slave-ssh-creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh """
                            ansible kslave1 -m apt -a "name=docker state=present" --extra-vars "ansible_user=${USERNAME} ansible_password=${PASSWORD}"
                        """
                    }
                }
            }
        }
        stage("Docker build and deploy"){
            steps{
                script{
                    sh """
                        docker build . -t website:php-app
                        [ "`docker ps -a | grep php_app`" ] && docker rm -f php_app || echo "No existing container found"
                        docker run -d --name php_app website:php-app
                    """
                }
            }
            post{
                failure{
                    sh """
                        docker rm php_app -f
                    """
                }
            }
        }
    }
}