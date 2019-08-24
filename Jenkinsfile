node {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'gnome-terminal']) {
        def String hostIp
        def String masterIp
        def String registryUrl='https://registry.hub.docker.com'
        def String registryCredentialsId = 'docker-hub'
        def String cfStackName

        cleanWs() 
        stage('Prepareation') {
        def wspace = pwd()
        masterIp = sh(returnStdout: true, script: "curl 169.254.169.254/latest/meta-data/public-ipv4").trim()
        git (url:"https://github.com/kravetsd/docker-demo", branch: "master")
        }
        stage('Unit tests') {
        def nodejs = docker.image('node:latest')
        nodejs.pull()
        nodejs.inside { 
            sh script: 'npm test', returnStdout: true
            sh script:'npm install', returnStdout: true
        }
        }
        stage('Build') {
        docker.withRegistry(registryUrl,registryCredentialsId ) {
            docker.build("kdykrg/docker-nodejs-demo").push('latest')
         }
        }
        stage('Build infratsructure') {
        withAWS(credentials:'awscredentials') {
            def outputs = cfnUpdate(stack: ${stackName}, file:'jenkinsmudule.yml',params:["JenkinsMasterIp=${masterIp}"],  timeoutInMinutes:10, tags:['Builder=Jenkins'], pollInterval:1000)
            hostIp = outputs.Ec2Ip
        }
        }
        stage('Delivery'){
            ansiblePlaybook( 
            playbook: 'playbook.yaml',
            installation: 'ansible',
            lock_timeout: 30,
            credentialsId : 'ansible-key',
            disableHostKeyChecking: true,
            inventoryContent: "${hostIp}"
            )

        }
        stage('e2e test'){
            printl("checking if your application is available with link: http://'${hostIp}'")
            sh(returnStdout: true, script: "curl '${hostIp}'" )
        }
    }
}