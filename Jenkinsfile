pipeline {
    agent {
      docker { image 'mcr.microsoft.com/dotnet/sdk:8.0' }
    }
    environment {
        DOTNET_CLI_HOME = "/tmp"  // Requerida para el contenedor de dotnet
    }
    triggers { // Sondear repositorio a intervalos regulares
        pollSCM('* * * * *')
    }

    stages {
        stage('Restore') {
            steps {
                sh "dotnet restore"
            }
        }
        stage('Clean') {
            steps {
                sh "dotnet clean"
            }
        }
        stage('Compile') {
            steps {
                sh "dotnet build"
            }
        }
        stage('Unit test') {
            steps {
                sh "dotnet test --logger:junit"
            }
            post {
                failure {
                    mail to: 'team@example.com',
                         subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                         body: "Something is wrong with ${env.BUILD_URL}"
                }
                success {
                    junit '**/TestResults/TestResults.xml'
                }
            }
        }
        // stage("SonarQube Analysis") {
        //     steps {
        //         withSonarQubeEnv('SonarQubeDockerServer') {
        //             //withEnv(['PATH="${env.PATH}:/tmp/.dotnet/tools"']) {
        //             sh "env"
        //             sh "export PATH=$PATH:/tmp/.dotnet/tools && dotnet sonarscanner begin /d:sonar.cs.vscoveragexml.reportsPaths=coverage.xml"
        //             sh "export PATH=$PATH:/tmp/.dotnet/tools && dotnet build --no-incremental"
        //             sh 'export PATH=$PATH:/tmp/.dotnet/tools && dotnet-coverage collect "dotnet test" -f xml -o "coverage.xml"'
        //             sh "export PATH=$PATH:/tmp/.dotnet/tools && dotnet sonarscanner end"
        //         }
        //         timeout(2) { // time: 2 unit: 'MINUTES'
        //           // In case of SonarQube failure or direct timeout exceed, stop Pipeline
        //           waitForQualityGate abortPipeline: waitForQualityGate().status != 'OK'
        //         }
        //     }
        // }
        stage('Release') {
            steps {
                sh "dotnet build --configuration Release --property:Version=1.0.${currentBuild.number}"
            }
             post {
                success {
                    archiveArtifacts '**/bin/Release/**/*.exe,**/bin/Release/**/*.dll'
                }
            }
        }
    }
    post {
        always {
            mail to: 'team@example.com',
                subject: "Resultado Paso a produccion: ${currentBuild.fullDisplayName}",
                body: "${env.BUILD_URL} has result ${currentBuild.currentResult}"
        }
    }
}