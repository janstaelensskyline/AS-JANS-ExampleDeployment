class Globals
{
  static String uploadOutput
}

pipeline
{
    agent any
    options
    {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
        timeout(time: 120, unit: 'MINUTES')
    }


    environment
    {
        dotnettoolsfolder = 'C:\\dotnet-tools\\'
    }

    stages
    {
        stage('Install Package Creation')
        {
            steps
            {
                bat "dotnet tool update \"Skyline.DataMiner.CICD.Tools.Packager\" --local"
            }
        }

        stage('Install Catalog Upload')
        {
            steps
            {
                bat "dotnet tool update \"Skyline.DataMiner.CICD.Tools.CatalogUpload\" --local --version 1.1.1-Beta1"
            }
        }

        stage('Install DataMiner Deploy')
        {
            steps
            {
                bat "dotnet tool update \"Skyline.DataMiner.CICD.Tools.DataMinerDeploy\" --local --version 1.0.1-Beta2"
            }
        }

        stage('Create DMAPP')
        {
            steps
            {
                bat "dotnet dataminer-package-create dmapp \"${WORKSPACE}\" --name HelloFromJenkins -o \"${WORKSPACE}\" --type automation"
            }
        }

       stage('Upload DMAPP')
       {
            steps
            {
              withCredentials([string(credentialsId: 'DeployExampleToken', variable: 'DATAMINER_CATALOG_TOKEN')])
              {
                script
                {
                    uploadOutput = bat(returnStdout: true, script: "@dotnet dataminer-catalog-upload --path-to-artifact \"${WORKSPACE}\\HelloFromJenkins.dmapp\"")
                    uploadOutput = uploadOutput.trim()
                }
              }
            }
       } 

       stage('Check output')
       {
            steps
            {
                echo(uploadOutput)
            }
       }

       stage("Deploy DMAPP")
       {
            steps
            {
              withCredentials([string(credentialsId: 'DeployExampleToken', variable: 'DATAMINER_CATALOG_TOKEN')])
              {
                bat "dotnet dataminer-package-deploy from-catalog --artifact-id \"${uploadOutput}\""
              }
            }
       }
    }
    post
    {
        cleanup
        {
            cleanWs()
        }
    }
}