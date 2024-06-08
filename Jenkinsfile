pipeline {
    agent any
   parameters {
  choice(choices: ['staging', 'prod'], name: 'environment', description: 'Choose the environment')
}
    environment {
        BUILD_NUMBER = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Clone') {
            steps {
                git 'git@github.com:deepak83s143/website-lab.git'
            }
        }
        stage('Build') {
            steps {
            	 script {
                    if (params.environment == 'staging') {
                	def buildNumber = env.BUILD_NUMBER
                	sh "sed 's/BUILD_NUMBER/${buildNumber}/' index1.html > index.html"
                	sh """
                        echo '<h2>This is Staging Environment</h2>' >> index.html
                        """
                	sh 'docker build -t website-image --pull=false .'                    
                	} 
                    if (params.environment == 'prod') {
                	def buildNumber = env.BUILD_NUMBER
                	sh "sed 's/BUILD_NUMBER/${buildNumber}/' index1.html > index.html"
                	sh """
                        echo '<h2>This is Production Environment</h2>' >> index.html
                        """
                	sh 'docker build -t website-image --pull=false .'                
                       }
                }
            }
        }
        stage('Creating Image') {
            steps {
                script {
                    if (params.environment == 'staging') {
                        def buildNumber = env.BUILD_NUMBER
                        def registryServer = "10.201.19.10"
                        def envURL = "staging.mademitech.local"
                        sh "docker tag website-image:latest 10.201.19.10:32000/website-image:ver.${buildNumber}"
                        sh "sed 's/BUILD_NUMBER/${buildNumber}/' website.yaml > website1.yaml"
                        sh "sed 's/REGISTRY_SERVER/${registryServer}/' website1.yaml > website2.yaml"
                        sh "sed 's/env_url/${envURL}/' ingress.yaml > ingress1.yaml"                       
                        sh "echo '${buildNumber}'"
                        sh "scp website2.yaml ingress1.yaml ubuntu@10.201.19.10:~/website"
                        sh "docker push 10.201.19.10:32000/website-image:ver.${buildNumber}"    
                    } 
                    if (params.environment == 'prod') {
                        def buildNumber = env.BUILD_NUMBER
                        def registryServer = "10.201.18.116"
                        def envURL = "prod.mademitech.local"
                        sh "docker tag website-image:latest 10.201.18.116:32000/website-image:ver.${buildNumber}"
                        sh "sed 's/BUILD_NUMBER/${buildNumber}/' website.yaml > website1.yaml"
                        sh "sed 's/REGISTRY_SERVER/${registryServer}/' website1.yaml > website2.yaml"
                        sh "sed 's/env_url/${envURL}/' ingress.yaml > ingress1.yaml"
                        sh "echo '${buildNumber}'"
                        sh "echo '${registryServer}'"
                        sh "scp website2.yaml ingress1.yaml ubuntu@10.201.18.116:~/website"
                        sh "docker push 10.201.18.116:32000/website-image:ver.${buildNumber}"
                    }
                }
            }
        }
        stage('Send artifacts to K8s') {
            steps {
            script {
                    if (params.environment == 'staging') {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'MK8S_19.10', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /home/ubuntu/website
                microk8s kubectl  delete -f website2.yaml
                microk8s kubectl delete -f ingress1.yaml
                sleep 5
                microk8s kubectl apply  -f website2.yaml
                microk8s kubectl apply -f ingress1.yaml''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/ubuntu/website', remoteDirectorySDF: false, removePrefix: '/home/ubuntu/website', sourceFiles: '/home/ubuntu/website/website2.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                    } 
                    if (params.environment == 'prod') {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'MK8S_18.116', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /home/ubuntu/website
                microk8s kubectl  delete -f website2.yaml
                microk8s kubectl delete -f ingress1.yaml
                sleep 5
                microk8s kubectl apply  -f website2.yaml
                microk8s kubectl apply -f ingress1.yaml''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '/home/ubuntu/website', remoteDirectorySDF: false, removePrefix: '/home/ubuntu/website', sourceFiles: '/home/ubuntu/website/website2.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
                    }
                }
            }
        }
        stage('Final Stage') {
            steps {
                echo "Selected environment: ${params.environment}"
            }
        }
    }
    post {
    always {
        script {
            def jobUrl = env.JOB_URL ?: "N/A"
            def branchName = env.BRANCH_NAME ?: "N/A"
            def buildNumber = env.BUILD_NUMBER ?: "N/A"
            def jobName = env.JOB_NAME ?: "N/A"

            emailext (
                bcc: '',
                body: """
                    Job Name: ${jobName}
                    Build Number: #${buildNumber}
                    Job URL: ${jobUrl}
                    Environment: ${params.environment}
                """,
                from: 'cloudoperations@netwebindia.com',
                replyTo: '',
                subject: '[${BUILD_STATUS}]${JOB_NAME} Build #${BUILD_NUMBER}',
                to: 'deepak.s@netwebindia.com'
                )
            }
        }
    }
}

