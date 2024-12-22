# GerenciamentoTarefasAWS
Sistema de Gerenciamento de Tarefas e Colaboradores na AWS 🚀
Este repositório apresenta uma aplicação simples e funcional utilizando serviços da AWS. A solução é voltada para o gerenciamento de tarefas e colaboradores, com armazenamento dinâmico, envio de notificações e escalabilidade automatizada. A aplicação utiliza os serviços EC2, S3, RDS e Lambda, e está configurada para ser escalável, elástica e com custo otimizado pelo modelo pay-as-you-go.


## 🌟 Funcionalidades Principais

- **Gerenciamento de Tarefas**: Criação, atualização e visualização de tarefas atribuídas a colaboradores.
- **Gestão de Colaboradores**: Cadastro e consulta de informações dos colaboradores.
- **Escalabilidade**: Aumento ou diminuição automática dos recursos conforme a demanda.
- **Elasticidade**: Ajuste dinâmico dos recursos computacionais para manter a performance.
- **Pay-As-You-Go**: Pague somente pelos recursos que utilizar, otimizando os custos.


---

## 📋 Pré-requisitos

1. **Conta AWS**: Crie ou use uma conta existente.
2. **AWS CLI Configurado**: Configure as credenciais utilizando o comando:
   ```bash
   aws configure
   ```
3. **Python 3.8+**: Instale o Python para executar scripts locais.

---
## 1️⃣ Configurando a Instância EC2

### Passos no Console AWS

1. Acesse o serviço **EC2** e clique em "Launch Instance".
2. Configure:
   - **AMI**: Escolha "Amazon Linux 2".
   - **Tipo de Instância**: `t2.micro` (elegível para o nível gratuito).
   - **Armazenamento**: 8GB (padrão).
3. Crie um **Security Group**:
   - Libere o tráfego nas portas `22` (SSH) e `5000` (aplicação).

     ### Acessando a Instância

1. Faça login na instância via SSH:
   ```bash
   ssh -i "seu-arquivo.pem" ec2-user@seu-endereco-ip
   ```
2. Instale as dependências necessárias:
   ```bash
   sudo yum update -y
   sudo yum install python3 git -y
   pip3 install flask boto3 pymysql
   ```

---

## 2️⃣ Configurando o Banco de Dados RDS

### Passos no Console AWS

1. Vá para **RDS** e clique em "Create Database".
2. Configure:
   - Tipo: MySQL ou PostgreSQL.
   - Modo: "Free Tier".
   - Armazene o **nome do banco**, **usuário**, **senha** e o **endpoint**.
3. Configure o **Security Group**:
   - Libere a porta `3306` (MySQL) ou `5432` (PostgreSQL).
   - Permita o acesso à instância EC2.

### Testando a Conexão com o Banco

Crie um script para testar a conexão:
```python
import pymysql

# Configurações do Banco de Dados
db_host = "SEU_RDS_ENDPOINT"
db_user = "SEU_USUARIO"
db_password = "SUA_SENHA"
db_name = "SEU_BANCO"

# Conectar ao Banco
connection = pymysql.connect(
    host=db_host,
    user=db_user,
    password=db_password,
    database=db_name
)

print("Conexão bem-sucedida!")
```

---

## 3️⃣ Configurando o Bucket S3

### Passos no Console AWS

1. Acesse o serviço **S3** e clique em "Create Bucket".
2. Configure:
   - Nome do bucket: `gerenciamento-tarefas`.
   - Região: Mesma da EC2 e RDS.
3. Ative o acesso público, se necessário, para armazenar arquivos acessíveis.

### Upload de Arquivos no S3

Use o script abaixo para enviar arquivos ao bucket:
```python
import boto3

# Configurações do S3
s3 = boto3.client('s3')
bucket_name = "gerenciamento-tarefas"

def upload_file_to_s3(file_name, object_name):
    s3.upload_file(file_name, bucket_name, object_name)
    print(f"Arquivo {file_name} enviado para {bucket_name}/{object_name}")

# Exemplo de uso
upload_file_to_s3("relatorio.txt", "relatorios/relatorio.txt")
```

---

## 4️⃣ Criando a Função Lambda

### Passos no Console AWS

1. Acesse o serviço **Lambda** e clique em "Create Function".
2. Configure:
   - Nome: `NotificacaoTarefas`.
   - Permissões: Conceda acesso ao S3 e SNS.
3. Adicione o seguinte código:
   ```python
   import boto3

   def lambda_handler(event, context):
       sns = boto3.client('sns')
       topic_arn = "arn:aws:sns:REGIAO:ID_DO_USUARIO:NomeDoTopico"
       mensagem = "Uma nova tarefa foi criada!"

       sns.publish(TopicArn=topic_arn, Message=mensagem)
       return {"statusCode": 200, "body": "Notificação enviada com sucesso!"}
   ```
4. Configure o **trigger** para eventos relevantes no bucket S3.

---

## 5️⃣ Backend da Aplicação (Flask)

Crie o arquivo `app.py` no servidor EC2:

### Passo a Passo

1. **Acesse sua Instância EC2 via SSH**:
   - No terminal, conecte-se à sua instância EC2.
   ```bash
   ssh -i "seu-arquivo.pem" ec2-user@seu-endereco-ip
   ```

2. **Instale as Dependências Necessárias**:
   - Atualize os pacotes e instale Python, Git, Flask, Boto3 e PyMySQL.
   ```bash
   sudo yum update -y
   sudo yum install python3 git -y
   pip3 install flask boto3 pymysql
   ```

3. **Clone o Repositório do Projeto**:
   - Clone o repositório do GitHub (substitua pelo seu URL do GitHub).
   ```bash
   git clone https://github.com/seuusuario/Projeto-Gerenciamento-Tarefas.git
   cd Projeto-Gerenciamento-Tarefas
   ```

4. **Crie o Arquivo `app.py`**:
   - Dentro do diretório do projeto, crie o arquivo `app.py`.
   ```bash
   touch app.py
   ```
   - Edite o arquivo `app.py` e adicione o seguinte conteúdo:
   ```python
   from flask import Flask, jsonify, request
   import boto3
   import pymysql

   app = Flask(__name__)

   # Configurações do Banco de Dados
   db_host = "SEU_RDS_ENDPOINT"
   db_user = "SEU_USUARIO"
   db_password = "SUA_SENHA"
   db_name = "SEU_BANCO"

   # Configurações do S3
   s3 = boto3.client('s3')
   bucket_name = "gerenciamento-tarefas"

   @app.route('/tarefas', methods=['GET'])
   def listar_tarefas():
       connection = pymysql.connect(
           host=db_host, user=db_user, password=db_password, database=db_name
       )
       cursor = connection.cursor()
       cursor.execute("SELECT * FROM tarefas")
       dados = cursor.fetchall()
       return jsonify(dados)

   @app.route('/tarefas', methods=['POST'])
   def criar_tarefa():
       dados = request.json
       titulo = dados['titulo']
       descricao = dados['descricao']
       connection = pymysql.connect(
           host=db_host, user=db_user, password=db_password, database=db_name
       )
       cursor = connection.cursor()
       cursor.execute("INSERT INTO tarefas (titulo, descricao) VALUES (%s, %s)", (titulo, descricao))
       connection.commit()
       return jsonify({"message": "Tarefa criada com sucesso!"})

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```

5. **Execute o Servidor Flask**:
   - Inicie o servidor Flask para verificar se tudo está funcionando.
   ```bash
   python3 app.py
   ```
   - O servidor estará acessível no endereço público da sua EC2 na porta 5000.

---
### Passo a Passo

1. **Crie a Página HTML para a Interface Web**:
   - Crie um arquivo `index.html` com o seguinte conteúdo:
   ```html
   <!DOCTYPE html>
   <html lang="pt-BR">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Gerenciamento de Tarefas</title>
       <style>
           body {
               font-family: Arial, sans-serif;
               background-color: #f4f4f4;
               margin: 0;
               padding: 0;
               display: flex;
               flex

   ---

## 📜 Como Contribuir

Obrigado por se interessar em contribuir para o **Sistema de Gerenciamento de Tarefas e Colaboradores na AWS**! Abaixo estão os passos para acessar, clonar e modificar o repositório para melhorias:

1. **Clone o Repositório**:
   - Primeiramente, faça um fork do repositório para sua conta no GitHub.
   - Em seguida, clone o repositório para seu ambiente local:
     ```bash
     git clone https://github.com/seuusuario/Projeto-Gerenciamento-Tarefas.git
     cd Projeto-Gerenciamento-Tarefas
     ```

2. **Crie uma Nova Branch**:
   - Para adicionar novas funcionalidades ou corrigir bugs, crie uma nova branch:
     ```bash
     git checkout -b minha-nova-feature
     ```

3. **Faça as Modificações Necessárias**:
   - Adicione suas melhorias ou correções ao projeto.
   - Depois de fazer as alterações, adicione os arquivos modificados:
     ```bash
     git add .
     ```

4. **Faça o Commit das Alterações**:
   - Faça o commit das suas alterações com uma mensagem descritiva:
     ```bash
     git commit -m "Descrição clara das melhorias"
     ```

5. **Envie suas Alterações para seu Fork**:
   - Envie suas alterações para o seu repositório no GitHub:
     ```bash
     git push origin minha-nova-feature
     ```

6. **Crie um Pull Request**:
   - Acesse o seu repositório no GitHub e clique em "Compare & pull request".
   - Envie o pull request para revisão.

Apreciamos todas as contribuições, sejam elas grandes ou pequenas. Se tiver alguma dúvida, sinta-se à vontade para abrir uma issue. Vamos trabalhar juntos para tornar este projeto ainda melhor!

---

Desenvolvido com 💡 e AWS!

---







