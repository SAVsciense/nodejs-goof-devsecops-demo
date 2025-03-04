pipeline {
  environment {
    APP_NAME='app-test'
    PROJECT_ID = 'cicd-security-integration'
    // COSIGN_PRIVATE=credentials('test-app-sign-private')
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
            tolerations:
            - key: "instance_type"
              operator: "Exists"
              effect: "NoSchedule"
            containers:
              - name: cosign
                image: jitesoft/cosign:latest
                command:
                - cat
                - sleep 30000
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
        container('jnlp') {
          dependencyCheck additionalArguments: ''' 
            -o "./" 
            -s "./"
            -f "ALL" 
            --prettyPrint
            --disableBundleAudit
            --disableRubygems
            ''', odcInstallation: 'dependency-check'

          dependencyCheckPublisher failedNewCritical: 10, failedNewHigh: 20, failedNewLow: 80, failedNewMedium: 40, failedTotalCritical: 10, failedTotalHigh: 20, failedTotalLow: 80, failedTotalMedium: 40, pattern: 'dependency-check-report.xml', stopBuild: true, unstableNewCritical: 5, unstableNewHigh: 10, unstableNewLow: 40, unstableNewMedium: 20, unstableTotalCritical: 5, unstableTotalHigh: 10, unstableTotalLow: 40, unstableTotalMedium: 20 
        }
      }
    }
  }
}

