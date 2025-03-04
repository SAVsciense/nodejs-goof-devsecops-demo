pipeline {
  environment {
    APP_NAME='app-test'
    PROJECT_ID = 'cicd-security-integration'
    COSIGN_PRIVATE=credentials('test-app-sign-private')
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
            nodeSelector:
              cloud.google.com/gke-nodepool: "spot"
            containers:
              - name: cosign
                image: jitesoft/cosign:latest
                command:
                - cat
                tty: true
                env: 
                - name: COSIGN_PASSWORD
                  valueFrom: 
                    secretKeyRef: 
                      name: cosign-pass 
                      key: password
              - name: zap 
                image: owasp/zap2docker-stable
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
            //     volumeMounts:
            //       - name: varlibcontainers
            //         mountPath: /var/lib/containers
            // volumes:
            //   - name: varlibcontainers
            //   - name: dp-check-pvc
            //     persistentVolumeClaim:
            //       claimName: dependency-check-data-pvc
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


  //   stage('SonarQube Scan') {
  //     steps {
  //       container('nodejs') {
  //         script {
  //           def scannerHome = tool name: 'sonarqube-scanner';
  //           withSonarQubeEnv('sq1') {
  //             sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=test-app -Dsonar.sources=. -Dsonar.css.node=. "
  //           }
  //         }
  //       }
  //     }
  //   }
  //   stage('Wait for SonarQube') {
  //     steps {
  //       timeout(time: 1, unit: 'MINUTES') {
  //         waitForQualityGate abortPipeline: true
  //       }
  //     }
  //   }
  //
  //   stage("dockerfile_check") {
  //     steps {
  //       container("trivy") {
  //         sh 'mkdir -p reports'
  //         sh 'trivy config Dockerfile --format template --template "@junit.tpl" -o junit-report.xml '
  //         junit skipPublishingChecks: true, testResults: 'junit-report.xml', checksName: 'ANTON'
  //         sh 'trivy config Dockerfile --exit-code 1 --severity CRITICAL '
  //       }
  //     }
  //   }
  //
  //
  //
  //   stage('Build') {
  //     steps {
  //       container('buildah') {
  //         sh "buildah build -t ${env.PROJECT_ID}/${env.APP_NAME}:${env.BUILD_ID} ."
  //         sh """
  //         rm -rf "target/images"
  //         mkdir -p "target/images"
  //         buildah push \
  //           --format docker \
  //           ${env.PROJECT_ID}/${env.APP_NAME}:${env.BUILD_ID} \
  //           docker-archive:target/images/test-app.tar 
  //         ls -la
  //         """
  //       }
  //     }
  //   }
  //
  //   stage("docker_image_scan") {
  //     steps {
  //       container('buildah') {
  //         container("trivy") {
  //           sh 'mkdir -p reports'
  //           sh 'trivy image --format template --template "@junit.tpl" -o junit-report.xml --input target/images/test-app.tar'
  //
  //           junit skipPublishingChecks: true, testResults: 'junit-report.xml'
  //           // Scan again and fail on CRITICAL vulns
  //           sh 'trivy image --exit-code 1 --severity CRITICAL --input target/images/test-app.tar'
  //         }
  //       }
  //     }
  //   }
  //
  //   stage('Login to GAR') {
  //     steps {
  //       container('buildah') {
  //         sh "cat ${env.STORAGE_ADMIN} | buildah login -u _json_key --password-stdin ${env.LOCATION}-docker.pkg.dev"
  //       }
  //       container('cosign') {
  //         sh "cat ${env.STORAGE_ADMIN} | cosign login -u _json_key --password-stdin ${env.LOCATION}-docker.pkg.dev"
  //       }
  //     }
  //   }
  //
  //   stage('tag image and push') {
  //     steps {
  //       container('buildah') {
  //         sh "buildah tag ${env.PROJECT_ID}/${env.APP_NAME}:${env.BUILD_ID} ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${env.APP_NAME}/app:${env.BUILD_ID}"
  //         sh "buildah tag ${env.PROJECT_ID}/${env.APP_NAME}:${env.BUILD_ID} ${LOCATION}-docker.pkg.dev/${PROJECT_ID}/${env.APP_NAME}/app:latest"
  //         sh "buildah push ${env.LOCATION}-docker.pkg.dev/${env.PROJECT_ID}/${env.APP_NAME}/app:${env.BUILD_ID}"
  //         sh "buildah push ${env.LOCATION}-docker.pkg.dev/${env.PROJECT_ID}/${env.APP_NAME}/app:latest"
  //       }
  //     }
  //   }
  //
  //   stage('sign image') {
  //     steps {
  //       container('cosign') {
  //         withCredentials([file(credentialsId: 'test-app-sign-private', variable: 'key')]) {
  //           sh 'cat `echo $key` > /tmp/cosign.key'
  //           sh 'cosign sign --key /tmp/cosign.key $LOCATION-docker.pkg.dev/$PROJECT_ID/$APP_NAME/app:latest'
  //         }
  //       }
  //     }
  //   }
  //
  //   stage("manifest_check") {
  //     steps {
  //       container("trivy") {
  //         sh 'mkdir -p reports'
  //         sh 'trivy config deployment.yaml --format template --template "@junit.tpl" -o junit-report.xml '
  //         junit skipPublishingChecks: true, testResults: 'junit-report.xml'
  //         sh 'trivy config deployment.yaml --exit-code 1 --severity CRITICAL '
  //       }
  //     }
  //   }
  //
  //   stage('Deploy to test') {
  //     steps {
  //     container("kubectl") {
  //         withKubeConfig([credentialsId: env.CREDENTIALS_ID, serverUrl: env.KUBERNETES_API_URL, clusterName: env.CLUSTER_NAME]) {
  //           sh 'kubectl apply -f deployment.yaml'
  //         }
  //       }
  //     }
  //   }
  //
  //   stage("dast") {
  //     steps {
  //       container("zap") {
  //         sh '''
  //         while true; do
  //           curl http://app.forever-free.online/ &> /dev/null
  //           if [[ $? -eq 0 ]]; then
  //               echo "Site is up!"
  //               break
  //           fi
  //         done
  //         '''
  //         sh 'zap-baseline.py -t http://app.forever-free.online/ -I -r zap-report.html'
  //         sh '''
  //           mkdir /zap/wrk/workspace/report
  //           cp /zap/wrk/zap-report.html /zap/wrk/workspace/report/zap-report.html
  //         '''
  //       }
  //     }
  //   }
  //
  //
  //   stage('Deploy to prod') {
  //     steps {
  //     container("kubectl") {
  //         withKubeConfig([credentialsId: env.CREDENTIALS_ID, serverUrl: env.KUBERNETES_API_URL_PROD, clusterName: env.CLUSTER_NAME_PROD ]) {
  //           sh 'kubectl apply -f deployment.yaml'
  //         }
  //       }
  //     }
  //   }
  //
  }
  post {
    always {
      publishHTML([allowMissing: true, 
        alwaysLinkToLastBuild: true, 
        keepAll: true,
        reportDir: '../report', 
        reportFiles: 'zap-report.html',
        reportName: 'ZAP_REPORT',
        reportTitles: '', 
        useWrapperFileDirectly: true])
    }
  }
}
