def currentDate = new Date().format("yyyy-MM-dd HH:mm:ss")
def plantumlVersion='1.2024.8'
def diagramsPageId = '5603334'
def diagramsPageVersion = 1

def diagramList = [
    'Sequence Diagram': 'sequenceDiagram',
    'Use Case Diagram': 'usecaseDiagram',
    'Class Diagram': 'classDiagram',
    'Activity Diagram': 'activityDiagram',
    'Component Diagram': 'componentDiagram',
    'Deployment Diagram': 'deploymentDiagram',
    'YAML Diagram': 'yamlDiagram',
    'JSON Diagram': 'jsonDiagram'
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

def createConfluencePayloadTemplate = """
{
    "type": "page",
    "title": "PlantUML {{date}}",
    "space": {
        "key": "EAI"
    },
    "ancestors": [
        {
            "id": "5603329"
        }
    ],
    "body": {
        "wiki": {
            "value": "{{template}}",
            "representation": "wiki"
        }
    }
}
"""

pipeline {
    agent any

    stages {
        stage('Clean WS') {
            when { expression { params.cleanWs == true } }
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
                    if (!fileExists('plantuml.jar')) {
                        sh "curl -L https://github.com/plantuml/plantuml/releases/download/v${plantumlVersion}/plantuml-${plantumlVersion}.jar -o plantuml.jar"
                    } else {
                        println "PlantUML jar already exists in workspace."
                    }
                }
            }
        }
        stage('Print input file list') {
            steps {
                script {
                    diagramList.each { title, file ->
                        println "Input file: diagrams/${file}.puml | Title: ${title} | Output file: diagrams/${file}.png"
                    }
                }
            }
        }
        stage('Generating PlantUML diagrams') {
            steps {
                script {
                    diagramList.each { title, file ->
                        if (!fileExists("diagrams/${file}.png")) {
                            sh "cat diagrams/${file}.puml | java -jar plantuml.jar -Playout=smetana -pipe > diagrams/${file}.png"
                            println "File diagrams/${file}.puml processed."
                        } else {
                            println "File diagrams/${file}.png already exists on workspace."
                        }
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
        stage('Create Confluence page') {
            when { expression { params.createNewPage == true } }
            steps {
                script {
                    // Handling template with all diagrams
                    def createConfluencePageTemplate = ""
                    diagramList.each { title, file ->
                        createConfluencePageTemplate += "h2. ${title}\\n!${file}.png!\\n"
                    }

                    createConfluencePayloadTemplate = createConfluencePayloadTemplate.replace('{{date}}', currentDate)
                    createConfluencePayloadTemplate = createConfluencePayloadTemplate.replace('{{template}}', createConfluencePageTemplate)

                    def createConfluencePageResponse
                    withCredentials([usernamePassword(credentialsId: 'Confluence_UserPass', usernameVariable: 'user', passwordVariable: 'password')]) {
                        createConfluencePageResponse = readJSON(text: sh(script: """
                                curl -u ${user}:${password} -H 'Content-Type: application/json' http://confluence:8090/rest/api/content/ --data '${createConfluencePayloadTemplate}'
                            """, returnStdout: true).trim())
                    }

                    println "${createConfluencePageResponse}"
                    println "Confluence page id: ${createConfluencePageResponse.id}"

                    diagramList.each { title, file ->
                        def postDiagramAttachmentResponse = ""
                        withCredentials([usernamePassword(credentialsId: 'Confluence_UserPass', usernameVariable: 'user', passwordVariable: 'password')]) {
                            postDiagramAttachmentResponse = readJSON(text: sh(script: """
                                    curl -u ${user}:${password} -X POST -H 'X-Atlassian-Token: nocheck' -F 'file=@"diagrams/${file}.png"' -F 'comment="${title} ${currentDate}"' http://confluence:8090/rest/api/content/${createConfluencePageResponse.id}/child/attachment
                                """, returnStdout: true).trim())
                        }

                        println "Response: ${postDiagramAttachmentResponse}"
                        println "File diagrams/${file}.png uploaded."
                    }
                }
            }
        }
        stage('Get Confluence page version') {
            when { expression { params.createNewPage == false } }
            steps {
                script {
                    def response
                    withCredentials([usernamePassword(credentialsId: 'Confluence_UserPass', usernameVariable: 'user', passwordVariable: 'password')]) {
                        response = readJSON(text: sh(script: """
                                curl -u ${user}:${password} http://confluence:8090/rest/api/content/'${diagramsPageId}'?expand=body.storage,version,space
                            """, returnStdout: true).trim())
                    }

                    diagramsPageVersion += response.version.number

                    println "${response}"
                    println "Diagrams Page next version: ${diagramsPageVersion}"
                }
            }
        }
        stage('Replace diagrams') {
            when { expression { params.createNewPage == false } }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'Confluence_UserPass', usernameVariable: 'user', passwordVariable: 'password')]) {
                        diagramList.each { title, file ->
                            def getResponse = readJSON(text: sh(script: """
                                    curl -u ${user}:${password} -X GET -H 'Accept: application/json' http://confluence:8090/rest/api/content/${diagramsPageId}/child/attachment?filename=${file}.png
                                """, returnStdout: true).trim())

                            if (getResponse.size != 0) {
                                def attachmentId = getResponse.results[0].id
                                def attachmentTitle = getResponse.results[0].title

                                println "Deleting old attachment id=${attachmentId}, name=${attachmentTitle}"

                                def deleteResponse = sh(script: """
                                        curl -w "%{http_code}" -u ${user}:${password} -X DELETE -H 'Content-Type: application/json' -H 'Accept: application/json' http://confluence:8090/rest/api/content/${attachmentId}?status=current
                                    """, returnStdout: true).trim()
                                
                                if (deleteResponse != '204') {
                                    error("Delete of ${file}.png was not successful.")
                                }
                            }


                            def postResponse = readJSON(text: sh(script: """
                                    curl -u ${user}:${password} -X POST -H 'X-Atlassian-Token: nocheck' -F 'file=@"diagrams/${file}.png"' -F 'comment="${title} ${currentDate}"' http://confluence:8090/rest/api/content/${diagramsPageId}/child/attachment
                                """, returnStdout: true).trim())

                            println "Response: ${postResponse}"
                            println "File diagrams/${file}.png uploaded."
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