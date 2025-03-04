pipeline {
  environment {
    APP_NAME='app-test'
    PROJECT_ID = 'cicd-security-integration'
    // COSIGN_PRIVATE=credentials('test-app-sign-private')
    // dceb7c5b-c952-43d9-a06f-9dfcc7887bd3 
  }

  agent {
      kubernetes {
        yaml '''
          apiVersion: v1
          kind: Pod
          metadata:
            labels:
              app: jenkins-nodejs
          spec:
            containers:
              - name: cosign
                image: jitesoft/cosign:latest
                command:
                - cat
                tty: true
              - name: zap 
                image: zaproxy/zap-stable
                command:
                - cat
                tty: true
                volumeMounts:
                - mountPath: "/zap/wrk"
                  name: "workspace-volume"
                  readOnly: false
              - name: nodejs
                image: sonarsource/sonar-scanner-cli
                command:
                - cat
                tty: true
              - name: trivy
                image: aquasec/trivy:latest
                imagePullPolicy: Always
                securityContext:
                  privileged: true
                  runAsUser: 0
                command:
                - sleep
                args:
                - 9999999
              - name: kubectl
                image: yonadev/jnlp-slave-k8s-helm:latest
                imagePullPolicy: Always
                command:
                - sleep
                args:
                - 9999999
              - name: buildah
                image: quay.io/buildah/stable:v1.23.1
                command:
                - cat
                tty: true
                securityContext:
                  privileged: true
                volumeMounts:
                  - name: varlibcontainers
                    mountPath: /var/lib/containers

              - name: dp
                image: jenkins/jnlp-agent-docker
                command:
                - sleep 30000
                tty: true
                securityContext:
                  privileged: true
                volumeMounts:
                  - name: dp-check-pvc
                    mountPath: /
            volumes:
              - name: varlibcontainers
              - name: dp-check-pvc
                persistentVolumeClaim:
                  claimName: dependency-check-data-pvc
            resources:
              requests:
                memory: "1024Mi"
                cpu: "300m"
              limits:
                memory: "1024Mi"
                cpu: "300m"
  '''
    }
  }
  stages {


    stage ('Dependency-Check') {
      steps {
        container('dp') {
          dependencyCheck additionalArguments: ''' 
            -o "./" 
            -s "./"
            -f "ALL" 
            --prettyPrint
            --disableBundleAudit
            --disableRubygems
            ''', 
            odcInstallation: 'dependency-check',
            nvdCredentialsId: '2e967df9-bcc8-4d21-b5f6-ec3ed457e6a9'

          dependencyCheckPublisher failedNewCritical: 10, failedNewHigh: 20, failedNewLow: 80, failedNewMedium: 40, failedTotalCritical: 10, failedTotalHigh: 20, failedTotalLow: 80, failedTotalMedium: 40, pattern: 'dependency-check-report.xml', stopBuild: true, unstableNewCritical: 5, unstableNewHigh: 10, unstableNewLow: 40, unstableNewMedium: 20, unstableTotalCritical: 5, unstableTotalHigh: 10, unstableTotalLow: 40, unstableTotalMedium: 20 
        }
      }
    }
  }
}

