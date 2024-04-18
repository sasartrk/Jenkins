def sendMail(name, id, test_results) {
    echo name
    echo id
    echo test_results
}
def results
pipeline {
    
    agent any

    parameters {
        string defaultValue: 'pipeline', name: 'GIT_BRANCH'
        booleanParam defaultValue: true, name: 'RUN_TEST'
        booleanParam defaultValue: true, name: 'SEND_MAIL'
    }
    stages {
        stage('Dynamic') {
            when {
               branch: "feature/multi/*"
            }
            steps {
                    echo 'Dynamic'
            }
        }
        
        stage('Download') {
            steps {
                cleanWs()
                
                withCredentials ([usernamePassword(credentialsId: 'CRED_MIRKO', passwordVariable: 'psw', usernameVariable: 'usr')]) {
                    echo psw
                    echo usr
                }
                
                dir('pipeline') {
                    git (
                        branch: params.GIT_BRANCH,
                        url: 'https://github.com/KLevon/jenkins-course'
                    )
                }
             
                rtDownload (
                    serverId: "Artifactory",
                    spec: '''{ 
                        "files": [
                            {
                              "pattern" : "generic-local/printer.zip",
                              "target": "printer.zip"
                            }
                          ]
                    }'''
                )
                
                unzip (
                    zipFile: "printer.zip",
                    dir: "pipeline"
                )
            }
                
        }
    
    
        stage('Build') {
            steps {
                    echo 'Build'
                    bat (
                        script: """
                        cd pipeline
                        Makefile.bat
                        """
                    )
            }
        }
        
        stage('Tests') {
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                    echo 'Tests'
                    script{
                        
                      
                        def niz = ["printer", "scanner", "main"]
                        results = ""
                        for (element in niz) {
                            results += bat (
                                script: """
                                cd pipeline
                                Tests.bat ${element}
                                """,
                                returnStdout: true
                            ).trim()
                        }
                    }
            }
        }
        
        stage('Publish') {
            steps {
                    echo 'Publish'
                    script
                    {
                        zip (
                            zipFile: "pipeline.zip",
                            archive: true,
                            dir: "pipeline"
                        )
                    }
                    
                    rtUpload (
                        serverId: "Artifactory",
                        spec: """{ 
                        "files": [
                            {
                              "pattern" : "pipeline.zip",
                              "target": "generic-local/${env.BUILD_ID}/"
                            }
                          ]
                        }"""
                    )
            }
        }
    }
    post {
        success {
            script {
                if (params.SEND_MAIL == true)
                {
                    sendMail(env.JOB_NAME, "${env.BUILD_ID}", results)
                }
            }
             
        }
    }
}
