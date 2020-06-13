node {
    def app

    stage('Clone repository') {
        /* Clone repository to our workspace */

        checkout scm
    }
    
   stage ('Deploy Policy') {
        /* Deploy runtime policy */
        sh "curl -H "Content-Type: application/json" -X PUT -u administrator:XXXXX -d @aqua_profile.json http://a84d335a29f2a11eaa485025822714ea-958075476.ap-southeast-1.elb.amazonaws.com:8080/api/v1/securityprofiles/$(cat aqua_policy_name.txt)"
    }
    
   stage('Package Dev Image') {
        docker.withRegistry('https://dstubked-docker.jfrog.io', 'jfrog') {
             /* Pull and retag for prod namespace */
            sh "docker pull dstubked-docker.jfrog.io/orders-nginx-dev:good"
            sh "docker tag dstubked-docker.jfrog.io/orders-nginx-dev:good dstubked-docker.jfrog.io/orders-nginx-prod:good"
        }
    }
    
    stage ('Aqua Scan Prod Namespace') {
         /* One final scan to check and register before push into prod namespace */
        aqua customFlags: '--layer-vulnerabilities', hideBase: false, hostedImage: '', localImage: 'dstubked-docker.jfrog.io/orders-nginx-prod:good', locationType: 'local', notCompliesCmd: '', onDisallowed: 'fail', policies: '', register: true, registry: 'JFrog', showNegligible: false
    }
    
    stage('Push into Prod Namespace') {
        docker.withRegistry('https://dstubked-docker.jfrog.io', 'jfrog') {
            /* Push into prod namespace */
            sh "docker push dstubked-docker.jfrog.io/orders-nginx-prod:good"
        }
    }
}
