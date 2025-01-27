podTemplate(yaml: '''
kind: Pod
metadata:
  name: kaniko
  namespace: ibrahim
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: IfNotPresent
    env:
     - name: container
       value: "docker"
    command:
     - /busybox/cat
    tty: true
'''){
  node(POD_LABEL) {
    def IMAGE_PUSH_DESTINATION="ibrahim999/examen"
    stage('Build with Kaniko a docker image and publish it into the Docker hub') {
        checkout scm
        container(name: 'kaniko', shell: '/busybox/sh') {
            withCredentials([file(credentialsId: 'docker-credentials', variable: 'DOCKER_CONFIG_JSON')]) {
                withEnv(['PATH+EXTRA=/busybox',"IMAGE_PUSH_DESTINATION=${IMAGE_PUSH_DESTINATION}"]) {
                    sh '''#!/busybox/sh
                        cp $DOCKER_CONFIG_JSON /kaniko/.docker/config.json
                        /kaniko/executor --context `pwd` --destination $IMAGE_PUSH_DESTINATION
                    '''
                }
            }
        }
    }

    stage('Linting') {
        container(name: 'kaniko', shell: '/bin/sh') {
            sh "flake8 --ignore=E501,E231 *.py"
            sh "pylint --disable=C0301 --disable=C0326 test_app.py"
        }
    }

    stage('Unit Testing') {
        container(name: 'kaniko', shell: '/bin/sh') {
            sh "python -m unittest --verbose --failfast"
        }
    }
        node (POD_LABEL) {
        stage('Deploy the app in Horizon Kubernetes Cluster') {
            checkout scm
            container('kubectl') {
                withKubeConfig([namespace: "ibrahim"]) {
                    sh 'kubectl apply -f deployment.yaml -n ibrahim'
                }
            }
        }
    }
  }
}
