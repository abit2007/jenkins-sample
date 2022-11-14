pipeline {
  agent any
  stages {
    stage('') {
      steps {
        sh '''node(\'lisytest\') {
    stage(\'Clone\') {
        echo "1.Clone Stage"
        git url: "https://github.com/abit2007/jenkins-sample.git"
        script {
            build_tag = sh(returnStdout: true, script: \'git rev-parse --short HEAD\').trim()
        }
    }
    stage(\'Test\') {
      echo "2.Test Stage"

    }
    stage(\'Build\') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t 11.0.1.184/jenkins-demo/jenkins-demo:${build_tag} ."
    }
    stage(\'Push\') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: \'dockerharbor\', passwordVariable: \'dockerHubPassword\', usernameVariable: \'dockerHubUser\')]) {
            sh "docker login 11.0.1.184 -u ${dockerHubUser} -p ${dockerHubPassword}"
            sh "docker push 11.0.1.184/jenkins-demo/jenkins-demo:${build_tag}"
        }
    }
    stage(\'Deploy to dev\') {
        echo "5. Deploy DEV"
		sh "sed -i \'s/<BUILD_TAG>/${build_tag}/\' k8s-dev-harbor.yaml"
        sh "sed -i \'s/<BRANCH_NAME>/${env.BRANCH_NAME}/\' k8s-dev-harbor.yaml"
//        sh "bash running-devlopment.sh"
        sh "kubectl apply -f k8s-dev-harbor.yaml  --validate=false"
	}	
	stage(\'Promote to qa\') {	
		def userInput = input(
            id: \'userInput\',

            message: \'Promote to qa?\',
            parameters: [
                [
                    $class: \'ChoiceParameterDefinition\',
                    choices: "YES\\nNO",
                    name: \'Env\'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        if (userInput == "YES") {
            sh "sed -i \'s/<BUILD_TAG>/${build_tag}/\' k8s-qa-harbor.yaml"
            sh "sed -i \'s/<BRANCH_NAME>/${env.BRANCH_NAME}/\' k8s-qa-harbor.yaml"
//            sh "bash running-qa.sh"
            sh "kubectl apply -f k8s-qa-harbor.yaml --validate=false"
            sh "sleep 6"
            sh "kubectl get pods -n qatest"
        } else {
            //exit
        }
    }
	stage(\'Promote to pro\') {	
		def userInput = input(

            id: \'userInput\',
            message: \'Promote to pro?\',
            parameters: [
                [
                    $class: \'ChoiceParameterDefinition\',
                    choices: "YES\\nNO",
                    name: \'Env\'
                ]
            ]
        )
        echo "This is a deploy step to ${userInput}"
        if (userInput == "YES") {
            sh "sed -i \'s/<BUILD_TAG>/${build_tag}/\' k8s-prod-harbor.yaml"
            sh "sed -i \'s/<BRANCH_NAME>/${env.BRANCH_NAME}/\' k8s-prod-harbor.yaml"
//            sh "bash running-production.sh"
            sh "cat k8s-prod-harbor.yaml"
            sh "kubectl apply -f k8s-prod-harbor.yaml --record --validate=false"
        }
    }
}'''
        }
      }

    }
  }