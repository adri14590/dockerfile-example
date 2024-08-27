pipeline{
    agent any
    environment{
        registry= "adri14590/jenkins-example"
        registryCredentials="dockerhub"
        project="jenkins-example"
        projectVersion="1.0"
        repository="https://github.com/adri14590/dockerfile-example.git"
        repositoryCredentials="github"
    }
    stages{

        stage('Clean Workspace'){
            steps{
                cleanWs()
            }
        }
        
        stage('Checkout code'){
            steps{
                script{
                    git branch: 'main',
                        credentialsId: repositoryCredentials,
                        url: repository
                }
            }
        }

        // stage('Code Analysis'){
        //     environment{
        //         scannerHome= tool 'Sonar'
        //     }
        //     steps{
        //         script{
        //             withSonarQubeEnv('Sonar'){
        //                 sh "${scannerHome}/bin/sonar-scanner \
        //                 -Dsonar.projectKey=$project \
        //                 -Dsonar.projectName=$project \
        //                 -Dsonar.projectVersion=$projectVersion \
        //                 -Dsonar.sources=./"
        //             }
        //         }
        //     }
        // }

        stage('Build'){
            steps{
                script{
                    dockerImage= docker.build registry
                }
            }
        }

        stage('Test'){
            steps{
                script{
                    try{
                        sh 'docker run --name $project -e "LENGTH=20" $registry'
                    }finally{
                        sh 'docker rm $project'
                    }

                }
            }
        }

        stage('Delivery'){
            steps{
                script{
                    docker.withRegistry('',registryCredentials ){
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Cleaning Up'){
            steps{
                script{
                    sh 'docker rmi $registry'
                }
            }
        }

    }
    post{
        always{
            echo 'Registrar Build'
        }
    }
}