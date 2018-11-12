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
    image: dtzar/helm-kubectl:2.10.0
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

node{
   
    stage('Git Clone From Github'){
        git branch: 'master', url: 'https://github.com/z8772083/jenkins_cicd_docker.git'
    }
   
    stage('Maven Build'){
        container('maven'){
        sh """
        mvn -Dskiptest clean package"
        mkdir -p /data/workspace/${JOB_NAME}/
        rm -f /data/workspace/${JOB_NAME}/*.jar
        cp ${WORKSPACE}/target/*.jar /data/workspace/${JOB_NAME}/
        """
    }
}
    stage('Build and Push image'){
        container('docker'){
withDockerRegistry(credentialsId: 'harbor_hub', url: 'harbor.tankme.top') {
        sh """
        docker build -t ${repo}/dev/${JOB_NAME}:${BUILD_NUMBER} .
        docker login ${repo} -u admin -p ${Docker_hub}
        docker push ${repo}/dev/${JOB_NAME}:${BUILD_NUMBER}
        """
        }
 
    }
  } 
    stage('Deploy'){
    sh """
    helm --host upgrade --install --wait  --set image.repository=${repo}/dev/${JOB_NAME},image.tag=${BUILD_NUMBER} nihao2 nihao2
    """
    }
           
}
