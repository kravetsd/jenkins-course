node { 
    stage('Prepare') {
    def workspace = pwd()
    echo "\u2600 workspace=${workspace}"
    git (url:"https://github.com/kravetsd/docker-demo", branch: "master")
    }
    stage('Unit tests') {
    def nodejs = docker.image('node:latest')
    nodejs.pull() // make sure we have the latest available from Docker Hub
    nodejs.inside { 
        sh 'npm test'
        sh 'npm install'
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
       def outputs = cfnUpdate(stack:'my-db-stack', file:'Task3.yml', paramsFile:'parameters.json', keepParams:['Version'], timeoutInMinutes:10, tags:['Builder=Jenkins'], pollInterval:1000)
    println(outputs)
    // do something
    }
    
        // 
    }
    stage('delivery'){
        ansiblePlaybook( 
        playbook: 'playbook.yaml'
        )
}
    }