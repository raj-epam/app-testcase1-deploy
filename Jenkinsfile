pipeline {

    agent any
    // trigger {

    // }

    // parameters {
    //     choice(name: 'Environment' )
    // }
    stages {
        stage ('checkout') {
            parallel {
                    stage('CheckoutTemplate') {
        steps {
            sh 'mkdir -p core-template'z
            dir("core-template")
            {
                git branch: "main",
                credentialsId: "raj-epam",
                url: 'https://github.com/raj-epam/templates.git'
            }
        }
    }
        stage('CheckoutCode') {
        steps {
            sh 'mkdir -p deployments'
            dir("deployments")
            {
                git branch: "main",
                credentialsId: "raj-epam",
                url: 'https://github.com/raj-epam/app-testcase1-deploy.git'
            }
        }
    }

            }
        }

                stage('deployment phase') {
        parallel {
            // Dev Build Phase
            stage('Dev Build') {
                stages {
                    stage('Dev Manifest Create') { //Manifest Build for Dev Env Values
                        steps {
                        dir("deployments/dev-env")
                        {  
                            sh "helm template  ../../core-template/core-template -f values.yaml > resources-dev.yaml"                            
                        }
                        }
                    }
                    stage('Dev Manifest Scan') {//Manifest Scan for Dev Env Values
                        steps {
                        echo "Manifest Display"
                        echo "Manifest scan"
                        }
                    }
                    stage('Dev Manifest Deploy') {//Manifest Deploy for Dev Env Values
                        steps {
                        dir("deployments/dev-env")
                        {  
                            sh "kubectl apply -f resources-dev.yaml"                            
                        }
                        }
                    }
                    stage('Dev Manifest Rollback Approval') {//Manifest Rollback Approve for Dev Env Values
                        steps {
                        timeout(time: 100, unit: "MINUTES") {
	                    input message: 'Do you want to approve the deployment?', ok: 'Yes'
	                }
			
	                echo "Initiating Rollback"
                        }
                    }
                    stage('Dev Manifest Rollback') {//Manifest Build for Dev Env Values
                        steps {
                        echo "kubectl rollback deployment app-name"
                        }
                    }
                }
                
            }


            }

}

    }
        post {
        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: true,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }

}
