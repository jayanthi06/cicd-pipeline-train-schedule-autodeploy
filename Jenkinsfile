pipeline {  
    agent any 
        environment {
            DockerImageName = "stark985/practicerepo"
            DockerCredentials = credentials('DockerLoginToken')
        }
        stages {
            stage('Build') {
                steps {
                    echo 'Running build automation'
                    sh './gradlew build --no-daemon'
                    archiveArtifacts artifacts: 'dist/trainSchedule.zip'
                }
            }
            stage('Docker Build') {
                steps {
                    script {
                        
                        dir("$env.workspace/cicd-pipeline-train-schedule-autodeploy") {
                            sh 'docker build -t $DockerImageName:$BUILD_NUMBER .'
                        }
                    }    
                }
            }
            stage('Docker Push') {
                steps {
                    script {
                        sh 'echo $DockerCredentials_PSW | docker login -u $DockerCredentials_USR --password-stdin'
                        sh 'docker push $DockerImageName:$BUILD_NUMBER'
                        sh 'docker logout'
                    }    
                }
            }
            stage('CanaryDeploy') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }}
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                script {
                    kubernetesDeploy(
                        
                        kubeconfigId: 'kubeconfig',
                        configs: 'train-schedule-kube-canary.yml',
                        enableConfigSubstitution: true
                    
                    )
                }
            }
        }
        stage('DeployToProduction') {
            when {
                expression {
                    return env.GIT_BRANCH == "origin/master"
                }}
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                milestone(1)
                kubernetesDeploy(
                   
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                
                )
                kubernetesDeploy(
                    
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                    
                )
            }
        }
    }

}
