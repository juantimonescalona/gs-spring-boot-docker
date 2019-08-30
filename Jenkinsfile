node {
   def mvnHome
   def docker
   def buildnumber = "${BUILD_NUMBER}"
   def remote = [:]
       remote.name = 'local'
       remote.host = '192.168.1.53'
       remote.user = 'jtimon'
       remote.password = 'juan2019'
       remote.allowAnyHosts = true
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/juantimonescalona/gs-spring-boot-docker'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
      mvnHome = tool 'M3'
      docker = tool 'Docker'
   }
   stage('Build Java') {
      withEnv(["MVN_HOME=$mvnHome"]) {
         if (isUnix()) {
            sh 'cd complete && "$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
            /* && java -jar target/gs-spring-boot-docker-0.1.0.jar --server.port=8083 */
         } else {
            bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
         }
      }
   }
/* REMOTE BUILD
    stage('Remote SSH Test') {
      sshCommand remote: remote, command: "ls -lrt"
    }
   
    stage('Remote Copy Files') {
        sshPut remote: remote, from: 'complete/Dockerfile', into: '/home/jtimon/temp_jenkins'
        sshPut remote: remote, from: 'complete/deployment-k8s.yaml', into: '/home/jtimon/temp_jenkins'
        sshPut remote: remote, from: 'complete/target', into: '/home/jtimon/temp_jenkins'
    }
   
    stage('Remote Build docker') {
        sshCommand remote: remote, command: "cd /home/jtimon/temp_jenkins && docker build -t jtimon/prueba:" + buildnumber +" ."
        sshCommand remote: remote, command: "cd /home/jtimon/temp_jenkins && rm -r target"
    }
*/   

/* BUILD IN JENKINS */
    //stage('Build docker with shell') {
    //    sh 'cd complete && build -t jtimon/prueba:' + buildnumber +' .'
    //}
    
    stage('Build docker with pipeline') {
        sh 'cd complete && cp Dockerfile target'
        dir('complete/target') {
            sh 'docker build -t jtimon/prueba:'+ buildnumber + ' .'
        }
    }
    
    stage('Check Api K8S') {
        def url_k8s = "https://192.168.77.10:6443"
        def secret_k8s = "default-token-kchrk"
        def token_k8s = "eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4ta2NocmsiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImUzZmJhMWRmLWMwZTktNDgxNS1iNjExLTcyYThjZjIwOWRmYSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.LsU4U5QdWOg36TWgDsF8S770BumDTwSV5jO1So4LkRD2JjnrAUBsP99xGJFwYqerdDFm2YYFQpJTDCojO3uiA--bMslD8bVnrtxMsbV_Gv06AqshdfOulBzNHndxZrSu_W8pAqN_Vs0L89DYdDIRpTojkjKgNjACXDWkl7-xXiCnkLDWG6b4ncJfAPipBkX6RYidLk9wPAwzHYwkMhtm0tOp-9YZwOVQgBDH5skAY5ypRwQTALZAaBAb-9wlHtRRM4Fh0jyhfm_VCoOFoINJsG3EEsy_5Pi_Bqd8Klz0NCP5S8k7nzNO1Rsdb-kp4b2JqdJSQOO0korPCMbadEs-vw"
        sshCommand remote: remote, command: 'curl '+ url_k8s + '/api' + ' --header "Authorization: Bearer ' + token_k8s +'" --insecure'
    }
/*    
    stage('Remote Copy image and deployment yaml to K8s master node') {
        sshCommand remote: remote, command: "cd /home/jtimon/temp_jenkins && docker save prueba:" + buildnumber + " | gzip > myimage_latest.tar.gz"
        //from: '/home/jtimon/temp_jenkins/myimage_latest.tar.gz', into: '/home/vagrant'
        // Connect ssh to mater node k8s --> sshCommand remote: remote, command: "ssh -p 2222 vagrant:vagrant@127.0.0.1 -i /home/jtimon/.vagrant.d/insecure_private_key"
        sshCommand remote: remote, command: "cd /home/jtimon/temp_jenkins && scp -P 2222 -i /home/jtimon/.vagrant.d/insecure_private_key  /home/jtimon/temp_jenkins/myimage_latest.tar.gz  vagrant@127.0.0.1:/home/vagrant"
        sshCommand remote: remote, command: "cd /home/jtimon/temp_jenkins && scp -P 2222 -i /home/jtimon/.vagrant.d/insecure_private_key  /home/jtimon/temp_jenkins/deployment-k8s.yaml  vagrant@127.0.0.1:/home/vagrant"
        sshCommand remote: remote, command: "ssh -p 2222 vagrant:vagrant@127.0.0.1 -i /home/jtimon/.vagrant.d/insecure_private_key 'docker load --input myimage_latest.tar.gz && rm myimage_latest.tar.gz && sed -i 's/#buildnumber/" + buildnumber + "/g' deployment-k8s.yaml'"
    }

    stage('Push image') {
        sshCommand remote: remote, command: "cat my_password.txt | docker login --username jtimon --password-stdin"
        sshCommand remote: remote, command: "docker push jtimon/prueba:" + buildnumber
    }
    */
    
    
    /*
    stage('Remote deploy to K8s master node') {
        sshCommand remote: remote, command: "ssh -p 2222 vagrant:vagrant@127.0.0.1 -i /home/jtimon/.vagrant.d/insecure_private_key  'kubectl apply -f deployment-k8s.yaml'"
    }
    
    stage('Remote get pods') {
        sshCommand remote: remote, command: "ssh -p 2222 vagrant:vagrant@127.0.0.1 -i /home/jtimon/.vagrant.d/insecure_private_key  'kubectl describe deployments jtimon-deployment'"
    }
    */ 
}
