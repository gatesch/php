def gitCommit() {
        sh "git rev-parse HEAD > GIT_COMMIT"
        def gitCommit = readFile('GIT_COMMIT').trim()
        sh "rm -f GIT_COMMIT"
        return gitCommit
    }

def answerQuestion = ''

    node {
        // Checkout source code from Git
        stage 'Checkout'
        checkout scm

        // Build Docker image
        stage 'Build'
        sh "docker build -t harbor.tesch.loc/library/php:${gitCommit()} ."

        // Login to DTR 
        stage 'Login'
        withCredentials(
            [[
                $class: 'UsernamePasswordMultiBinding',
                credentialsId: 'dtr',
                passwordVariable: 'DTR_PASSWORD',
                usernameVariable: 'DTR_USERNAME'
            ]]
        ){ 
        sh "docker login -u admin -p ${env.DTR_PASSWORD}  harbor.tesch.loc"}

        // Push the image 
        stage 'Push'
        sh "docker push harbor.tesch.loc/library/php:${gitCommit()}"

//        clean all
        try {
          stage('Deployment') {
                sh "kubectl create ns php"
        	sh "kubectl create -f limits.yaml"
        	sh "kubectl create deployment php-safe -n php --image=harbor.tesch.loc/library/php:${gitCommit()}"
        	sh "kubectl expose deployment php-safe --port=80 --name=php-service -n php"
        	sh "kubectl create -f php-ingress.yaml"
        	sh "kubectl autoscale deployment php-safe --cpu-percent=50 --min=1 --max=10 -n php"
}}
          catch(e) {
                     build_ok = false
                     echo e.toString()
}

        // update the image in the deployement
        stage 'update'
        sh "kubectl set image deployment php-safe -n php php=harbor.tesch.loc/library/php:${gitCommit()}"
    }

// remember to copy the dtr cert in /etc/pki/ca-trust/source/anchors
// run update-ca-trust
// run systemctl restart atomic-openshift-*
