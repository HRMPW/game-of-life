@Library("github.com/pwolfbees/pipeline") _

pipeline{
   agent docker:'pwolf/cjptower'
   
   stages {
      stage('Build and Package') {
         steps {
            sh "mvn clean package -Dtest=WhenYouStoreGamesInADatabase -DfailIfNoTests=false"
         }
      }
      stage ('Publish Artifact to S3'){
         steps {
               wrap([$class: 'AmazonAwsCliBuildWrapper', credentialsId: 's3-cjptower', defaultRegion: 'us-west-2']) {
               sh 'aws s3 cp gameoflife-web/target/gameoflife.war s3://cjptower/gameoflife.war --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers'
               }
         }  
      }
      stage ('Promote Build'){
         steps {
            script{
               env.TARGET = input message: 'Deploy Application?', ok: 'Go! Go! Go!', parameters: [choice(choices: 'Development\nStaging\nProduction', description: 'Pick Target Environment for Deployment.', name: 'TARGET')]
            }
         }
      }
      stage("Run Tower Playbook"){
         steps {
            withTower(host:"https://104.198.10.204", credentials:"tower-cli"){
               sh "tower-cli job launch --job-template=41 --monitor --extra-vars='target='${env.TARGET}"
         }
       }
      }   
   }
}
