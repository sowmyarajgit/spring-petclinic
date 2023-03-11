def registry = 'https://sowmyaraj15.jfrog.io'
//def imageName = 'sowmyaraj15.jfrog.io/twitter-docker/twitter'
//def version   = '2.0.3'
pipeline {
    agent{
        node{
            label "ansible"
         }
    }
    
   // environment {
     //    PATH = "/opt/apache-maven-3.9.0/bin/:$PATH"
    //}

    stages {
        stage('gitclone') {
            steps {
              git branch: 'main', url: 'https://github.com/sowmyarajgit/spring-petclinic.git'
            }
        }
        stage('build'){
            
            steps{
            sh 'mvn clean test'
        }        
        }
    
    stage ("Sonar Analysis") {
            environment {
               scannerHome = tool 'admin_sonarscanner'
            }
            steps {
                echo '<--------------- Sonar Analysis started  --------------->'
                withSonarQubeEnv('admin_sonarcube') {    
                    sh "${scannerHome}/bin/sonar-scanner"
                echo '<--------------- Sonar Analysis stopped  --------------->'
                }    
               
            }   
        }
         stage("Quality Gate") {
            steps {
                script {
                  echo '<--------------- Sonar Gate Analysis Started --------------->'
                    timeout(time: 1, unit: 'HOURS'){
                       def qg = waitForQualityGate()
                        if(qg.status !='OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }  
                  echo '<--------------- Sonar Gate Analysis Ends  --------------->'
                }
            }
        }
               
         stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog1111"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "pet_clinic-libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
            }
        }   
    }  
    // stage(" Docker Build ") {
    //       steps {
    //         script {
    //            echo '<--------------- Docker Build Started --------------->'
    //            app = docker.build(imageName+":"+version)
    //            echo '<--------------- Docker Build Ends --------------->'
    //         }
    //       }
    //     }

    //             stage (" Docker Publish "){
    //         steps {
    //             script {
    //                echo '<--------------- Docker Publish Started --------------->'  
    //                 docker.withRegistry(registry, 'jfrog1111'){
    //                     app.push()
    //                 }    
    //                echo '<--------------- Docker Publish Ended --------------->'  
    //             }
    //         }
    //     }

    //      stage(" Deploy ") {
    //    steps {
    //      script {
    //         echo '<--------------- Helm Deploy Started --------------->'
    //         sh  './deploy.sh'
    //         echo '<--------------- Helm deploy Ends --------------->'
    //      }
    //    }
    // }
    }

    
    }
