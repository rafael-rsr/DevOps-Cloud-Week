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
    sudo wget https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
    sudo rpm -ivh amazon-ssm-agent.rpm
    sudo systemctl enable amazon-ssm-agent
    
### **Dificuldades:** 
    Os comandos do linux ainda não são automaticos. Muita coisa ainda preciso pesquisar e entender o porque usei. Mas com a ajuda da comunidade cloud e linux estou me adequando cada dia mais.

### **Observações:** 
    Essa foi minha primeira experiencia com a ferramenta GIT e Jenkins. Tudo que tinha visto até essa semana era a aprte teórica. Colacando em prpática ficou muito mais fácil o entedimento.

## **AULA 2**


    
