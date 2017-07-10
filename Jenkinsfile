def sh_out(cmd){
   sh(script: cmd, returnStdout:true).trim()
}

node ('master') {
  withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'k8s-aws-ecr', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
    stage ('Prepare') {
      sh_out("""
      git reset --hard
      git clean -fxd
      git tag -d \$(git tag) &> /dev/null
      """)
      checkout scm
      gitcommit_email = sh_out('git --no-pager show -s --format=\'%ae\'')
      currentBuild.displayName = "#${BUILD_NUMBER} ${gitcommit_email}"
      sh_out('''
      curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
      chmod +x ./kubectl
      ${HOME}/.local/bin/aws s3 cp s3://k8s-hub-tikal-io/hub.tikal.io/kconfig .
      ''')
    }
    stage ('Deploy to K8S') {
			cmd = """
					\$(find \$(pwd) -type f -name "*-deployment.yaml" | xargs #-I {} --no-run-if-empty awk '{if(/app:/) print \$2}' < "{}" ;)
			"""
			appNamesa = sh(returnStdout: true, script: cmd)
      /*sh_out("""
      appNames=\$(find \$(pwd) -type f -name "*-deployment.yaml" | xargs -I {} --no-run-if-empty awk '{if(/app:/) print \$2}' < "{}" ;)
			print "appnames: $appNames"
      for appName in \${appNames};
      do
        ./kubectl apply -f \${appName}-service.yaml --kubeconfig=\$(pwd)/kconfig
        until [[ \${available} ]];
        do
          sleep 10
          available=\$(./kubectl get svc -l app=\${appName} --namespace=fuze -o json --kubeconfig=\$(pwd)/kconfig | python -c 'import json, sys; print(json.loads(sys.stdin.read())["items"][0]["metadata"]["labels"]["app"])')
        done
        echo "Service Available: \${appName}"
      done
      """)*/
      sh(script: """
      #!/bin/bash
      find \$(pwd) -type f -name "*-deployment.yaml" | xargs -I {} --no-run-if-empty kubectl apply -f {} --kubeconfig=\$(pwd)/kconfig;
      """)*/
    }
  
}
