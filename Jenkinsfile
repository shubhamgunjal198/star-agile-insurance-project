node {

    // def mavenHome
    // def mavenCMD
    // def dockerCMD
    // def tagName

    // stage('Prepare Environment') {
    //     echo 'Initializing environment variables...'
    //     mavenHome = tool name: 'maven', type: 'maven'
    //     mavenCMD = "${mavenHome}/bin/mvn"
    //     dockerCMD = '/usr/bin/docker'  // System Docker path
    //     tagName = "3.0"
    // }
    def mavenCMD = '/usr/bin/mvn'          // Use system Maven
    def dockerCMD = '/usr/bin/docker'      // Use system Docker
    def tagName = "3.0"

    stage('Git Code Checkout') {
        try {
            echo 'Checking out the code from Git repository...'
            git 'https://github.com/shubhamgunjal198/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext(
                body: """Dear All,<br>
                The Jenkins job <b>${JOB_NAME}</b> has failed.<br>
                Please check it here: <a href='${BUILD_URL}'>${BUILD_URL}</a>""",
                subject: "Job ${JOB_NAME} #${BUILD_NUMBER} Failed",
                to: 'shubhamgunjal198@gmail.com'
            )
            error("Stopping build due to Git checkout failure.")
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        echo 'Publishing test reports...'
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: "${WORKSPACE}/target/surefire-reports",
            reportFiles: 'index.html',
            reportName: 'HTML Report',
            reportTitles: ''
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image...'
        sh "${dockerCMD} build -t shubhamgunjal1619/insure-me:${tagName} ."
    }

    stage('Pushing to DockerHub') {
        echo 'Pushing the Docker image to DockerHub...'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh """
                echo $dockerHubPassword | ${dockerCMD} login -u shubhamgunjal1619 --password-stdin
                ${dockerCMD} push shubhamgunjal1619/insure-me:${tagName}
            """
        }
    }

    stage('Configure and Deploy to the Test Server') {
        echo 'Deploying the application using Ansible...'
        ansiblePlaybook(
            become: true,
            credentialsId: 'ansible-key',
            disableHostKeyChecking: true,
            installation: 'ansible',
            inventory: '/etc/ansible/hosts',
            playbook: 'ansible-playbook1.yml'
        )
    }
}
