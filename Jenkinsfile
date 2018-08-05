pipeline {
    agent none

    stages {
        def getJenkinsMaster() {
            return env.BUILD_URL.split('/')[2].split(':')[0]
        }


        stage('Unit Tests') {
            agent { 
                label 'apache'
            }

            steps {
                sh 'ant -f test.xml -v'
                junit 'reports/result.xml'
            }
        }

        stage('build') {
            agent { 
                label 'apache'
            }

            steps {
                sh 'ant -f build.xml -v'
            }

            post {
                success {
                    archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
                }
        
            }
        }

        stage('deploy') {
            agent { 
                label 'apache'
            }

            steps {
                sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/"
            }
        }

        stage('Running on CentOS') {
            agent { 
                label 'CentOS'
            }

            steps {
                echo getJenkinsMaster()
                //sh "wget $JENKINS_IP/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
                //sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 5 6"
            }
        }

        stage('Test on Debian docker') {
            agent {
                docker 'openjdk:8u171-jre'    
            }
        
            steps {
                //sh "wget $JENKINS_IP/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
                //sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 5 6"
            }
        }
     
        stage('Promote to Green') {
            agent {
                label 'apache'
            }
            steps {
                //sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.BUILD_NUMBER}.jar"
            }
        }
    }


}
