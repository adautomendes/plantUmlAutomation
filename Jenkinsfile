def plantumlVersion='1.2024.8'
def diagramList = [
    'Sequence Diagram': 'sequenceDiagram',
    'Use Case Diagram': 'usecaseDiagram',
    'Class Diagram': 'classDiagram',
    'Activity Diagram': 'activityDiagram',
    'Component Diagram': 'componentDiagram',
    'Deployment Diagram': 'deploymentDiagram'
]

def htmlTemplate = """
<html>
<head>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <link type="text/css" rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css"  media="screen,projection"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>PlantUML Output Diagrams</title>
</head>

<body>
    <div class="container">
        <div class="row">
            {{diagrams}}
        </div>
    </div>
    <script type="text/javascript" src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
</body>
</html>
"""

def markdownTemplate = """
# PlantUML Output Diagrams
{{diagrams}}
"""

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
                    diagramList.each { title, file ->
                        println "File: ${file} ==> Title: ${title}"
                    }
                }
            }
        }
        stage('Generating PlantUML diagrams') {
            steps {
                script {
                    diagramList.each { title, file ->
                        sh "java -jar plantuml.jar -Playout=smetana diagrams/${file}.puml"
                        println "File ${file} processed."
                    }
                }
            }
        }
        stage('Output files') {
            parallel {
                stage('Generating HTML document') {
                    steps {
                        script {
                            def htmlDiagrams = ''
                            diagramList.each { title, file ->
                                htmlDiagrams += """<div class="col s4 m4 l4">
    <div class="card">
        <div class="card-image">
            <img src="diagrams/${file}.png">
            <span class="card-title"></span>
        </div>
        <div class="card-content">
            <p>${title}</p>
        </div>
    </div>
</div>"""
                            }

                            htmlTemplate = htmlTemplate.replace('{{diagrams}}', htmlDiagrams)
                            println "${htmlTemplate}"
                            writeFile(file: 'output.html', text: htmlTemplate)
                        }
                    }
                }
                stage('Generating MD document') {
                    steps {
                        script {
                            def markdownDiagrams = ''
                            diagramList.each { title, file ->
                                markdownDiagrams += "## ${title}\n![${title}](diagrams/${file}.png)\n"
                            }

                            markdownTemplate = markdownTemplate.replace('{{diagrams}}', markdownDiagrams)
                            println "${markdownTemplate}"
                            writeFile(file: 'output.md', text: markdownTemplate)
                        }
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
            archiveArtifacts artifacts: '**/*.html, **/*.png, **/*.md', allowEmptyArchive: true
        }
    }
}