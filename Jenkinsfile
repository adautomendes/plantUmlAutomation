def plantumlVersion='1.2024.8'
def fileList = [
    'sequenceDiagram.puml',
    'usecaseDiagram.puml',
    'classDiagram.puml',
    'activityDiagram.puml',
    'componentDiagram.puml',
    'deploymentDiagram.puml'
]

pipeline {
    agent any

    stages {
        stage('Clean WS') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adautomendes/plantUmlAutomation', credentialsId: 'Github - adautomendes'
            }
        }
        stage('Download latest PlantUML') {
            steps {
                script {
                    sh "curl -L https://github.com/plantuml/plantuml/releases/download/v${plantumlVersion}/plantuml-${plantumlVersion}.jar -o plantuml.jar"
                }
            }
        }
        stage('Print input file list') {
            steps {
                script {
                    fileList.each { println "File ${it} will be processed." }
                }
            }
        }
        stage('Generating PlantUML diagram') {
            steps {
                script {
                    for (file in fileList) {
                        sh "java -jar plantuml.jar -Playout=smetana diagrams/${file}"
                        println "File ${file} processed."
                    }
                }
            }
        }
    }
    post {
        failure {
            echo 'The pipeline failed.'
        }
        success {
            echo 'Pipeline concluded successfully!'
        }
        always {
            archiveArtifacts artifacts: '**/*.png', allowEmptyArchive: true
        }
    }
}
