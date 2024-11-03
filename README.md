# ArgosAI-CheckPoint

Para a matéria de DevOps
Link do Youtube: 

## 🌍 ArgosAI Sprint 3 - API
Este projeto é uma API desenvolvida em Java usando Spring Boot, com integração ao banco de dados SQL no Azure. Ele oferece um CRUD completo para produtos, clientes e vendas. Este documento fornece instruções detalhadas sobre como realizar o deploy e testar a API.

## ⚙️ Pré-requisitos
Antes de iniciar o deploy, certifique-se de ter as seguintes ferramentas instaladas:

Java 17 ou superior
Maven para gerenciamento de dependências
Azure CLI configurado
Acesso a uma conta no Azure


## Arquivo infraACR.sh

grupoRecursos=rg-bcosql

regiao=brazilsouth

nomeServidorSQL=sqlserver-rm550531

adminSQL=admlnx

senhaSQL=Fiap@2tdsvms

nomeBanco=argos

nomeACR=rm550531acr

skuACR=Basic

if [ $(az group exists --name $grupoRecursos) = true ]; then
    echo "O grupo de recursos $grupoRecursos já existe"

else
    az group create --name $grupoRecursos --location $regiao
    echo "Grupo de recursos $grupoRecursos criado na localização $regiao"
    
fi

az sql server create --name $nomeServidorSQL --resource-group $grupoRecursos --location $regiao --admin-user $adminSQL --admin-password $senhaSQL
echo "Servidor SQL $nomeServidorSQL criado com sucesso."

az sql server firewall-rule create --resource-group $grupoRecursos --server $nomeServidorSQL --name AllowAllIPs --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
echo "Regra de firewall para acesso de todos os IPs configurada com sucesso."

az sql db create --resource-group $grupoRecursos --server $nomeServidorSQL --name $nomeBanco --service-objective Basic --backup-storage-redundancy Local --zone-redundant false
echo "Banco de dados $nomeBanco criado com sucesso no servidor $nomeServidorSQL."

if az acr show --name $nomeACR --resource-group $grupoRecursos &> /dev/null; then
    echo "O ACR $nomeACR já existe"
    
else
    az acr create --resource-group $grupoRecursos --name $nomeACR --sku $skuACR --location $regiao
    echo "ACR $nomeACR criado com sucesso"
    
fi

az acr login --name $nomeACR --expose-token
echo "Login no ACR $nomeACR realizado com sucesso."

----------

chmod +x infraACR.sh

./infraACR.sh

----------

## Arquivo infraACR.sh

grupoRecursos=rg-bcosql

nomeACR=rm550531acr

imageACR=rm550531acr.azurecr.io/argos-app:1.0

serverACR=rm550531acr.azurecr.io

userACR=$(az acr credential show --name $nomeACR --query "username" -o tsv)

passACR=$(az acr credential show --name $nomeACR --query "passwords[0].value" -o tsv)

nomeACI=argosrm550531

az container create \

--resource-group $grupoRecursos \

--name $nomeACI \

--image $imageACR \

--cpu 2 \

--memory 2 \

--registry-login-server $serverACR \

--registry-username $userACR \

--registry-password $passACR \

--ip-address Public \

--dns-name-label $nomeACI \

--ports 8080

----------

chmod +x infraACI.sh

./infraACI.sh

----------

## Arquivo infraWebApp.sh

#!/bin/bash

grupoRecursos=rg-bcosql

regiao=brazilsouth

planService=planACRWebApp

sku=F1

appName=acrwebappRM550531

imageACR=rm550531acr.azurecr.io/argos-app:1.0

port=8080

if [ $(az group exists --name $grupoRecursos) = true ]; then
    echo "O grupo de recursos $grupoRecursos já existe"
    
else
    az group create --name $grupoRecursos --location $regiao
    echo "Grupo de recursos $grupoRecursos criado na localização $regiao"
    
fi

if az appservice plan show --name $planService --resource-group $grupoRecursos &> /dev/null; then
    echo "O plano de serviço $planService já existe"
    
else
    az appservice plan create --name $planService --resource-group $grupoRecursos --is-linux --sku $sku
    echo "Plano de serviço $planService criado com sucesso"
fi

if az webapp show --name $appName --resource-group $grupoRecursos &> /dev/null; then
    echo "O Serviço de Aplicativo $appName já existe"
    
else
    az webapp create --resource-group $grupoRecursos --plan $planService --name $appName --deployment-container-image-name $imageACR
    echo "Serviço de Aplicativo $appName criado com sucesso"
    
fi

if az webapp show --name $appName --resource-group $grupoRecursos > /dev/null 2>&1; then
    az webapp config appsettings set --resource-group $grupoRecursos --name $appName --settings WEBSITES_PORT=$port
    
fi

----------

chmod +x infraWebApp.sh

./infraWebApp.sh

----------

## Inline Script 

az container create \

--resource-group $(grupoRecursos) \

--name $(nomeACI) \

--image $(imageACRnoLabel):$(Build.BuildId) \

--cpu 2 \

--memory 2 \

--registry-login-server $(serverACR) \

--registry-username $(userACR) \

--registry-password $(passACR) \

--ip-address Public \

----------

## Variavel

grupoRecursos: rg-bcosql

imageACR: rm550531acr.azurecr.io/argos-app:1.0

imageACRnoLabel: rm550531acr.azurecr.io/argos-app

nomeACI: argosrm550531

passACR: <Senha> // GIT HUN NAO DEIXA COLOCAR SENHA

serverACR: rm550531acr.azurecr.io

----------

## Caso deseje verificar se a Imagem Existe no ACR

az acr repository show-tags --name rm550531acr --repository argos-app --output table

----------

## Caso deseje confirmar as Credenciais do ACR

az acr credential show --name rm550531acr

----------

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
