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
    docker.withRegistry(url: "https://docker.io/kdykrg/docker-nodejs-demo", credentialsId: "docker-hub") {   
    println("I am in the method body!")
    docker.build("kdykrg/docker-nodejs-demo").push('latest')
     }

    println(mydockerrepo)
        // 
    }
    stage('Deploy') { println("Hello stage3")
        // 
    }
}