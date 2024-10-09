@Library('Jenkins_shared_library') _
def COLOR_MAP = [
    'FAILURE' : 'danger',
    'SUCCESS' : 'good'
]
pipeline{
    agent any
    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Select create or destroy.')
        string(name: 'DOCKER_HUB_USERNAME', defaultValue: 'mdngph411', description: 'Docker Hub Username')
        string(name: 'IMAGE_NAME', defaultValue: 'diutup', description: 'Docker Image Name')
    }
    tools{
        jdk 'jdk21'
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWorkspace()
            }
        }
        stage('checkout from Git'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/dungphung411/youtube-clone-app.git'
            }
        }
        stage('sonarqube Analysis'){
        when { expression { params.action == 'create'}}
            steps{
                sonarqubeAnalysis()
            }
        }
        stage('sonarqube QualitGate'){
        when { expression { params.action == 'create'}}
            steps{
                script{
                    def credentialsId = 'sonar-token'
                    qualityGate(credentialsId)
                }
            }
        }
        stage('Npm'){
        when { expression { params.action == 'create'}}
            steps{
                npmInstall()
            }
        }
        stage('Trivy file scan'){
        when { expression { params.action == 'create'}}
            steps{
                trivyFs()
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/DP-report.xml'
            }
        }
        stage('Docker Build'){
        when { expression { params.action == 'create'}}
            steps{
                script{
                   def dockerHubUsername = params.DOCKER_HUB_USERNAME
                   def imageName = params.IMAGE_NAME
                   dockerBuild(dockerHubUsername, imageName)
                }
            }
        }
        stage('Trivy iamge'){
        when { expression { params.action == 'create'}}
            steps{
                trivyImage()
            }
        }
        stage('Run container'){
        when { expression { params.action == 'create'}}
            steps{
                runContainer()
            }
        }
        stage('Remove container'){
        when { expression { params.action == 'delete'}}
            steps{
                removeContainer()
            }
        }
        // stage('Kube deploy'){
        // when { expression { params.action == 'create'}}
        //     steps{
        //         kubeDeploy()
        //     }
        // }
        // stage('kube deleter'){
        // when { expression { params.action == 'delete'}}
        //     steps{
        //         kubeDelete()
        //     }
        // }
    }
    post {
    always {
        echo 'Slack Notifications'
        slackSend (
            channel: '#jenkins', 
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        )
    }
}
}