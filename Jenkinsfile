import java.text.SimpleDateFormat

pipeline {
  options {
    buildDiscarder logRotator(numToKeepStr: '5')
    disableConcurrentBuilds()
  }
  agent {
    kubernetes {
      cloud "go-demo"
      label "go-demo-5"
      yamlFile "KubernetesPod.yaml"
    }      
  }
  environment {
    image = "vfarcic/go-demo-5"
    project = "go-demo-5"
    domain = "192.168.0.189.nip.io"
    cmAddr = "cm.192.168.0.189.nip.io"
  }
  stages {
    stage("build") {
      steps {
        container("docker") {
          script {
            currentBuild.displayName = new SimpleDateFormat("yy.MM.dd").format(new Date()) + "-${env.BUILD_NUMBER}"
          }
          k8sBuildImageBeta(image, false)
        }
      }
    }
    stage("func-test") {
      steps {
        container("helm") {
          k8sUpgradeBeta(project, domain, "--set replicaCount=2 --set dbReplicaCount=1")
        }
        container("kubectl") {
          k8sRolloutBeta(project)
        }
        container("golang") {
          k8sFuncTestGolang(project, domain)
        }
      }
      post {
        failure {
          container("helm") {
            k8sDeleteBeta(project)
          }
        }
      }
    }
    stage("release") {
      when {
          branch "master"
      }
      steps {
        container("docker") {
          k8sPushImage(image, false)
        }
        container("helm") {
          k8sPushHelm(project, "", cmAddr)
        }
      }
    }
  }
}
