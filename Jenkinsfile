node {
    cleanWs() 
    stage('Prepare') {
    def wspace = pwd()
    echo "\u2600 workspace=${wspace}"
    git (url:"https://github.com/kravetsd/docker-demo", branch: "master")
    }
    stage('Unit tests') {
    def nodejs = docker.image('node:latest')
    nodejs.pull() // make sure we have the latest available from Docker Hub
    nodejs.inside { 
        sh script: 'npm test', returnStdout: true
        sh script:'npm install', returnStdout: true
    // …as above
    }
            // def newParamsList = [] 
        
    }
    stage('Build') {
    def registryUrl='https://registry.hub.docker.com'
    def registryCredentialsId = 'docker-hub'
    println("Hello stage2")
    docker.withRegistry(registryUrl,registryCredentialsId ) {   
    println("I am in the method body!")
    docker.build("kdykrg/docker-nodejs-demo").push('latest')
     }

        // 
    }
    stage('Build infratsructure') { println("Hello stage3")
   withAWS(credentials:'awscredentials') {
       def outputs = cfnUpdate(stack:'my-deployment', file:'jenkinsmudule.yml', timeoutInMinutes:10, tags:['Builder=Jenkins'], pollInterval:1000)
    println(outputs)
    sh "echo ${outputs.Ec2Ip} >> host_vars/hosts.ini"
    sh "cat host_vars/hosts.ini"
    // do something
    }
    
        // 
    }
    stage('delivery'){
        sh "ansiblePlaybook "
        playbook: 'playbook.yaml',
        installation: 'ansible'
        inventory: 'host_vars/hosts.ini'
        )
        
    }
}