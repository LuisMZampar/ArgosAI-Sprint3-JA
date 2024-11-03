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

#Variáveis
grupoRecursos=rg-docker
regiao=eastus
nomeACR=javamsqlrm550531
skuACR=Basic


#Criação do Grupo de Recursos
Verifica a existência do grupo de recursos e se não existir, cria

if [ $(az group exists --name $grupoRecursos) = true ]; then
    echo "O grupo de recursos $grupoRecursos já existe"
else
    # Cria o grupo de recursos
    az group create --name $grupoRecursos --location $regiao
    echo "Grupo de recursos $grupoRecursos criado na localização $regiao"
fi

# Criação do Azure Container Registry
# Verifica se o ACR já existe

if az acr show --name $nomeACR --resource-group $grupoRecursos &> /dev/null; then
    echo "O ACR $nomeACR já existe"
else
    # Cria o ACR
    az acr create --resource-group $grupoRecursos --name $nomeACR --sku $skuACR
    echo "ACR $nomeACR criado com sucesso"
    # Habilita o Usuário Administrador no Azure Container Registry
    az acr update --name $nomeACR --resource-group $grupoRecursos --admin-enabled true
    echo "Habilitado com sucesso o usuário Administrador para o ACR $nomeACR"
fi

# Essa parte do Script só é recomendada para fins de testes e aprendizado
# Recuperar as credenciais do usuário administrador, armazenar em variáveis de ambiente e mostrar as credenciais

ADMIN_USER=$(az acr credential show --name $nomeACR --query "username" -o tsv)
ADMIN_PASSWORD=$(az acr credential show --name $nomeACR --query "passwords[0].value" -o tsv)

# Cria variáveis de ambiente

export ACR_ADMIN_USER=$ADMIN_USER
export ACR_ADMIN_PASSWORD=$ADMIN_PASSWORD

# Mostra as variáveis de ambiente

echo $ACR_ADMIN_USER
echo $ACR_ADMIN_PASSWORD


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
