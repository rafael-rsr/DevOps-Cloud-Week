# **DevOps Cloud Week**

## **AULA 1**
Primeiramente criamos um repositorio no github e usamos o Git para fazer o versionamento dos codigos.
Comandos executados:
- Comando para configurar o git com a conta do github:
 
    git config --global user.name "username"
    
    git config --global user.email "email"
    
- Comando para clonar o repositorio do github no meu PC loc:

    git clone "link do repositório"

Após o Git configurado e o a conta configurada, coloquei os dados da automação dentro do diretorio no meu PC local e usei os seguintes comandos para commit:

    git add . (para adicionar todos os arquivos)
    
    git commit -m "Envio de dados da automação" (para comentar)
    
    git push (para enviar ao repositorio remoto)
    
    git status (para verificar se estava tudo ok)
    
### **AWS**
Na AWS primeiramente criei uma instancia  Amazon Linux, para o Jenkis e outra instancias linux ubuntu para ser nosso servidor de aplicação (app-server)

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
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    

- Instancia app-server
    
Inciei uma instancia com um script pronto para instalação do docker e suas dependencias.

    #!/bin/bash
    sudo apt-get update
    sudo apt-get upgrade -y
    sudo apt-get install docker.io git -y
    sudo usermod -aG docker ubuntu
     
### **Dificuldades:** 
Os comandos do linux ainda não são automaticos. Muita coisa preciso pesquisar e entender o porque usei. Mas com a ajuda da comunidade cloud e linux estou me adequando cada dia mais.

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

Nesse momento foi necessário abrir o configurador do Jenkins para criar um Job. Porém, precisamos criar uma credencial do docker hub dentro do jenkins e baixar o plugin do docker pipelibe. Isso para que nosso script do pipeline rodar certinho.

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


