# Etapes Jenkins : Les types de Jobs 

Notes prises lors de la phase Jenkins

## Création du fichier config et structure des répertoires

```bash
mkdir ~/.kube
sudo k3s kubectl config view --raw >> ~/.kube/config
cp -p ~/.kube/config ~/datascientest-jenkins/.kube/

mkdir datascientest-jenkins
cd datascientest-jenkins
mkdir app
touch app/main.py
helm create fastapi
touch Dockerfile
touch requirements.txt
```

## Adaptation du Jenkinsfile
`Docker Build`

```bash
stages {
  stage(' Docker Build'){ // docker build image stage
    steps {
      script {
      sh '''
        docker rm -f jenkins || true
        docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
      sleep 6
      '''
      }
    }
  }
```

`Docker Push`

```bash
  stage('Docker Push'){ //we pass the built image to our docker hub account

    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text ca
lled docker_hub_pass saved on jenkins
    }
    steps {
      script {
        retry(3) {
          sh '''
          echo $DOCKER_PASS | docker login -u $DOCKER_ID --password-stdin
          docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
          '''
        }
      }
    }
  }
```
`Déploiement dev, homo et prod`

```bash
  stage('Deploiement en dev'){
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved
 on jenkins
    }
    steps {
      script {
      sh '''
      export KUBECONFIG=/home/vagrant/.kube/config
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app fastapi --values=values.yml --namespace dev
      '''
      }
    }
  }
```

## Envoi des fichiers vers le svc github

Tous les fichiers du projet, on était envoyé dans le dépôt  [github][github].

### Ajouter les users jenkins et docker dans les groupes

```bash
sudo usermod -aG vagrant jenkins
sudo usermod -aG docker jenkins
sudo usermod -aG vagrant docker
```

### Problèmes de droits sur ~/.kube

```bash
chmod 775 -R ~/.kube/
```

[github]: https://github.com/Mossice/Jenkins-datascientest

---
