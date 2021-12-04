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
