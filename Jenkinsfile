node{
   
    stage('Git Clone From Github'){
        git branch: 'master', url: 'https://github.com/z8772083/helm_spring_boot_demo.git'
    }
   
    stage('Maven Build'){
        def MvnHome = tool name: 'maven', type: 'maven'
        def MvnCmd = "${MvnHome}bin/mvn"
        sh "${MvnCmd} -Dskiptest clean package"
    }

    stage('Build and Push image'){
        sh """
        docker build -t ${repo}/dev/${JOB_NAME}:${BUILD_NUMBER} .
        docker login ${repo} -u admin -p ${Docker_hub}
        docker push ${repo}/dev/${JOB_NAME}:${BUILD_NUMBER}
        """
    }
   
    stage('Deploy'){
    sh """
    helm --host ${helm_host} upgrade --install --wait  --set image.repository=${repo}/dev/${JOB_NAME},image.tag=${BUILD_NUMBER} nihao nihao
    """
    }
           
}
