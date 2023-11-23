pipeline {

    agent any
    stages {
        stage ('checkout') {
            parallel {
                stage('CheckoutTemplate') {
                    steps {
                        sh 'mkdir -p core-template'
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
                            echo "-------------------*******Manifest Display*******-------------------"    
                            sh "cat resources-dev.yaml"       
                            echo "-------------------*******Manifest Display End*******-------------------"                     
                        }
                        }
                    }
                    stage('Dev Manifest Deploy') {//Manifest Deploy for Dev Env Values
                        steps {
                        dir("deployments/dev-env")
                        { withCredentials([string(credentialsId: 'svc_principal', variable: 'PASSWORD')]) {
                                    sh "az login --service-principal -u 01c88a92-5b91-4664-a88a-6b39f649ce24 -p $PASSWORD  --tenant b41b72d0-4e9f-4c26-8a69-f949f367c91d"
                            }
                            sh "az account set --subscription 40d692fa-1896-4563-9f1d-95ae2fa15c38"
                            sh "az aks get-credentials --resource-group dhl_awow_demo --name dhl-awow-demo-cluster --admin"
                            sh "kubectl apply -f resources-dev.yaml"                            
                        }
                        }
                    }
                    stage('Dev Manifest Rollback Approval') {//Manifest Rollback Approve for Dev Env Values
                        steps {
                                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                                timeout(time: 100, unit: "MINUTES") {
                                    input message: 'Do you want to approve the deployment?', ok: 'Yes'
                                }
	                            echo "Initiating Rollback"
	                            sh "kubectl rollback deployment nginx -n nginx-dev"
                            }
                        }
                    }
                }
                
            }

            stage('QA Build') {
                stages {
                    stage('QA Manifest Create') { //Manifest Build for QA Env Values
                        steps {
                        dir("deployments/qa-env")
                        {  
                            sh "helm template  ../../core-template/core-template -f values.yaml > resources-qa.yaml" 
                            echo "-------------------*******Manifest Display*******-------------------"    
                            sh "cat resources-qa.yaml"       
                            echo "-------------------*******Manifest Display End*******-------------------"                         
                        }
                        }
                    }

                    stage('QA Manifest Deploy') {//Manifest Deploy for QA Env Values
                        steps {
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            timeout(time: 100, unit: "MINUTES") {
	                            input message: 'Do you want to approve the deployment?', ok: 'Yes'
	                        }}
                        dir("deployments/qa-env")
                        { withCredentials([string(credentialsId: 'svc_principal', variable: 'PASSWORD')]) {
                                    sh "az login --service-principal -u 01c88a92-5b91-4664-a88a-6b39f649ce24 -p $PASSWORD  --tenant b41b72d0-4e9f-4c26-8a69-f949f367c91d"
                            }
                            sh "az account set --subscription 40d692fa-1896-4563-9f1d-95ae2fa15c38"
                            sh "az aks get-credentials --resource-group dhl_awow_demo --name dhl-awow-demo-cluster --admin"
                            sh "kubectl apply -f resources-qa.yaml"                            
                        }
                        }
                    }

                    stage('QA Manifest Rollback Approval') {//Manifest Rollback Approve for QA Env Values
                        steps {
                            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            timeout(time: 100, unit: "MINUTES") {
	                            input message: 'Do you want to approve the deployment?', ok: 'Yes'
	                        }
			                echo "Initiating Rollback"
                            sh "kubectl rollback deployment nginx -n nginx-qa"
                            }
                        }
                    }
                
                }


            }

        }

    }}
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