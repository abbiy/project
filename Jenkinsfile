//pipeline code
pipeline {
    agent any
    stages {
        stage('Networking Configuration') {
            steps {
                sh 'docker network ls'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'cd $WORKSPACE'
                sh 'rm -rf project'
                git branch: "master",
                    url: "https://github.com/abbiy/project"
                sh 'ls'
            }
        }
        stage('Pre-Build Tests') {
            parallel {
                stage('Git Repository Scanner') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'git clone https://github.com/abbiy/project && cd project'
                        sh 'docker run --rm -v "$(pwd):/proj" dxa4481/trufflehog file:///proj | tee truffleout-test'
                        //sh 'trufflehog https://github.com/abbiy/project --json | jq "{branch:.branch, commitHash:.commitHash, path:.path, stringsFound:.stringsFound}" > trufflehog_report.json || true'
                        sh 'cat truffleout-test'
                        sh 'echo "Scanning Repositories.....done"'
                       // telegrambot attachmentspattern: 'trufflehogtest',
                        archiveArtifacts artifacts: 'truffleout-test', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: 'truffleout-test', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Thankyou,\n CDAC-Project Group-7", 
                        subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME} - success", mimeType: 'text/html', to: "abbyvishnoi@gmail.com"
                    }
                }
                stage('Image Security') {
                    steps {
                        sh 'cd $WORKSPACE'
                        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock  goodwithtech/dockle:v0.4.5 login-image | tee login-report.json'
                        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock  goodwithtech/dockle:v0.4.5 postgres | tee postgres-report.json'
                        sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock  goodwithtech/dockle:v0.4.5 pgadmin4 | tee pgadmin4.json'
                        archiveArtifacts artifacts: '*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.json', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "abbyvishnoi@gmail.com"
                    }
                }
            }
        }
        stage('Build Stage') {
            steps {
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn package'
            }
        }
        stage('Initializing Docker') {
            steps {
                sh 'docker stop pgadmin_container || true'
                sh 'docker stop postgres_container || true'
                sh 'docker stop login || true'
                sh 'docker start pgadmin_container || true'
                sh 'docker start postgres_container || true'
                sh 'docker start login || true'
            }
        }
        stage('SCA') {
            parallel {
                stage('Dependency Check') {
                    steps {
                        sh 'wget https://github.com/abbiy/project/blob/master/dc.sh'
                        sh 'chmod +x dc.sh'
                        sh './dc.sh'
                        archiveArtifacts artifacts: 'odc-reports/*.html', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.csv', onlyIfSuccessful: true
                        archiveArtifacts artifacts: 'odc-reports/*.json', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*.html', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "abbyvishnoi@gmail.com"
                    }
                }
                stage('Junit Testing') {
                    steps {
                        sh 'echo "Junit Reports are created using archiveArtifacts"'
                        archiveArtifacts artifacts: '*junit.xml', onlyIfSuccessful: true
                        emailext attachLog: true, attachmentsPattern: '*junit.xml', 
                        body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}\n Please Find Attachments for the following:\n Thankyou\n CDAC-Project Group-7",
                        subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - success", mimeType: 'text/html', to: "abbyvishnoi@gmail.com"
                    }
                }
            }
        }
        stage('DAST') {
            steps {
                sh 'docker run -u root --name zap1 --rm -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable:latest zap-baseline.py -t http://192.168.96.147:8080/LoginWebApp -r report_html || true' 
                //sh 'docker run --name dast_full --network project_project -t owasp/zap2docker-stable zap-full-scan.py -t http:192.168.96.135/LoginWebApp/ || true'
                //sh 'docker run --name dast_baseline --network project_project -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.96.135/LoginWebApp/ --autooff || true'
            }
        }
    }
}
        //stage('telegram') {
          //  steps {
           //     script{
             //        withCredentials([string(credentialsId: ‘telegramToken’, variable: 'TOKEN'),
               //      string(credentialsId: ‘telegramChatId’, variable: ‘CHAT_ID’)]) {
                 //    telegramSend(messsage:"build confirmation",chatId:${895874551})
           //  }
        //}
   // }
//}
