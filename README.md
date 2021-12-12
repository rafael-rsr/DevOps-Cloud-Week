# **DevOps Cloud Week**

## **AULA 1**
Primeiramente criamos um repositorio no gitlab e usamos o Git para fazer o versionamento dos codigos.
Comandos executados:
- Comando para configurar o git com a conta do gitlab:
 
    git config --global user.name "username"
    
    git config --global user.email "email"
    
- Comando para clonar o repositorio do gitlab no meu PC loc:

    git clone "link do repositório"

Após o Git e a conta configurada, coloquei os dados da aplicação dentro do diretorio no meu PC local e usei os seguintes comandos para commit:

    git add . (para adicionar todos os arquivos)
    
    git commit -m "Envio de dados da aplicação" (para comentar)
    
    git push (para enviar ao repositorio remoto)
    
    git status (para verificar se estava tudo ok)
    
### **AWS**
Na AWS primeiramente criei uma instancia Amazon Linux, para o Jenkis e uma outra instancia linux ubuntu, para ser nosso servidor de aplicação (app-server)

- Instancia Jenkis
    
Inciei uma instancia com um script pronto para instalação do Jenkis e algumas dependencias.

     #!/bin/bash
    sudo yum update -y
    sudo amazon-linux-extras install epel -y
    sudo yum install daemonize -y
    sudo wget -O /etc/yum.repos.d/jenkins.repo \
        https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    sudo yum upgrade
    sudo yum install jenkins java-1.8.0-openjdk-devel -y
    sudo systemctl daemon-reload
    sudo systemctl start jenkins
   
   

- Instancia app-server
    
Inciei uma instancia com um script pronto para instalação do docker e suas dependencias.

    #!/bin/bash
    sudo apt-get update
    sudo apt-get upgrade -y
    sudo apt-get install docker.io git -y
    sudo usermod -aG docker ubuntu
     
### **Dificuldades:** 
Os comandos do linux ainda não são automaticos. Algumas coisas preciso pesquisar e entender o porque usei. Mas com a ajuda da comunidade cloud e linux estou me adequando cada dia mais.

### **Observações:** 
Essa foi minha primeira experiencia com a ferramenta GIT e Jenkins. Tudo que tinha visto até essa semana era a parte teórica. Colacando em prática ficou muito mais fácil o entendimento.

## **AULA 2**

Iniciei a instancia de nome "app-server" na AWS e clonei meu repositorio remoto.

      git clone "link do repositório"

Executamos docker file com as intruçõespré definidas.

 - dockerfile
 
      FROM node:14 (para utilizar a imagem do Node do docker hub)
      
      WORKDIR /usr/src/app (para trabalhar na pasta especificada)
      
      COPY . . 
      
      RUN npm install (instalar dependencias)
      
      EXPOSE 3000 (aplicação usa a porta 3000)
      
      CMD [ "npm", "start" ] 
  
  
 - Comandos executados
 
      docker build -t "nome da imagem" (criar sua propria imagem baseado em instruções no dockerfile)
      
      docker run -itd -p 80:3000 "nome da imagem" ( Irá rodar o container com a imagem previamente criada e expondo a aplicação na porta 80)
      
      docker ps (para verificar se o container esta rodando)
   
### Pronto! A aplicação agora esta no ar e rodando em um container

### **Observações:** 
Essa foi minha primeira experiencia com a ferramenta docker. Tudo que tinha visto até essa semana era a parte teórica. Colacando em prática ficou muito mais fácil o entendimento.


## **AULA 3**

Conectei na instancia Jenkis para instalar o Git e o Docker. Comandos executados:

       sudo yum update -y (Atualiza os softwares disponiveis para o sistema)

       sudo yum install git -y (Instala o git)

       sudo amazon-linux-extras install docker -y (Instala o docker)

       sudo systemctl start docker (Inicia o docker)


Conectei na instancia  "app-server" para fazer o login no docker hub e enviar a imagem para o repositorio de imagens remoto.

- Comandos utilizados

      docker login( Efetua o login no Docker Hub)

      docker tag devops-cw rafaelrsr0505/devops-cw (Cria um "nome simbolico" da imagem)

      docker push rafaelrsr0505/devops-cw (para enviar a imagem ao docker hub)

Agora ja temos uma imagem do Sistema no Docker Hub.

Nesse momento foi necessário abrir o configurador do Jenkins para criar um Job. Porém, precisamos criar uma credencial do docker hub dentro do jenkins e baixar o plugin do docker pipeline. Isso para que nosso script do pipeline rode certinho.

-Criação de Job Pipeline com secript para clonar o repositorio do GitLab, buildar a imagem do docker, e eviar para o repositorio remoto. Segue script:

       stages {
           stage('Clone Repository') {
            steps {  
                      git branch: "main", url: 'REPO_URL'    # Alterar GIT REPO HTTP URL 
         }
           }
           stage('Build Docker Image') {
                  steps{
                      script {
                          dockerImage = docker.build registry + ":$BUILD_NUMBER"
                      }
                  }
              }
           stage('Send image to Docker Hub') {
                  steps{
                      script {
                          docker.withRegistry( '', registryCredential) {
                              dockerImage.push()
                          }
                      }
                  }
              }
           stage('Cleaning up') {
               steps {
                   sh "docker rmi $registry:$BUILD_NUMBER"
               }
        }
          }
      }



Rodamos o Job e nesse momento automatizei o processo de build pelo Jenkins. 

### CodeDeploy Aula 3.1

- Conectei no painel AWS para criar funções dentro do IAM.

- Criei uma função de CodeDeploy e uma outra para EC2 de full access.

- Fui na instancia app-server e mudei a função do IAM para função criada para EC2.

- Conectei na instancia do Jenkins para rodar as dependencias do ColdDeploy

   sudo apt install ruby-full -y (Instala o Ruby)
   
   wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install (Faz o download do agente do codedeploy)
   
   chmod +x ./install  (Da permissão de execução para o arquivo)
   
   sudo ./install auto > /tmp/logfile  (executa a instalação do agente)
   
   sudo service codedeploy-agent status (Verifica se o agente foi iniciado corretamente)
   
   
   
- No painel da AWS criei um Bucket S3 para armazenamento de objetos para upar minha aplicação compactada (.zip).

- No painel da AWS entrei na função CodeDeploy > Aplicativos.

- Criei um aplicativo para plataforma de computação EC2.

- Criei um grupo de implantação com a função de serviço criada anteriormente e congiguração de instancia EC2 com a tag name criada para "app-server"


Agora precisei criar os scripts para o deploy. Os arquivos são "appspec.yml" , "start-container.sh", "stop-container.sh"
 
 APPSPEC.YML
 
        version: 0.0
    os: linux
    hooks:
      AfterInstall:
        - location: Scripts/stop-container.sh
      ApplicationStart:
        - location: Scripts/start-container.sh


START-CONTAINER.SH

       #!/bin/bash

      docker run -itd -p 80:3000 rafael0505/devops-cw:develop



STOP-CONTAINER.SH

      #!/bin/bash

     docker rm -f $(docker ps -qa) || true
     
     docker rmi rafael0505/devops-cw:develop || true
     

- Após os arquivos criados e devidamentes inseridos no diretorio da aplicação, fiz o commit para meu repositorio remoto do GitLab.

- Entrei no painel do Jenkins para alterar duas linhas do comando na pipeline, tendo em vista que agora demos um nome para a tag da versão criada (develop). Então ficou assim:

   dockerImage = docker.build registry + ":develop"
   
   sh "docker rmi $registry:develop"
 
- Rodei o JOB novamente para o DockerHub reconhecer essa nova tag "develop".

- Voltamos no CodeDeploy da AWS para criar uma nova implantação direcionando o local da revisão para a uri do bucket s3 criado anteriormente.

- Nesse momento ele ja vai realizar o deploy com a nova tag "develop" tbm.

No final dessa aula, criamos o deploy e preparamos o Jenkis para realizar a configuração da atuomatização do CodeDeploy. 

Agora nosso laboratorio esta assim: Qualquer alteração feita na aplicação pelo desenvolvedor, o DevOps teria que entrar no Jenkis, iniciar um novo Job e depois iniciar um processo de implantação do CodeDeploy na plataforma da AWS.

### **Observações:** 
Nessa aula usei na pratica pela primeira vez o ColdDeploy, e ver ele funcionando, mesmo não automatizado ainda, foi muito bacana. São ferramentas que impressionam qualquer profissional de T.I que esta buscando novos conhecimentos.

## **AULA 4**


### AUTOMATIZANDO DEPLOY

- No painel do Jenkins e instalamos o plugin AWS CodeDeploy.

- No painel da AWS, entrei em IAM e criei um novo usuario com acesso programatico para que o Jenkins se autentique no CodeDeploy.

- Na pipeline do Jenkins adicionamos uma etapa de deploy. Ficou assim:




pipeline {
    agent any

    environment {
		registry = "DOCKER_USER/DOCKER_HUB_REPO"    # Alterar
        registryCredential = "dockerhub_id"
        dockerImage = ''
    }

    stages {
    	stage('Clone Repository') {
    		steps {  
                git branch: "main", url: 'GIT_URL'   # Alterar
			}
    	}
    	stage('Build Docker Image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":develop"
                }
            }
        }
    	stage('Send image to Docker Hub') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    	stage('Deploy') {
		    steps{
                step([$class: 'AWSCodeDeployPublisher',
                    applicationName: 'app01-application',                          # Alterar
                    awsAccessKey: "AKIAZTOKAUMQYNQ37WP5",                          # Alterar
                    awsSecretKey: "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",      # Alterar
                    credentials: 'awsAccessKey',
                    deploymentGroupAppspec: false,
                    deploymentGroupName: 'service',                                # Alterar
                  TODO O PROCESSO  deploymentMethod: 'deploy',
                    excludes: '',
                    iamRoleArn: '',
                    includes: '**',
                    pollingFreqSec: 15,
                    pollingTimeoutSec: 600,
                    proxyHost: '',
                    proxyPort: 0,
                    region: 'us-east-2',
                    s3bucket: 'devopscloudweek2-app01',                            # Alterar
                    s3prefix: '', 
                    subdirectory: '',
                    versionFileName: '',
                    waitForCompletion: true])
            }
        }
    	stage('Cleaning up') {
        	steps {
            	sh "docker rmi $registry:develop"
        	}
		}
    }
}







### AUTOMATIZANDO TODO PROCESSO APÓS O COMMIT DO CODIGO.

- Instalei o plugin do GitLab no painel do Jenkins.

- Configurei integração do GitLab com o Jenkins.

- Criei um SecretToken do job do Jenkins para o GitLab

- Configurei esse sercret token e URL dentro de Webhooks no GitLab

Agora o dev faz qualquer alteração no código, faz o commit para o repositorio remoto e o processo de clonar, de build do imagem docker, do envio da imagem para o docker hub e o deploy dessa imagem são realizados automaticamente pela pipeline do Jenkins.

### PRONTO AGORA TODO O PROCESSO ESTA AUTOMATIZADO.


### **Dificuldades:** 
Nessa quarta aula tive dificuldades pois usamos uma instancia free tier (t2.micro) para realizar o laboratorio. Essa instancia tinha apenas 1 CPU e 1GB de memoria. Já no primeiro envio, durante a automatização a instancia travou sendo necessário interromper ela no painel da AWS e iniciar novamente.

Após isso, minha pipeline parou de funcionar. Não conectava mais com o docker. Sendo assim, comecei a revisar todos os processos, codigos, keys e usuaurios. E não conseguia achar o que de fato eu tinha feito de errado. Enfim, depois de alguns minutos revendo tudo e nao achando nada, lembrei que após reinciar a instancia do Jenkins eu não iniciei o serviço do docker no linux. Fiz o pocedimento (sudo systemctl start docker) para iniciar o docker e tudo voltou ao normal.

### **Conclusão:** 

Foi uma imersão de treinamentos que duraram 4 dias. Realizei laboratorios com serviços e ferramentas open source, onde até na AWS usei instancias free tier para realizar todos os testes. Aprendi muito vendo todas as ferramentas na prática. Saio desse projeto agregando muito conhecimento e vontade de estar nessa área em breve. Agora é realizar a especialização DevOps e continuar na busca de uma oportunidade para entrar de vez nesse mundo. A decisão já esta tomada, quero ser um DevOps.











   

