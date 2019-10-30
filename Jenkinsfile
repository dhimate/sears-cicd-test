node {
   def mvnHome
  
	
   /////////////////////////////////////////////////////////////////////////////////
   //Only changed the following 5 parameter as needed                             //
   //more information about options below can be found at                         //
   //https://github.com/orgs/SearsHomeServices/teams/mulesoft/discussions?pinned=1//
   /////////////////////////////////////////////////////////////////////////////////
   /// ALL VARIABLES BELOW ARE CASE SENSITIVE ////
   ///////////////////////////////////////////////
   def projectName="sears-cicd-test"
   def muleenv = "dev" //Options "dev","stage","prod","qa"
   def region = "us-east-1"  // Options "us-east-1" (use this for non prod)  "us-east-2" (prod)
   def workerType = "SMALL"  // Options "MICRO" , "SMALL" , "MEDIUM" , "LARGE" ,'XLARGE' ,"XXLARGE" ,"4XLARGE" 
   def workers = 1  // Options --> number of workers to deploy. (1 for non prod) (2 for prod / Stage(load test))
   def muleenvironment = "Development" //Options "Development", "QA", "Stage", "Production", "Design"
	
	
   stage('Preparation') { // for display purposes

     
              //git 'https://github.com/dhimate/gateway-proxy-domain.git'
        	      //sh "'mvn' -Dmaven.test.failure.ignore clean package install"
              git (
                  url: 'https://github.com/dhimate/sears-cicd-test.git'
                  )
              sh 'git config credential.helper store'
              sh 'git clean -f'
              sh 'git checkout .'
            

        
   } 
	
   
   stage('Build') {

	  
      if ("$BRANCH_NAME"=='master') {
           sh 'mvn clean package -DskipMunitTests'
           archiveArtifacts 'target/*.jar'
                
      } else if ("$BRANCH_NAME"!='master') {
         def pom = readMavenPom file: 'pom.xml'
         POM_VERSION = readMavenPom().getVersion()
         BUILD_RELEASE_VERSION = readMavenPom().getVersion().replace("-SNAPSHOT", "")
         echo POM_VERSION
         echo BUILD_RELEASE_VERSION
         def versionList = BUILD_RELEASE_VERSION.tokenize(".")
         def newRelease = Eval.me(versionList[2])+1
         echo ''+newRelease
         def version = "${BUILD_RELEASE_VERSION}"
         stage('masterRelease') {
            withMaven(maven : "${HSMAVEN}", mavenSettingsConfig: 'aef5450c-39ae-4804-924a-cd7e30ad7fd7', mavenLocalRepo: '.repository') {
                sh "mvn clean versions:set -DnewVersion=${BUILD_RELEASE_VERSION}"
                sh 'mvn clean deploy -DskipMunitTests'
            }
         }
         stage('tag') {
            sh "git tag -f ${projectName}-${version} -m 'JenkinsRelease'"
            sh "git push -u origin master --tag"
         }
         stage('updatedevelopmentPOMVersion') {
            sh "git checkout ."
            sh "git checkout development"
            sh "git clean -f"
            sh "git reset --hard"
            sh "git checkout ."
            sh "git pull origin development"
            withMaven(maven : "${HSMAVEN}", mavenSettingsConfig: 'aef5450c-39ae-4804-924a-cd7e30ad7fd7', mavenLocalRepo: '.repository') {
                sh 'mvn clean versions:set -DnewVersion=${versionList[0]}.${versionList[1]}.${newRelease}-SNAPSHOT'
            }
            sh "git add -- pom.xml"
            sh "git commit -m 'jenskinsChangingVersion'"
            sh "git push -u origin development"
         }
      }
   
   stage('Preparation') { // for display purposes
	 println "in prep step"
	 step ([$class: 'CopyArtifact', projectName: env.JOB_NAME]);
     	  
	   }
	   
   stage('Deploy') {

      println "Deploy starting credentials ..."
 
      withCredentials([
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'anypointAccountPassword',
          usernameVariable: 'ANY_USERNAME', passwordVariable: 'ANY_PASSWORD'],
          [$class: 'UsernamePasswordMultiBinding', credentialsId: 'anypointPlatformClient',
          usernameVariable: 'CLIENT_USERNAME', passwordVariable: 'CLIENT_PASSWORD'],
          [$class: 'StringBinding', credentialsId: 'muleKeyDev', variable: 'MULE_KEY'],
          [$class: 'StringBinding', credentialsId: 'businessGroup', variable: 'BUSINESS_GROUP'],
	  [$class: 'StringBinding', credentialsId: "${muleenvironment}", variable: 'MULE_ENVIRONMENT'],
	  [$class: 'StringBinding', credentialsId: "${muleenv}", variable: "ENV"],
          [$class: 'StringBinding', credentialsId: "${workerType}", variable: 'WORKER_TYPE'],
          [$class: 'StringBinding', credentialsId: "${region}", variable: 'WORKER_REGION'],
          [$class: 'StringBinding', credentialsId: "${workers}", variable: 'WORKER_AMOUNT'],
          [$class: 'StringBinding', credentialsId: 'dev', variable: 'DEV_VALUE']
          ]) {
          
            println "Starting Maven Deploy..."

        
            withMaven(maven : "${HSMAVEN}", mavenSettingsConfig: 'aef5450c-39ae-4804-924a-cd7e30ad7fd7',mavenLocalRepo: '.repository') {

	    println "Starting Maven Deploy..."

       
		  sh 'mvn --batch-mode mule:deploy "-Dartifact=target/'+readMavenPom().getArtifactId()+'-'+readMavenPom().getVersion()+'-mule-application.jar" "-DbusinessGroup=${BUSINESS_GROUP}" "-Dmule.env=${ENV}" "-Dregion=${WORKER_REGION}" "-DworkerType=${WORKER_TYPE}" "-Dworkers=${WORKER_AMOUNT}" "-Dmule.environment=${MULE_ENVIRONMENT}" "-DapplicationName='+readMavenPom().getArtifactId()+'-${ENV}" "-Dusername=${ANY_USERNAME}" "-Dpassword=${ANY_PASSWORD}" "-Dvault.key=${MULE_KEY}" "-Dclient.id=${CLIENT_USERNAME}" "-Dclient.secret=${CLIENT_PASSWORD}" -X -e'
	    }
         }
   }}


}
