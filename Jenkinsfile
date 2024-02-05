pipeline {
    agent {
        // Define the agent to use Windows, adjust as necessary
        label 'windows'
    }
    environment {
        // Define your environment variables here
        SOLUTION = '**/*.sln'
        BUILD_PLATFORM = 'Any CPU'
        BUILD_CONFIGURATION = 'Release'
        // Make sure to securely add your Veracode API ID and KEY to Jenkins credentials
        VERACODE_API_ID = credentials('VERACODE_API_ID')
        VERACODE_API_KEY = credentials('VERACODE_API_KEY')
    }
    stages {
        stage('Restore NuGet Packages') {
            steps {
                script {
                    bat 'nuget restore %SOLUTION%'
                }
            }
        }
        stage('Build Solution') {
            steps {
                script {
                    bat 'msbuild %SOLUTION% /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="%WORKSPACE%\\$(Build.ArtifactStagingDirectory)" /p:Platform="%BUILD_PLATFORM%" /p:Configuration="%BUILD_CONFIGURATION%"'
                }
            }
        }
        stage('Veracode Static Pipeline Scan') {
            steps {
                script {
                    // Downloading Veracode Pipeline Scanner
                    bat 'curl -sSO https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                    bat 'unzip -o pipeline-scan-LATEST.zip'
                    // Scanning the artifact
                    bat 'java -jar pipeline-scan.jar -vid %VERACODE_API_ID% -vkey %VERACODE_API_KEY% -f "%WORKSPACE%\\$(Build.ArtifactStagingDirectory)\\Verademo-dotnet.zip" || true'
                    // Assuming you want to save the scan results as "results.json" in the workspace
                }
            }
        }
        stage('Publish Artifacts') {
            steps {
                archiveArtifacts artifacts: 'results.json', onlyIfSuccessful: true
            }
        }
    }
    post {
        always {
            // Cleanup if needed
        }
    }
}
