pipeline {
    agent any
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }
    parameters {
      string(name: 'sonar_IP', defaultValue: '54.221.90.229', description: 'IP of sonarqube')
      string(name: 'nexus_IP', defaultValue: '18.234.209.237', description: 'IP of nexus')
      string(name: 'deploy_IP', defaultValue:'50.17.8.182', description: 'IP of Deploy Server')  
    }
    environment {
      SONARQUBE_URL="http://${params.sonar_IP}:9000"
      SONARQUBE_TOKEN=credentials('Sonar-token')
      NEXUS_URL="http://${params.nexus_IP}:8081"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                url: 'https://github.com/kavyakr9599-dotcom/webapp.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('webapp') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=MavenProject \
                    -Dsonar.host.url=$SONARQUBE_URL \
                    -Dsonar.login=$SONARQUBE_TOKEN
                    """
                }
            }
        }
        stage('Build') {
            steps {
                dir('webapp') {
                sh 'mvn clean package -DskipTests'
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                dir('webapp') {
                    sh '''
                    mvn clean deploy -DskipTests \
                    -DaltDeploymentRepository=nexus-snapshots::$NEXUS_URL/nexus/content/repositories/maven-snapshots/
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(credentials:['ec2-ssh']) {
                    sh '''
                    scp -o StrictHostKeyChecking=no \
                        webapp/target/webapp.war \
                        ubuntu@${deploy_IP}:/tmp/webapp.war

                    ssh -o StrictHostKeyChecking=no ubuntu@${deploy_IP} "
                        sudo cp /tmp/webapp.war /home/ubuntu/tomcat/webapps/webapp.war &&
                        /home/ubuntu/tomcat/bin/shutdown.sh
                        /home/ubuntu/tomcat/bin/startup.sh
                    "
                    '''
                }
            }
        }
    }
}
