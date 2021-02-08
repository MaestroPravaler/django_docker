## Criação de um projeto Django com Docker
![](assets/images/django-docker.png)
O primeiro passo é verificar a instalação do docker e do docker-compose
```
docker --version

docker-compose --version
```
Se tudo estiver correto continue este tutorial, caso contrário instale o docker e o docker compose para o seu sistema operacional.

1. Crie um diretório onde deseja instalar a aplicação
2. Crie um ambiente virtual
```
python3 -m venv venv
```
3. Ative o ambiente virtual
```
source venv/bin/activate
```
4. Instale o Django
```
pip install django
```
5. Crie o projeto django
```
django-admin startproject "nomedoprojeto" . #(se der o espaco com o . depois o django não criará o diretório duplicado - é uma escolha pessoal)
```
6. Crie um arquivo na raiz com o nome requirements.txt
```
Django==3.1.6
psycopg2-binary==2.8.6 # necessário para o acesso do postgre
```
7. Crie na raiz do projeto o arquivo Dockerfile (não tem extensão)
```
FROM python:3.8 #Imagem base de nossa construção

ENV PYTHONDONTWRITEBYTECODE 1 # Python não gerar os arquivos .pyc
ENV PYTHONUNBUFFERED 1 # Mensagens de log não ficar armazenadas em buffer - entregas imediatas

WORKDIR /code # Diretório de trabalho

COPY requirements.txt . # Copiar este arquivo para a pasta code (ponto depois da instrução)
RUN pip install -r requirements.txt # Instalar meus requerimentos

COPY . . # Copia tudo da pasta do projeto (primeiro ponto) enviado para pasta code (segundo ponto)
```
8. Criar na raiz o arquivo docker-compose.yml
```
version: "3.8" # Versão do compose

services: # Definição dos serviços
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code # Sincroniza nossos arquivos locais com o Docker
    ports:
      - 8000:8000 # Expose das portas 8000 é a padrão do django
    depends_on: # Meu serviço Web depende do meu serviço db
      - db
  db:
    image: postgres:13
    environment: 
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data/

volumes: # Os dados do banco vão persistir (não há perda de dados)
  postgres_data:
```
9. Configurar o settings.py para o banco postgre
```
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "postgres",
        "USER": "postgres",
        "PASSWORD": "postgres",
        "HOST": "db", # Configurado no Docker-compose
        "PORT": 5432,
    }
}
```
10. Subir o projeto
```
docker-compose up
```
11. Caso tenha realizado alguma modificação no projeto
```
docker-compose up --build
```
12. Caso deseja subir o projeto com o terminal livre para digitar comandos
```
docker-compose up -d
```
13. Continuar visualizando os logs
```
docker-compose logs
```
14. Utilização dos comandos do django docker-compose exec web "comando"
```
# migrações
docker-compose exec web python3 manage.py migrate

#criando super usuários
docker-compose exec web python3 manage.py createsuperuser
```
# Conclusão
O Projeto acima é 100% funcional e fácil de ser replicado em outros contextos.
* Copie para um diretório os arquivos Dockerfile; docker-compose.yml e requirements.txt
* Crie um projeto django como de costume (incluindo a venv)
* Não esqueça do database para postgre
* Suba seu projeto no container e trabalhe normalmente.