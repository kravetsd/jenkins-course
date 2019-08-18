node { 
    stage('Test'){
    def workspace = pwd()
    echo "\u2600 workspace=${workspace}"
    git [url:"https://github.com/kravetsd/docker-demo", bransh: "master"]
    def nodejs = docker.image('nodejs:latest')
    nodejs.pull() // make sure we have the latest available from Docker Hub
    nodejs.inside { 
        sh 'npm test'
        sh 'npm install'
    // …as above
    }
            // def newParamsList = [] 
    mydockerrepo = docker.withDockerRegistry(registry[url: "kdykrg/docker-nodejs-demo", credentialsId:"docker-hub"])        
    }
    stage('Test') { println("Hello stage2")
        // 
    }
    stage('Deploy') { println("Hello stage3")
        // 
    }
}