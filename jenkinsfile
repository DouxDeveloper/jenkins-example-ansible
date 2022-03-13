
pipeline {
    agent any
    
    environment {
        // Branche actuelle
        String CURRENT_BRANCH = "develop" // "${GIT_BRANCH.split("/")[1]}"
        dockerImage = ""
        // admin_dockerhub = ID du credentials dans le Jenkins
        DOCKERHUB_CREDENTIALS="dockerhub"
        //DOCKERHUB_CREDENTIALS=credentials('dockerhub')
        SUDO_ANSIBLE_CREDENTIALS=credentials('sudo_ansible')
        IMAGE_NAME="${CURRENT_BRANCH}"
        IMAGE_TAG="1.0.5"
        REGISTRY = "doudouxthegeek/frontend"
        
        boolean doitPreparerLivraison = "${env.PREPARER_LIVRAISON}";
    }
    stages {
        
        stage("SCM") {
            steps {
                git branch: 'develop', 
                changelog: false, 
                credentialsId: 'github', 
                poll: false, 
                url: 'https://github.com/DouxDeveloper/frontend.git'
            }
        }
        
        stage("Build image") {
                steps {
                    echo "🐳 Construction Image Docker"
                    echo "Image tag : ${IMAGE_TAG}"
                    script {
                        // bat "docker build -t ${REGISTRY}:${IMAGE_TAG} --build-arg ENV=${CURRENT_BRANCH} ."
                        dockerImage = docker.build("${REGISTRY}:${IMAGE_TAG}", "--build-arg ENV=${CURRENT_BRANCH} -f Dockerfile .")
                    }
                }
        }
        
         stage("Push to Dockerhub") {
            steps {
                script {
                    if(doitPreparerLivraison == "true") {
                        echo "Publishing image to Dockerhub ..."
                        docker.withRegistry( '', DOCKERHUB_CREDENTIALS ) {
                            dockerImage.push()
                        }
                    }

                }
            }
        }
        
        stage("Deploy image") {
           agent {
                label 'ansible'
           }
           steps {
               script {
                   echo "${doitPreparerLivraison}"
                        if(doitPreparerLivraison == "true") {
                            ansiblePlaybook credentialsId: 'ansible', 
                            extras: '-e ansible_sudo_pass=ansible -e REGISTRY=${REGISTRY} -e IMAGE_TAG=${IMAGE_TAG}',
                            installation: 'ansible', 
                            disableHostKeyChecking: true,
                            inventory: '/home/ansible/project/inventory.ini', 
                            playbook: '/home/ansible/project/deploy_docker.yml'
                    }
               }
            }
       }
    }
    
    //post {
    //    always {
    //        bat 'docker logout'
    //    }
    //}

}
