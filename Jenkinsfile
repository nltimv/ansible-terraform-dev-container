podTemplate(namespace: 'jenkins-ci', yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    stage('Pull repository') {
      git url: 'https://github.com/nltimv/ansible-terraform-dev-container.git', branch: 'main'
      container('kaniko') {
        stage('Build image') {
          sh '''
            /kaniko/executor --context "`pwd`" --destination nltimv/ansible-terraform-dev-container:$BUILD_NUMBER --destination nltimv/ansible-terraform-dev-container:latest
          '''
        }
      }
    }
  }
}
podTemplate(namespace: 'jenkins-ci', yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: ansible
        image: nltimv/ansible-terraform-dev-container:$BUILD_NUMBER
        command:
        - sleep
        args:
        - 9999999
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
      restartPolicy: Never
''') {
  node(POD_LABEL) {
    stage('Ansible version') {
      container('kaniko') {
        sh '''
          ansible --version
        '''
      }
    }
  }
}