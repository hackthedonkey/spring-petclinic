// Generate a random id for pod label to avoid waiting for executor. Just start create pod right away.
// See this workaround from issue: https://issues.jenkins.io/browse/JENKINS-39801
import static java.util.UUID.randomUUID
def uuid = randomUUID() as String
def myid = uuid.take(8)

pipeline {
  environment {
    APP_VER = "v1.0.${BUILD_ID}"
    // HARBOR_URL = ""
    DEPLOY_GITREPO_USER = "hackthedonkey"
    DEPLOY_GITREPO_URL = "github.com/${DEPLOY_GITREPO_USER}/spring-petclinic-helmchart.git"
    DEPLOY_GITREPO_BRANCH = "main"
    DEPLOY_GITREPO_TOKEN = credentials('my-github')
  }
  agent any
  stages {
      stage('Cloning Git') {
      steps {
        git 'https://github.com/hackthedonkey/spring-petclinic.git'
      }
    }
    stage('Build') {
      steps {
          echo sh(script: 'env|sort', returnStdout: true)
          sh """
            mvn -B -ntp -T 2 package -DskipTests -DAPP_VERSION=${APP_VER}
            """
     }
    }
    stage('Building Image') {
      steps {
          sh "docker build -t hackthedonkey/library/samples/spring-petclinic:v1.0.${env.BUILD_ID}" 
      }
    }
    stage('Approval') {
      input {
        message "Proceed to deploy?"
        ok "YES"
      }
      steps {
        echo "Update helm chart to trigger GitOps-based deployment..."
      }
    }
    stage('GitOps-based Deploy') {
      steps {
          sh """
            git config --global user.name $env.GIT_AUTHOR_NAME
            git config --global user.email $env.GIT_AUTHOR_EMAIL
            git clone https://$env.DEPLOY_GITREPO_USER:$env.DEPLOY_GITREPO_TOKEN@$env.DEPLOY_GITREPO_URL --branch=$env.DEPLOY_GITREPO_BRANCH deploy
            # After cloning
            cd deploy
            # update values.yaml
            sed -i -r 's,repository: (.+),repository: hackthedonkey/library/samples/spring-petclinic,' values.yaml
            sed -i 's/tag: v1.0.*/tag: v1.0.${env.BUILD_ID}/' values.yaml
            cat values.yaml
            git commit -am 'bump up version number'
            git remote set-url origin https://$env.DEPLOY_GITREPO_USER:$env.DEPLOY_GITREPO_TOKEN@$env.DEPLOY_GITREPO_URL
            git push origin main
          """
      }
    }
  }
}
