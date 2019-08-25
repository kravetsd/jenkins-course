node {
    wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'gnome-terminal']) {
        def String hostIp
        def String gitBranch = branch
        def String masterIp
        def String registryUrl='https://registry.hub.docker.com'
        def String registryCredentialsId = 'docker-hub'
        def String cfStackName = "Jenkins-mp2019"
        def String result
        def String dockerRepo = "kdykrg/docker-nodejs-demo"
        def String codeRepo = "https://github.com/kravetsd/docker-demo"
        def String cdRepo = "https://github.com/kravetsd/jenkins-course"

        cleanWs() 
        stage('Preparation') {
        def wspace = pwd()
        masterIp = sh(returnStdout: true, script: "curl 169.254.169.254/latest/meta-data/public-ipv4").trim()
        git(url:codeRepo, branch: gitBranch)
        }
        stage('Unit tests') {
        def nodejs = docker.image('node:latest')
        nodejs.pull()
        nodejs.inside { 
            sh(script: 'npm test', returnStdout: true).trim()
            sh(script:'npm install', returnStdout: true).trim()
        }
        }
        stage('Build') {
        docker.withRegistry(registryUrl,registryCredentialsId ) {
            docker.build("${dockerRepo}:${BUILD_NUMBER}",'.').push()
         }
        }
        stage('Build infratsructure') {
            cleanWs()
            git(url: cdRepo, branch: "master")
            withAWS(credentials:'awscredentials') {
            def outputs = cfnUpdate(stack: "${cfStackName}", file:'jenkinsmudule.yml',params:["JenkinsMasterIp=${masterIp}"],  timeoutInMinutes:10, tags:['Builder=Jenkins'], pollInterval:1000)
            hostIp = outputs.Ec2Ip
        }
        }
        stage('Delivery'){
            ansiblePlaybook( 
            playbook: 'playbook.yaml',
            installation: 'ansible',
            credentialsId : 'ansible-key',
            disableHostKeyChecking: true,
            extraVars: "tag"=${BUILD_NUMBER},
            inventoryContent: "${hostIp}"
            )

        }
        stage('e2e test'){
            sleep(10)
            sh(script: "curl ${hostIp}" )
            println("Your application is available via link: http://${hostIp}")
        }
    }
}