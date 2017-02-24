node {
             // define commands
             def ocCmd = "/opt/ocp/bin/oc"
             def mvnHome = tool 'M3'
             def mvnCmd = "${mvnHome}/bin/mvn"

             sh "${ocCmd} login -u springboot -p springboot --server=https://master1-c8f8.oslab.opentlc.com --insecure-skip-tls-verify=true"
            
             stage 'Build'
             git branch: 'master', url: 'https://github.com/taksato-redhat/springboot-app.git'
             def v = version()
             sh "${mvnCmd} clean install -DskipTests=true"
             
             stage 'Test and Analysis'
             parallel (
                 'Test': {
                     sh "${mvnCmd} test"
                     step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                 },
                 'Static Analysis': {
                     sh "${mvnCmd} sonar:sonar -Dsonar.host.url=http://localhost:9000 -DskipTests=true"
                 }
             )
             
             stage 'Deploy DEV'
             sh "${ocCmd} delete bc,dc,svc,route -l app=springboot-app -n springboot-dev"
             // create build. override the exit code since it complains about exising imagestream
             sh "${ocCmd} new-build --name=springboot-app --image-stream=springboot-rhel7 --binary=true --labels=app=springboot-app -n springboot-dev || true"
             // build image
             sh "${ocCmd} start-build springboot-app --from-file=target/app.jar --wait=true -n springboot-dev"
             // deploy image
             sh "${ocCmd} new-app springboot-app:latest -e APP_PARAM_1=TEST_PARAM -n springboot-dev"
             sh "${ocCmd} expose svc/springboot-app -n springboot-dev"

             emailext attachLog: true, body: 'Jenkins build has finished.', compressLog: true, subject: 'Jenkins: springboot-app-pipeline', to: 'taksato@redhat.com'

}

def version() {
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            matcher ? matcher[0][1] : null
}
