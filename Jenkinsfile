pipeline {
    agent any

    parameters {
        string(name: 'NAMESPACE', defaultValue: 'sandbox', description: 'Enter the namespace')
        string(name: 'GIT_CONTEXT', defaultValue: 'git:github.com/Srinadhgoud/test.git#refs/heads/main', description: 'Enter the git context')
        string(name: 'REGISTRY_USERNAME', defaultValue: 'sahithiperka', description: 'Enter the registry username')
        string(name: 'IMAGE_NAME', defaultValue: 'test2', description: 'Enter the image name')
        string(name: 'TAG_NAME', defaultValue: 'T1', description: 'Enter the tag name')
    }

    environment {
        KUBECONFIG = '/var/jenkins_home/.kube/config'
        YAML_FILE_PATH = '/tmp/deployment.yaml'  // Temp file to store the YAML content
    }

    stages {
        stage('Docker image Build and Push to ACR') {
            steps {
                script {
                    // Clone the repository and checkout the main branch
                    git branch: 'main', url: 'https://github.com/Srinadhgoud/kaniko-project2.git'
                    //get the commit id
                    def commitId = sh(script: "git rev-parse HEAD", returnStdout: true).trim()

                    // Print the commit ID to the console
                    echo "Git Commit ID: ${commitId}"
                    // Define YAML content directly in the pipeline
                    def yamlContent = """
apiVersion: v1
kind: Pod
metadata:
  name: ${commitId}
spec:
  containers:
  - name: ${commitId}
    image: gcr.io/kaniko-project/executor:v1.22.0-debug
    args:
    - "--dockerfile=Dockerfile"
    - "--context=${params.GIT_CONTEXT}"
    - "--destination=${params.REGISTRY_USERNAME}/${params.IMAGE_NAME}:${params.TAG_NAME}"
    - "--verbosity=info"  # Enables detailed debug logs
    - "--log-format=text"  # Optional, can use json for structured logs
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
      secretName: regcred
      items:
        - key: .dockerconfigjson
          path: config.json
"""

                    // Write the YAML content to a file
                    writeFile(file: YAML_FILE_PATH, text: yamlContent)

                    // Apply the YAML file using kubectl
                    withCredentials([file(credentialsId: 'abdfc3d2-e7cb-41cc-b12d-dda520d61ab4', variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f ${YAML_FILE_PATH}"
                    withCredentials([file(credentialsId: 'abdfc3d2-e7cb-41cc-b12d-dda520d61ab4', variable: 'KUBECONFIG')]) {
                        // Wait for the pod to be in 'running' state
                        // Loop until the pod is in 'running' state
                        sh """
                        until kubectl get pod ${commitId} -n ${params.NAMESPACE} -o=jsonpath='{.status.phase}' | grep -q 'Running'; do
                            echo 'Waiting for pod to run...'
                            sleep 3
                        done
                        echo 'Pod is running, fetching logs...'
                        kubectl logs -f ${commitId} -n ${params.NAMESPACE}
                        """
                        }
                    }
                }
            }
        }
    }
}
