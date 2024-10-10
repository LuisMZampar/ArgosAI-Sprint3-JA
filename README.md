# ArgosAI-CheckPoint

Para a matéria de DevOps
Link do Youtube: (https://youtu.be/9kuN30muiZc?si=NITEKfyvPiqp-ov9)

## 🌍 ArgosAI Sprint 3 - API
Este projeto é uma API desenvolvida em Java usando Spring Boot, com integração ao banco de dados SQL no Azure. Ele oferece um CRUD completo para produtos, clientes e vendas. Este documento fornece instruções detalhadas sobre como realizar o deploy e testar a API.

## ⚙️ Pré-requisitos
Antes de iniciar o deploy, certifique-se de ter as seguintes ferramentas instaladas:

Java 17 ou superior
Maven para gerenciamento de dependências
Azure CLI configurado
Acesso a uma conta no Azure

## 📥 Clonar o Repositório
Primeiro, faça o clone do repositório:

git clone https://github.com/LuisMZampar/ArgosAI-Sprint3-JA.git
cd ArgosAI-Sprint3-JA

## 🏗️ Compilar e Empacotar a Aplicação
Utilize o Maven para compilar e empacotar a aplicação:

mvn clean package -DskipTests

## 🗄️ Criação do Banco de Dados no Azure SQL
Execute os seguintes comandos para criar o banco de dados e o servidor SQL no Azure:

Criar o grupo de recursos
az group create --name rg-bcosql --location brazilsouth

Criar o servidor SQL
az sql server create --name sqlserver-rm550531 --resource-group rg-bcosql --location brazilsouth --admin-user admlnx --admin-password Fiap@2tdsvms

Configurar regra de firewall para liberar acesso
az sql server firewall-rule create --resource-group rg-bcosql --server sqlserver-rm550531 --name AllowAllIPs --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

Criar o banco de dados
az sql db create --resource-group rg-bcosql --server sqlserver-rm550531 --name argos --service-objective Basic --backup-storage-redundancy Local --zone-redundant false


## ☁️ Deploy no Azure App Service
Para fazer o deploy da aplicação no Azure App Service, utilize o seguinte comando:

az webapp deploy --resource-group rg-argos --name argos-rm550531 --src-path "target/ArgosAI-Sprint3-0.0.1-SNAPSHOT.jar"


## 📦 Publicar Imagem no Azure Container Registry (ACR)
Realize as etapas para criar e enviar a imagem Docker para o ACR:

## Criar o ACR
az acr create --resource-group rg-bcosql --name rm550531acr --sku Basic --location brazilsouth

## Login no ACR
az acr login --name rm550531acr --expose-token

## Construir a imagem Docker
docker build -t rm550531/argos-app:1.0 .

## Tag da imagem para o ACR
docker tag rm550531/argos-app:1.0 rm550531acr.azurecr.io/argos-app:1.0

## Push da imagem para o ACR
docker push rm550531acr.azurecr.io/argos-app:1.0

## Atualizar ACR para habilitar login de administrador
az acr update -n rm550531acr --admin-enabled true

## Obter as credenciais do ACR
az acr credential show --name rm550531acr


## 📊 Monitoramento de Logs
Acompanhe os logs em tempo real com o seguinte comando:

az webapp log tail --resource-group rg-argos --name argos-rm550531


# Visão Geral
O projeto ArgosAI-Sprint3 é desenvolvido para gerenciar clientes e produtos em um sistema de recomendação. Ele faz uso de APIs RESTful, documentação com Swagger, e a arquitetura segue padrões como HATEOAS (Hypermedia as the Engine of Application State) para uma navegação mais dinâmica entre recursos.

## Tecnologias Utilizadas
Java 17: Linguagem principal do projeto.
Spring Boot: Framework para simplificar o desenvolvimento de APIs RESTful.
Maven: Ferramenta de gerenciamento de dependências e build.
Oracle SQL: Banco de dados relacional para persistência dos dados.
Hibernate: Framework de ORM para gerenciar entidades JPA.
Swagger/OpenAPI: Documentação automática da API.
ModelMapper: Conversão entre entidades e DTOs.
Lombok: Redução de código boilerplate.
Thymeleaf: Renderização de páginas HTML.
HATEOAS: Navegação entre recursos REST.
Fly.io: Plataforma para deployment da aplicação.
Arquitetura e Design Pattern
A arquitetura segue o padrão Layered Architecture (Camadas), separando responsabilidades em Controller, Service e Repository. O projeto utiliza o pattern DTO para transferir dados entre a API e serviços, e o ModelMapper facilita a conversão entre modelos e DTOs.

## Endpoints
Clientes
GET /api/clientes: Lista todos os clientes.
GET /api/clientes/{id}: Obtém um cliente por ID.
POST /api/clientes: Cria um novo cliente.
PUT /api/clientes/{id}: Atualiza um cliente.
DELETE /api/clientes/{id}: Exclui um cliente.
Produtos
GET /api/produtos: Lista todos os produtos.
GET /api/produtos/{id}: Obtém um produto por ID.
POST /api/produtos: Cria um novo produto.
PUT /api/produtos/{id}: Atualiza um produto.
DELETE /api/produtos/{id}: Exclui um produto.
Dependências
As principais dependências do projeto incluem:

Spring Boot Starter Web e Data JPA
ModelMapper e Lombok
Swagger/OpenAPI para documentação
Oracle JDBC Driver para conexão com banco de dados Oracle
HikariCP para gerenciar a conexão com o banco
Deploy
O deploy é realizado utilizando Fly.io e Docker. Os passos principais incluem o build com Maven e o deploy usando Fly.io, expondo a aplicação na porta 8080 para acesso à API.
