pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "stsybulniak/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                echo 'Remove node_modules folder'
                sh 'rm -rf node_modules'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("stsybulniak/train-schedule")
                    app.inside {
                        sh 'echo Hello, world!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_username_secret') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToStaging') {
            when {
                branch 'stage'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProduction') {
            // when {
            //     branch 'master'
            // }
            steps {
                // input 'Deploy to Production?'
                // milestone(1)
                withKubeConfig([credentialsId: 'k8s-conf', serverUrl: 'http://10.0.0.10']) {
                    sh 'curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.26.2/bin/linux/amd64/kubectl"'
                    sh 'chmod u+x ./kubectl'
                    sh 'kubectl get nodes'
                }
                
                // podTemplate(cloud: 'k8s master', yaml: readTrusted('train-schedule-kube.yml')) {
                //     node(POD_LABEL) {
                //         sh 'hostname'
                //     }
                }
                // kubernetesDeploy(
                //     kubeconfigId: 'kubeconfig',
                //     configs: 'train-schedule-kube.yml',
                //     enableConfigSubstitution: true
                // )
            }
        }
    }
}