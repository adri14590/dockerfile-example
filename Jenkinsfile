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
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    git branch: env.BRANCH_NAME,
                        credentialsId: repositoryCredentials,
                        url: repository
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    try {
                        def gitRef = env.BRANCH_NAME || env.GIT_TAG_NAME
                        def containerName = env.project + "-" + gitRef
                        echo "Creando el contenedor con el nombre: ${containerName}"
                        sh 'docker run --name ${containerName} -e "LENGTH=20" $registry'
                    } finally {
                        sh 'docker rm $project'
                    }
                }
            }
        }

        stage('Delivery') {
            steps {
                script {
                    def dockerTag = ""
                    
                    if (env.BRANCH_NAME ==~ /^develop$/ || env.BRANCH_NAME ==~ env.FEATURE_REGEX) {
                        // Si estamos en develop o feature/*, usa el SHA1 del commit como tag
                        dockerTag = "${env.GIT_COMMIT}"
                    } else if (env.BRANCH_NAME ==~ env.RELEASE_REGEX) {
                        // Si estamos en release/x.y.z, usa pre-x.y.z como tag
                        def releaseVersion = env.BRANCH_NAME.replace("release/", "")
                        dockerTag = "pre-${releaseVersion}"
                    } else if (env.BRANCH_NAME == "master" || env.BRANCH_NAME == "main") {
                        // Si estamos en master o main, usa latest como tag
                        dockerTag = "latest"  
                    } else if (env.GIT_TAG_NAME ==~ env.TAG_REGEX) {
                        // Si estamos en un tag semver, usa x.y.z como tag
                        dockerTag = env.GIT_TAG_NAME
                    } else {
                        error("No se pudo determinar el tag de la imagen Docker")
                    }

                    echo "Subiendo la imagen con el tag: ${dockerTag}"

                    docker.withRegistry('', registryCredentials) {
                        dockerImage.push(dockerTag)
                    }
                }
            }
        }

        stage('Cleaning Up') {
            steps {
                script {
                    sh 'docker rmi $registry'
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
