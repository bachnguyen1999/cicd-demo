node {
    stage ("Checkout SCM") {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main' ]],
            extensions: scm.extensions,
            userRemoteConfigs: [[
                url: 'git@github.com:bachnguyen1999/cicd-gke.git',
                credentialsId: 'github-key'
                    ]]
            ])    
            }

   
   stage ('Maven build') {
         def mvnHome = tool name: 'maven', type: 'maven'
         def mvnCMD = "${mvnHome}/bin/mvn "
         sh "${mvnCMD} clean package"           
        }
    stage ('Build image') { 
            sh "docker build -t terraform-labs-infras/springboot:${BUILD_NUMBER} ."
        }
    stage ('Push image to gcr') {
            sh "gcloud auth configure-docker"
            sh "docker tag terraform-labs-infras/springboot:${BUILD_NUMBER} gcr.io/terraform-labs-infras/springboot:${BUILD_NUMBER}"
            sh "docker push gcr.io/terraform-labs-infras/springboot:${BUILD_NUMBER}"
    }    
    stage('Update GIT') {
        script {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') 
            {
                withCredentials([usernamePassword(credentialsId: 'gitlogin', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    def encodedPassword = URLEncoder.encode("$GIT_PASSWORD",'UTF-8')
                    sh "git config user.email bachtcdn@gmail.com"
                    sh "git config user.name bachnguyen1999"
                    sh "sed -i 's+terraform-labs-infras/springboot.*+terraform-labs-infras/springboot:${env.BUILD_NUMBER}+g' spring-boot.yaml"
                    sh "git add ."
                    sh "git commit -m 'jenkinsbuild: ${env.BUILD_NUMBER}'"
                    sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/${GIT_USERNAME}/cicd-demo.git HEAD:main"
                }
                    
                  }
                
            }
    }        
}
