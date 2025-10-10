node {

    def mavenHome
    def mavenCMD
    def dockerCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initializing environment variables...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        dockerCMD = '/usr/bin/docker'
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        git 'https://github.com/shubhamgunjal198/star-agile-insurance-project.git'
    }

    stage('Build the Application') {
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: "${WORKSPACE}/target/surefire-reports",
            reportFiles: 'index.html',
            reportName: 'HTML Report'
        ])
    }

    stage('Containerize the Application') {
        sh "${dockerCMD} build -t shubhamgunjal1619/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh """
                echo $dockerHubPassword | ${dockerCMD} login -u shubhamgunjal1619 --password-stdin
                ${dockerCMD} push shubhamgunjal1619/insure-me:${tagName}
            """
        }
    }

    stage('Configure and Deploy to the Test Server') {
        ansiblePlaybook(
            become: true,
            credentialsId: 'ansible-key',
            disableHostKeyChecking: true,
            installation: 'ansible',
            inventory: '/etc/ansible/hosts',
            playbook: 'ansible-playbook.yml'
        )
    }
}
