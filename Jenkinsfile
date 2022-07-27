def PROJECT_NAME = "icistr-gateway"
def GIT_OPS_NAME = "icistr-gateway-gitops"
def gitOpsUrl = "gitlab.dspace.kt.co.kr/icis-tr/sa/${GIT_OPS_NAME}.git"
def gitLabOrigin = "gitlab.dspace.kt.co.kr/icis-tr/sa/${PROJECT_NAME}.git"
def gitLabUrl = "https://${gitLabOrigin}"
def NEXUS_URL = 'https://nexus.dspace.kt.co.kr'
def gitLabAccessToken = "8XsBxG_DDHv5si9vCfAS"
def TAG = getTag()
def ENV = getENV()


/////////////////////////////
pipeline {
        
    agent {
        docker {
            image 'nexus.dspace.kt.co.kr/icis/icistr-sa-build-tools:v1.0.4'
            args '-u root:root -v /var/run/docker.sock:/var/run/docker.sock'
            registryUrl 'https://nexus.dspace.kt.co.kr'
            registryCredentialsId 'icistr-sa-nexus'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
            steps {
                script{
                    // git branch: opsBranch, credentialsId: "SA-CICD-TEST", url: gitLabUrl
                    docker.withRegistry(NEXUS_URL, "icistr-sa-nexus") {
                        configFileProvider([configFile(fileId: 'icis-tr-maven_setting', variable: 'maven_settings')]) {
                            sh  """
                                pwd
                                chmod 777 ./mvnw
                                echo 'TAG => ' ${TAG}
                                echo 'ENV => ' ${ENV}
                                skaffold build -p ${ENV} -t ${TAG}
                            """
                        }
                    }
                }
            }
        }

        stage('GitOps update') {
            steps{
                print "======kustomization.yaml tag update====="
                script{
                    sh """   
                        cd ~
                        rm -rf ./${GIT_OPS_NAME}    
                        git clone https://gitlab-ci-token:${gitLabAccessToken}@${gitOpsUrl} 
                        cd ./${GIT_OPS_NAME}
                        git checkout ${ENV}
                        kustomize edit set image nexus.dspace.kt.co.kr/icis/icis-apigw:${TAG}
                        git config --global user.email "sa-admin@icistr.com"
                        git config --global user.name "sa-admin"
                        git add .
                        git commit -am 'update image tag ${TAG}'
                        git push origin ${ENV}
                    """
                }
                print "git push finished !!!"
            }
        }
    }
}

def  getTag(){
    
    def TAG
    def opsBranch
    def gitBranch = "${scm.branches[0].name.split("/")[1]}"
    DATETIME_TAG = new Date().format('yyyyMMddHHmmss')  

    if(gitBranch == "develop" || gitBranch == "olv" ){
      TAG = "${gitBranch}-${DATETIME_TAG}"
    }else{
      TAG = "${params.Branch.replaceFirst(/^.*\//,'')}-${DATETIME_TAG}"
    }
      
    return TAG
}

def getENV(){
    
    def deployENV
    def opsBranch
    def paramBranch = "${params.Branch}"
    def gitBranch = "${scm.branches[0].name.split("/")[1]}"

    if( gitBranch == "develop" ){
         deployENV = "dev"
    }else if( gitBranch == "olv" ){
         deployENV = gitBranch
    }else{
        opsBranch = paramBranch.replaceFirst(/^.*\//,'')
        deployENV =  opsBranch.replaceAll(/\-.*/,'')
    }
    
    return deployENV
}
