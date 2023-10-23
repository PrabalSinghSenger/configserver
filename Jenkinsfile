 node{
  def repourl = "${REGISTRY_URL}/${PROJECT_ID}/${ARTIFACT_REGISTRY}"
  def mvnhome = tool name: 'maven', type:'maven'
  def mvncmd = "${mvnhome/bin/mvn} "
   stage('Checkout'){
   	checkout([$class : 'GitSCM',
   			  branches :[[name : '*/main']],
   			  extensions :[],
   			  userRemoteConfigs : [[credentialsId:'git',
   			  url:'https://github.com/PrabalSinghSenger/configserver.git']]
   	)
   }
   stage('Build and Push Image'){
     withCredentials([file(credentialsId:'gcp',variable:'GC_KEY')]){
      sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
      sh "gcloud auth configure-docker us-central1-docker.pkg.dev"
      sh "${mvncmd} clean install jib:build -DREPO_URL=${repourl}"
      
     }
   }
   stage('Deploy'){
    sh "sed -i 's|IMAGE_URL|${repourl}|g' deployment.yaml"
    step([$class:'KubernatesEngineBuilder',
          projectId:env.PROJECTID,
          clusterName:env.CLUSTER,
          location: env.ZONE,
          manifestPattern: 'deployment.yaml',
          credentialsId : env.PROJECT_ID,
          verifyDeployments: true])
   }
 }