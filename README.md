# ArgosAI-Sprint3

Para a matéria de DevOps

Este projeto é uma API desenvolvida em Java usando Spring Boot, com integração ao banco de dados SQL no Azure. Ele oferece um CRUD completo para produtos, 
clientes e vendas. Este documento fornece instruções detalhadas sobre como realizar o deploy e testar a API.

Pré-requisitos
Java 17 instalado.
Maven para gerenciar as dependências.
Azure CLI instalado e configurado.
Acesso a uma conta no Azure para criação dos recursos necessários.

1. Clonar o Repositório
git clone https://github.com/LuisMZampar/ArgosAI-Sprint3-JA.git

cd ArgosAI-Sprint3-JA

3. Compilar e empacotar a aplicação
mvn clean package

4. Criação do Banco de Dados no Azure SQL
Execute os seguintes comandos para criar o banco de dados e o servidor SQL:

az group create --name rg-bcosql --location brazilsouth

az sql server create -l brazilsouth -g rg-bcosql -n sqlserver-rm550531 -u admsql -p devops@Fiap2tds --enable-public-network true

az sql db create -g rg-bcosql -s sqlserver-rm550531 -n argos --service-objective Basic --backup-storage-redundancy Local --zone-redundant false

az sql server firewall-rule create -g rg-bcosql -s sqlserver-rm550531 -n AllowAll --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

5. Deploy no Azure App Service
az webapp deploy --resource-group rg-argos --name argos-rm550531 --src-path "target/ArgosAI-Sprint3-0.0.1-SNAPSHOT.jar"

6. Monitoramento de Logs
az webapp log tail --resource-group rg-argos --name argos-rm550531

## Visão Geral

ArgosAI-Sprint3 é um projeto desenvolvido para gerenciar clientes e produtos em um sistema de recomendação. Ele faz uso de APIs RESTful, documentação com Swagger, e a arquitetura segue padrões como HATEOAS (Hypermedia as the Engine of Application State) para uma navegação mais dinâmica entre recursos.

## Tecnologias Utilizadas

- **Java 17**: Linguagem principal do projeto.
- **Spring Boot**: Framework para simplificar o desenvolvimento, especialmente para a criação de APIs RESTful.
- **Maven**: Ferramenta de gerenciamento de dependências e build.
- **Oracle SQL**: Banco de dados relacional utilizado para persistência dos dados.
- **Hibernate**: Framework de ORM (Object Relational Mapping) para gerenciar as entidades JPA.
- **Swagger/OpenAPI**: Utilizado para gerar a documentação da API automaticamente.
- **ModelMapper**: Para converter entidades em DTOs e vice-versa.
- **Lombok**: Para reduzir o código boilerplate, como getters, setters e construtores.
- **Thymeleaf**: Template engine para renderização de páginas HTML.
- **HATEOAS**: Para fornecer links de navegação entre recursos REST.
- **Fly.io**: Plataforma para deployment da aplicação.

## Arquitetura e Design Pattern

A arquitetura segue o padrão de **Camadas (Layered Architecture)**, separando responsabilidades em:

- **Controller**: Manipula as requisições HTTP e orquestra o fluxo de dados.
- **Service**: Contém a lógica de negócios da aplicação.
- **Repository**: Responsável pela comunicação com o banco de dados.

O projeto também utiliza o design pattern **DTO (Data Transfer Object)** para encapsular e transmitir dados entre a API e os serviços. O **ModelMapper** facilita a conversão entre os modelos e os DTOs.

**HATEOAS** é implementado para incluir links de navegação entre as respostas das APIs, permitindo que o cliente possa explorar os recursos relacionados sem precisar conhecer todos os endpoints.

## Explicação das Classes

### Configurações

- **CorsConfig**: Define as permissões de CORS, permitindo que o frontend, rodando em outra origem (localhost ou Fly.io), se comunique com a API.
- **ModelMapperConfig**: Configura o `ModelMapper` para realizar a conversão entre entidades e DTOs.
- **SwaggerConfig**: Configura o Swagger para a documentação da API, que pode ser acessada via `/swagger-ui.html`.

### Controllers

#### ClienteController

Define endpoints para gerenciar clientes:
- `GET /api/clientes`: Retorna todos os clientes.
- `GET /api/clientes/{id}`: Retorna um cliente por ID.
- `POST /api/clientes`: Cria um novo cliente.
- `PUT /api/clientes/{id}`: Atualiza um cliente existente.
- `DELETE /api/clientes/{id}`: Exclui um cliente.

Implementa HATEOAS, adicionando links de navegação às respostas.

#### ProdutoController

Define endpoints para gerenciar produtos:
- `GET /api/produtos`: Retorna todos os produtos.
- `GET /api/produtos/{id}`: Retorna um produto por ID.
- `POST /api/produtos`: Cria um novo produto.
- `PUT /api/produtos/{id}`: Atualiza um produto existente.
- `DELETE /api/produtos/{id}`: Exclui um produto.

Também utiliza HATEOAS para fornecer links de navegação.

#### HomeController

Responsável por gerenciar a página inicial da aplicação, que é acessada via `/`.

#### ClienteThymeleafController e ProdutoThymeleafController

São controladores para as páginas renderizadas com Thymeleaf, permitindo a interação com os clientes e produtos através de formulários HTML.

### DTOs

- **ClienteDto**: DTO utilizado para transferir dados de clientes. Inclui validações de campo como `@NotBlank` e `@Size` para garantir que os dados estejam no formato correto.
- **ProdutoDto**: DTO para transferência de dados de produtos. Também inclui validações como `@NotBlank` e `@Positive` para garantir consistência nos dados.

### Modelos

- **Cliente**: Entidade JPA que mapeia a tabela `TB_ARGOS_CLIENTES` no banco de dados. Inclui campos como `nome`, `idade`, `cpf`, `email` e `celular`, todos com validações apropriadas.
- **Produto**: Entidade JPA que mapeia a tabela `TB_ARGOS_PRODUTOS`. Inclui campos como `nome`, `descricao`, `preco`, `quantidade` e `imagem`.

### Serviços

- **ClienteService**: Responsável pela lógica de negócios relacionada aos clientes, como buscar, salvar e deletar clientes.
- **ProdutoService**: Lida com a lógica de negócios para produtos, oferecendo métodos para buscar, salvar e deletar produtos.

### Exceções

- **ResourceNotFoundException**: Exceção personalizada lançada quando um recurso solicitado não é encontrado no banco de dados.

### Repositórios

- **ClienteRepository**: Interface que herda de `JpaRepository`, fornecendo métodos de busca e persistência para a entidade `Cliente`.
- **ProdutoRepository**: Interface que herda de `JpaRepository` para gerenciar a persistência de `Produto`.

## Endpoints

### Clientes

- `GET /api/clientes`: Lista todos os clientes.
- `GET /api/clientes/{id}`: Obtém um cliente por ID.
- `POST /api/clientes`: Cria um novo cliente.
- `PUT /api/clientes/{id}`: Atualiza um cliente.
- `DELETE /api/clientes/{id}`: Exclui um cliente.

### Produtos

- `GET /api/produtos`: Lista todos os produtos.
- `GET /api/produtos/{id}`: Obtém um produto por ID.
- `POST /api/produtos`: Cria um novo produto.
- `PUT /api/produtos/{id}`: Atualiza um produto.
- `DELETE /api/produtos/{id}`: Exclui um produto.

## Dependências

- **Spring Boot Starter Web**: Para a criação de APIs RESTful.
- **Spring Boot Starter Data JPA**: Para integração com o banco de dados usando Hibernate.
- **Spring Boot Starter Thymeleaf**: Para renderização de páginas HTML.
- **ModelMapper**: Para conversão entre entidades e DTOs.
- **Lombok**: Para reduzir código boilerplate, como getters e setters.
- **Springdoc OpenAPI**: Para a documentação automática da API.
- **Oracle JDBC Driver**: Para conexão com o banco de dados Oracle.
- **HikariCP**: Para otimização da conexão com o banco de dados.

## Desenho UML

![ArgosJaSprint3Uml](https://github.com/user-attachments/assets/f92eb320-244f-4f7e-b632-8bf716d74276)


## Deploy

O deploy é realizado utilizando Fly.io, com o arquivo `fly.toml` configurando o serviço e o Docker sendo utilizado para compilar e executar a aplicação. O processo envolve duas etapas principais:

1. **Build** da aplicação usando Maven para gerar o arquivo `.jar`.
2. **Deploy** da aplicação com Fly.io, expondo a porta 8080 para a API.
