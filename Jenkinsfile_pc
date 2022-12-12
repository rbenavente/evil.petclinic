node {
environment {
    PRISMA_API_URL='https://api.eu.prismacloud.io'
 }
stage('Clone repository') {
 checkout scm       

 }
stage('Scan SCA from pom file') {
   script { 
       docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''")  {     
           try {
               sh 'checkov -f pom.xml --use-enforcement-rules -o cli -o junitxml --output-file-path console,results.xml --prisma-api-url https://api.eu.prismacloud.io  --bc-api-key ${ACCESS_KEY}::${SECRET_KEY} --repo-id rbenavente/evil.sca --branch main'
                              junit skipPublishingChecks: true, testResults: 'results.xml'
              } catch (err) {
                              junit skipPublishingChecks: true, testResults: 'results.xml'
                              throw err
                          }
   }  
 }  
}
 stage('Build image') {
 /// This step simulates the image build where the source code plus open source packages are used with Dockerfile to create the image
          
          echo 'buidling image '
     
 }
 
 stage('Scan image before Pushing to Registry') {
      try {
	    
	    sh 'docker pull rbenavente/evilpetclinic:latest'  
            withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
           	prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: 'rbenavente/evilpetclinic:latest', key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
		 
            }
	 } finally {
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
       
       }

    }
 stage('Push image to the registry') {
 /// This step simulates the image Push to the registry 
          
          echo 'Pushing image to registry'
     
 }
 /// Optiona stage to check how Dockerfile scanning works
 
 /// stage('Scan Dockerfile') {
 /// script { 
 ///       docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''")  {     
 ///            try {
 ///               sh 'checkov -f Dockerfile --use-enforcement-rules -o cli -o junitxml --output-file-path console,results.xml --prisma-api-url https://api.eu.prismacloud.io  --bc-api-key ${ACCESS_KEY}::${SECRET_KEY} --repo-id rbenavente/evil.Dockerfile --branch main'
 ///                              junit skipPublishingChecks: true, testResults: 'results.xml'
 ///              } catch (err) {
 ///                              junit skipPublishingChecks: true, testResults: 'results.xml'
 ///                               throw err
 ///                           }
 ///   }  
 /// }  
 ///   }
 
stage('Scan k8s manifest') {
script { 
       docker.image('bridgecrew/checkov:latest').inside("--entrypoint=''")  {     
           try {
               sh 'checkov -f deploy.yaml --use-enforcement-rules -o cli -o junitxml --output-file-path console,results.xml --prisma-api-url https://api.eu.prismacloud.io  --bc-api-key ${ACCESS_KEY}::${SECRET_KEY} --repo-id rbenavente/evil.yaml --branch main'
                              junit skipPublishingChecks: true, testResults: 'results.xml'
              } catch (err) {
                              junit skipPublishingChecks: true, testResults: 'results.xml'
                              throw err
                          }
   }  
 }  


  }
      stage('Deploy evilpetclinic') {
        sh 'kubectl create ns evil --dry-run -o yaml | kubectl apply -f -'
        sh 'kubectl delete --ignore-not-found=true -f files/deploy.yml -n evil'
        sh 'kubectl apply -f files/deploy.yml -n evil'
        sh 'sleep 30'
    }
    
   stage('Run bad Runtime attacks') {
        sh('chmod +x files/runtime_attacks.sh && ./files/runtime_attacks.sh')
    }
	
    stage('Run bad HTTP stuff for WAAS to catch') {
        sh('chmod +x files/waas_attacks.sh && ./files/waas_attacks.sh')
    }
 }    
   
  
