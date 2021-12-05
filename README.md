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
Essa foi minha primeira experiencia com a ferramenta GIT e Jenkins. Tudo que tinha visto até essa semana era a parte teórica. Colacando em prática ficou muito mais fácil o entedimento.

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
   
###Pronto! A aplicação agora esta no ar e rodando em um container
 
 

