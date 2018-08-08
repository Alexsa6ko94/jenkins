pipeline {
    agent none

    environment {
        MAJOR_VERSION = 1
    }

    stages {

        stage('Say Hello') {
            agent any

            steps {
                groovyHello 'Groovy Master!'
            }
        }

        stage('Git info:') {
            agent any
            
            steps {
        
                echo "My Branch Name: ${env.BRANCH_NAME}"

                script {
                    def myLib = new aboyanov.git.gitStuffs();

                    echo "My Commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
                }
            }
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
                sh "if ![ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
                sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
            }
        }

        stage('Running on CentOS') {
            agent { 
                label 'CentOS'
            }

            steps {
                sh "wget iliya-belichev-pontica-d0fa788d1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 5 6"
            }
        }

        stage('Test on Debian docker') {
            agent {
                docker 'openjdk:8u171-jre'    
            }
        
            steps {
                sh "wget iliya-belichev-pontica-d0fa788d1.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
                sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 5 6"
            }
        }
     
        stage('Promote to Green') {
            agent {
                label 'apache'
            }

            when {
                branch 'master'
            }

            steps {
                sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
            }
        }

        stage('Promote Dev to Master Banch') {
            agent {
                label 'apache'
            }

            when {
                branch 'dev'
            }

            steps {
                echo "Stashing Any Local Changes"
                sh 'git stash'
                echo "Checking Out Dev Branch"
                sh 'git checkout dev'
                echo "Git pull"
                sh 'git pull origin'
                echo "Checking Out Master Branch"
                sh 'git checkout master'
                echo "Git merge master"
                sh 'git pull origin'
                echo "Merging Dev into Master Branch"
                sh 'git merge dev'
                echo "Pushing to Origin Master"
                sh 'git push origin master'
                echo "Tagging the Release"
                sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
                sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
            }

            post {
                success {
                    emailext(
                        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Dev Promoted to Master",
                        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]'  Dev Promoted to Master":</p>
                        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                        to: "aleksandar.boyanov.pontica@gmail.com"
                    )
                }
            }
        }
    }

    post {
        failure {
            emailext(
                subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
                body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
                <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                to: "aleksandar.boyanov.pontica@gmail.com"
            )
        }
    }
}
