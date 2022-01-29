node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    def dockerTag = 'latest'
    stage("Docker Tag"){
        gittags = sh(returnStdout: true, script: 'git tag -l --contains HEAD').trim()
        if (gittags?.trim()){
            echo " there are one tag : ${gittags}"
            dockerTag = gittags
        }
    }

    dir(""){
        stage('clean') {
            sh "chmod +x ./mvnw"
            sh "./mvnw clean"
        }

        stage('packaging') {
            sh "./mvnw verify -Pprod -DskipTests"
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
        }

        stage('build docker') {
            sh "cp -R src/main/docker target/"
            sh "cp target/*.war target/docker/"
            dockerImage = docker.build("exba2000/cours-devops-e-commerce:${dockerTag}", 'target/docker')
        }
    }

    stage('publish docker') {
        docker.withRegistry('', 'credential') {
            dockerImage.push dockerTag
            sh "docker rmi exba2000/cours-devops-e-commerce:${dockerTag}"
        }
    }
}
