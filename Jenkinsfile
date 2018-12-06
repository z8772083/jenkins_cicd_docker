def label = "maven-docker-${UUID.randomUUID().toString()}"

podTemplate(label: label, yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.3.9-jdk-8-alpine
    command: ['cat']
    tty: true
    volumeMounts:
    - name: mvn-storage
      mountPath: /root/.m2/repository
      readOnly: false
    - name: workspace-pv-storage
      mountPath: /data/workspace
      readOnly: false
  - name: docker
    image: docker:1.13
    command: ['cat']
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
    - name: workspace-pv-storage
      mountPath: /data/workspace
      readOnly: false
  - name: helm
    image: z8772083/helm:v2.12.0-rc.1
    command: ['cat']
    tty: true
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock    
  - name: mvn-storage
    persistentVolumeClaim:
      claimName: jenkins-maven
  - name: workspace-pv-storage
    persistentVolumeClaim:
      claimName: jenkins-pvc
"""
  ) {

node(label){
   
    stage('Git Clone From Github'){
        git branch: 'master', url: 'https://github.com/z8772083/jenkins_cicd_docker.git'
    }
   
    stage('Maven Build'){
        container('maven'){
        sh """
        mvn -Dskiptest clean package
        mkdir -p /data/workspace/${JOB_NAME}/
        rm -f /data/workspace/${JOB_NAME}/*.jar
        cp ${WORKSPACE}/target/*.jar /data/workspace/${JOB_NAME}/
        """
    }
}
    stage('Build and Push image'){
        container('docker'){
        withDockerRegistry(credentialsId: '9f63989b-b75d-4943-be02-93eba548ce9b', url: 'harbor.k8sops.com') {
        sh """
        docker build -t ${repo}/ops/${JOB_NAME}:${BUILD_NUMBER} .
         docker push ${repo}/ops/${JOB_NAME}:${BUILD_NUMBER}
        """
       }
    }
  } 
    stage('Deploy'){
        container('helm'){
    sh """
    helm  upgrade --install --wait  --set image.repository=${repo}/dev/${JOB_NAME},image.tag=${BUILD_NUMBER} nihao2 nihao2
    """
    }
    }
           
}
}
