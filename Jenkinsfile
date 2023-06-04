#!/bin/groovy

// def existingEndpointId = ""

pipeline {
  agent any
  parameters{
     string(name: 'existingEndpointId', defaultValue: '5', description: 'If this stack does not exist, then you must specify a host. DockingBay1 = ?, DockingBay2 = ?, Dockingbay3 (local) = ?')
  }

  stages {
    stage('Get JWT Token') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'portainer', usernameVariable: 'PORTAINER_USERNAME', passwordVariable: 'PORTAINER_PASSWORD')]) {
              def json = """
                  {"Username": "$PORTAINER_USERNAME", "Password": "$PORTAINER_PASSWORD"}
              """
              def jwtResponse = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: json, url: "https://portainer2.alderaan.co:9443/api/auth"
              def jwtObject = new groovy.json.JsonSlurper().parseText(jwtResponse.getContent())
              env.JWTTOKEN = "Bearer ${jwtObject.jwt}"
          }
        }
        echo "${env.JWTTOKEN}"
      }
    }
    stage('Delete old Stack') {
      steps {
        script {

          // Get all stacks
          String existingStackId = ""
          // String existingEndpointId = ""
          if("true") {
            def stackResponse = httpRequest httpMode: 'GET', ignoreSslErrors: true, url: "https://portainer2.alderaan.co:9443/api/stacks", validResponseCodes: '200', consoleLogResponseBody: true, customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
            def stacks = new groovy.json.JsonSlurper().parseText(stackResponse.getContent())
            
            stacks.each { stack ->
              if(stack.Name == "plexmm") {
                existingStackId = stack.Id
                existingEndpointId = stack.EndpointId
              }
            }
          }

          if(existingStackId?.trim()) {
            // Delete the stack
            def stackURL = """
              https://portainer2.alderaan.co:9443/api/stacks/$existingStackId?endpointId=$existingEndpointId
            """
            httpRequest acceptType: 'APPLICATION_JSON', validResponseCodes: '204', httpMode: 'DELETE', ignoreSslErrors: true, url: stackURL, customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]

          }

        }
      }
    }
    stage('Deploy new stack to Portainer') {
      steps {
        script {

          def createStackJson = ""

          // if(existingEndpointId == ""){
          //   existingEndpointId = 2
          // }

          withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {

            createStackJson = """
              {"Name": "plexmm", "SwarmID": "", "RepositoryURL": "https://github.com/$GITHUB_USERNAME/docker-pmm", "ComposeFile": "docker-compose.yml", "RepositoryAuthentication": true, "RepositoryUsername": "$GITHUB_USERNAME", "RepositoryPassword": "$GITHUB_PASSWORD"}
            """

          }

          if(createStackJson?.trim()) {
            httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: createStackJson, url: "https://portainer2.alderaan.co:9443/api/stacks?method=repository&type=2&endpointId=$existingEndpointId", customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
          }

        }
      }
    }
  }
}
