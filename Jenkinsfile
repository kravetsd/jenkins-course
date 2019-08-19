node { 
    stage('Prepare') {
    def workspace = pwd()
    echo "\u2600 workspace=${workspace}"
    git (url:"https://github.com/kravetsd/docker-demo", branch: "master")
    def nodejs = docker.image('node:latest')
    nodejs.pull() // make sure we have the latest available from Docker Hub
    nodejs.inside { 
        sh 'npm test'
        sh 'npm install'
    // …as above
    }
            // def newParamsList = [] 
        
    }
    stage('Test') {
    println("Hello stage2")
    def mydockerrepo = withDockerRegistry(url: "https://hub.docker.com/r/kdykrg/docker-nodejs-demo", credentialsId: "docker-hub") {   
    println("I am in the method body!")
    def img = docker.build 'kdykrg/docker-nodejs-demo:latest'
    img.push()
    
     }

    println(mydockerrepo)
        // 
    }
    stage('Deploy') { println("Hello stage3")
        // 
    }
}