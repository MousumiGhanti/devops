try{
    node{
        
        def mavenHome
        def mavenCMD
        def docker
        def dockerCMD
        def tagName = "1.0"
        
        stage('Environment Preparation'){
            echo "Preparing the Jenkins environment with required tools..."
            mavenHome = tool name: 'maven-3', type: 'maven'
            mavenCMD = "${mavenHome}/bin/mvn"
            docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
            dockerCMD = "$docker/bin/docker"
        }
        
        stage('git checkout'){
            echo "Checking out the code from git repository..."
            git 'https://github.com/MousumiGhanti/devops.git'
        }
        stage('Build and Package'){
            echo "Building the application..."
            sh "${mavenCMD} clean package"
        }
        
        stage('Sonar Scan'){
            echo "Scanning application for vulnerabilities..."
            sh "${mavenCMD} sonar:sonar -Dsonar.host.url=http://34.134.175.243:9000/"
        }
        
        stage('Publish Test Report'){
            echo " Publishing HTML report.."
            sh "${mavenCMD} surefire-report:report"
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target/site', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: ''])
            
        }
        stage('Build Docker Image'){
            echo "Building docker image for my application ..."
            sh "${dockerCMD} build -t mousumighanti/mouapp:${tagName} ."
        }
        stage("Push Docker Image to Docker Registry"){
            echo "Pushing image to docker hub"
            withCredentials([string(credentialsId: 'dockerpassword', variable: 'dockerpwd')]) {
            sh "${dockerCMD} login -u mousumighanti -p ${dockerpwd}"
            sh "${dockerCMD} push mousumighanti/mouapp:${tagName}"
            }
        }
        
        stage('Deploy Application'){
            echo "Deploying  application"
            ansiblePlaybook credentialsId: 'ubuntu-ssh-connection', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'deploy.yml'
        }
    }
}
catch(Exception err){
    echo "Exception occured..."
    currentBuild.result="FAILURE"
   
     emailext attachLog: true, body: '', subject: 'Build Failure Result', to: 'ghantimousumi2607@gmail.com'
}
finally {
    (currentBuild.result!= "ABORTED") && node("master") {
        echo "finally gets executed and end an email notification for every build"
        emailext attachLog: true, body: 'Build completed', subject: 'Build Result', to: 'ghantimousumi2607@gmail.com'
    }
    
}
