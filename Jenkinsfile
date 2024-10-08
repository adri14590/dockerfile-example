pipeline {
    agent any
    environment {
        registry = "adri14590/jenkins-example"
        registryCredentials = "dockerhub"
        project = "jenkins-example"
        repository = "https://github.com/adri14590/dockerfile-example.git"
        repositoryCredentials = "github"
        FEATURE_REGEX = /^feature\/.+$/
        RELEASE_REGEX = /^release\/\d+\.\d+\.\d+$/
        TAG_REGEX = /^\d+\.\d+\.\d+$/ // Expresión regular para el formato SemVer
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    // Lógica para determinar el tag de la imagen
                    def dockerTag = ""
                    if (env.BRANCH_NAME == "develop" || env.BRANCH_NAME ==~ env.FEATURE_REGEX) {
                        // Si estamos en develop o feature/*, usa el SHA1 del commit como tag
                        dockerTag = "${env.GIT_COMMIT}"
                    } else if (env.BRANCH_NAME ==~ env.RELEASE_REGEX) {
                        // Si estamos en release/x.y.z, usa pre-x.y.z como tag
                        def releaseVersion = env.BRANCH_NAME.replace("release/", "")
                        dockerTag = "pre-${releaseVersion}"
                    } else if (env.BRANCH_NAME == "master" || env.BRANCH_NAME == "main") {
                        // Si estamos en master o main, usa latest como tag
                        dockerTag = "latest"  
                    } else if (env.BRANCH_NAME ==~ env.TAG_REGEX) {
                        // Si estamos en un tag semver, usa x.y.z como tag
                        dockerTag = env.BRANCH_NAME
                    } else {
                        error("No se pudo determinar el tag de la imagen Docker")
                    }
                    // Set the dockerTag in the pipeline environment for use in other stages
                    env.DOCKER_TAG = dockerTag
                }
            }
        }

        // stage('Clean Workspace') {
        //     steps {
        //         cleanWs()
        //     }
        // }

        // stage('Checkout Code') {
        //     steps {
        //         script {
        //              git branch: env.BRANCH_NAME,
        //                 credentialsId: repositoryCredentials,
        //                 url: repository
        //         }
        //     }
        // }

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

        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    def containerName = "${env.project}-${env.BRANCH_NAME}"
                    try {
                        echo "Creando el contenedor con el nombre: ${containerName}"
                        // Usa la interpolación correcta para pasar la variable a la shell
                        sh "docker run --name ${containerName} -e 'LENGTH=20' ${registry}"
                    } finally {
                        // Asegúrate de eliminar el contenedor con el nombre correcto
                        sh "docker rm ${containerName}"
                    }
                }
            }
        }

        stage('Delivery') {
            steps {
                script {            
                    echo "Subiendo la imagen con el tag: ${env.DOCKER_TAG}"

                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push(env.DOCKER_TAG)
                    }
                }
            }
        }

        stage('Cleaning Up') {
            steps {
                script {
                    // Use the dockerTag from the environment variable
                    def dockerTag = env.DOCKER_TAG
                    
                    if (dockerTag) {
                        sh "docker rmi ${registry}:${dockerTag}"
                    } else {
                        echo "No se pudo eliminar la imagen ya que dockerTag no está definido."
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Registrar Build'
        }
    }
}
