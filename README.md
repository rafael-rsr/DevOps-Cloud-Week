# **DevOps Cloud Week**

## **AULA 1**
Primeiramente criamos um repositorio no github e usamos o Git para fazer o versionamento dos codigos.
Comandos executados:
- Comando para configurar o git com a conta do github:
 
    git config --global user.name "username"
    
    git config --global user.email "email"
    
- Comando para clonar o repositorio do github no meu PC loc:

    git clone "link do repositório"

Após o Git e a conta configurada, coloquei os dados da automação dentro do diretorio no meu PC local e usei os seguintes comandos para commit:

    git add . (para adicionar todos os arquivos)
    
    git commit -m "Envio de dados da automação" (para comentar)
    
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

### **Dificuldades:** 
Com a experiencia da primeira aula, nao houve muitas dificuldades na aula 2. O entendimento melhor do docker virá com as práticas da ferramenta.

### **Observações:** 
Essa foi minha primeira experiencia com a ferramenta docker. Tudo que tinha visto até essa semana era a parte teórica. Colacando em prática ficou muito mais fácil o entendimento.


## **AULA 3**

Conectei na instancia Jenkis para instalar o Git e o Docker. Comandos executados:

       sudo yum update -y (Atualiza os softwares disponiveis para o sistema)

       sudo yum install git -y (Instala o git)

       sudo amazon-linux-extras install docker -y (Instala o docker)

       sudo systemctl start docker (Inicia o docker)

       sudo usermod -aG docker jenkins (Da permissões ao Jenkins para rodar comandos Docker)

       sudo systemctl restart jenkins (Reinicia o Jenkins para aplicar as permissões anteriormente adicionadas)
 
Conectei na instancia  "app-server" para fazer o login no docker hub e enviar a imagem para o repositorio de imagens remoto.

- Comandos utilizados

      docker login( Efetua o login no Docker Hub)

      docker tag devops-cw rafaelrsr0505/devops-cw (Cria um "nome simbolico" da imagem)

      docker push rafaelrsr0505/devops-cw (para enviar a imagem ao docker hub)

Agora ja temos uma imagem do Sistema no Docker Hub.

Nesse momento foi necessário abrir o configurador do Jenkins para criar um Job. Porém, precisamos criar uma credencial do docker hub dentro do jenkins e baixar o plugin do docker pipeline. Isso para que nosso script do pipeline rode certinho.

-Criação de Job Pipeline com secript para clonar o repositorio do Git Hub, buildar a imagem do docker, e eviar para o repositorio remoto. Segue script:

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
     

- Após os arquivos criados e devidamentes inseridos no diretorio da automação. Fiz o commit para meu repositorio remoto do GitHub.

- Entrei no painel do Jenkins para alterar duas linhas do comando na pipeline, tendo em vista que agora demos um nome para a tag da versão criada (develop). Então ficou assim:

   dockerImage = docker.build registry + ":develop"
   
   sh "docker rmi $registry:develop"
 
- Rodei o JOB novamente para o DockerHub reconhecer essa nova tag "develop".


   

