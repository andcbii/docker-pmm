#!/bin/groovy

//String existingStackId = ""
//String existingEndpointId ""

pipeline {
  agent any
  parameters{
    string(name: 'existingEndpointId', defaultValue: '')
  }
  stages {
    stage('Get JWT Token') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: '28289f1f-f98a-44b8-b33c-a43630272b35', usernameVariable: 'PORTAINER_USERNAME', passwordVariable: 'PORTAINER_PASSWORD')]) {
              def json = """
                  {"Username": "$PORTAINER_USERNAME", "Password": "$PORTAINER_PASSWORD"}
              """
              def jwtResponse = httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: json, url: "https://portainer.alderaan.co/api/auth"
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
          if("true") {
            def stackResponse = httpRequest httpMode: 'GET', ignoreSslErrors: true, url: "https://portainer.alderaan.co/api/stacks", validResponseCodes: '200', consoleLogResponseBody: true, customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
            def stacks = new groovy.json.JsonSlurper().parseText(stackResponse.getContent())
            
            stacks.each { stack ->
              if(stack.Name == "haproxy") {
                existingStackId = stack.Id
                existingEndpointId = stack.EndpointId
              }
            }
          }

          if(existingStackId?.trim()) {
            // Delete the stack
            def stackURL = """
              https://portainer.alderaan.co/api/stacks/$existingStackId?endpointId=$existingEndpointId
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

          // Stack does not exist
          // Generate JSON for when the stack is created
          withCredentials([usernamePassword(credentialsId: '0a5dd0f0-7df9-4b62-b07b-0d8ce44062c1', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_PASSWORD')]) {
            // def swarmResponse = httpRequest acceptType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'GET', ignoreSslErrors: true, consoleLogResponseBody: true, url: "https://portainer.alderaan.co/api/endpoints/${env.existingEndpointId}/docker/swarm", customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
            // def swarmInfo = new groovy.json.JsonSlurper().parseText(swarmResponse.getContent())

            createStackJson = """
              {"Name": "HAProxy", "SwarmID": "", "RepositoryURL": "https://github.com/$GITHUB_USERNAME/Docker", "ComposeFilePathInRepository": "/HAProxy/", "RepositoryAuthentication": true, "RepositoryUsername": "$GITHUB_USERNAME", "RepositoryPassword": "$GITHUB_PASSWORD"}
            """

          }

          if(createStackJson?.trim()) {
            httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', validResponseCodes: '200', httpMode: 'POST', ignoreSslErrors: true, consoleLogResponseBody: true, requestBody: createStackJson, url: "https://portainer.alderaan.co/api/stacks?method=repository&type=2&endpointId=${env.existingEndpointId}", customHeaders:[[name:"Authorization", value: env.JWTTOKEN ], [name: "cache-control", value: "no-cache"]]
          }

        }
      }
    }
  }
}
