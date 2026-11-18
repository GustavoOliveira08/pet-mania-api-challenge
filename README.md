# 🐾 Desafio PetMania — Correção e Estruturação da API

Este repositório contém a solução para o desafio de correção e finalização da API REST **PetMania**, desenvolvida com **Java** e **Spring Boot**. O objetivo principal foi identificar falhas estruturais, corrigir mapeamentos de rotas e persistência, e implementar o endpoint de remoção de registros.

## 📋 Visão Geral das Correções

Abaixo estão detalhadas as principais alterações realizadas no código-fonte e o porquê de cada decisão técnica:

### 1. Implementação do Endpoint `DELETE` e Correção de Rotas (`PetController.java`)

* **O que foi mudado:** Adicionado o método `deletar` anotado com `@DeleteMapping("/{id}")` e `@PathVariable Long id`, além da injeção de dependência via construtor para o `PetRepository`.
* **Por que essa mudança foi feita:** Anteriormente, não havia uma rota configurada para excluir um pet específico pelo seu identificador único. Além disso, o uso de `@PathVariable` garante que o ID passado na URL (ex: `/api/pets/2`) seja capturado corretamente pelo Spring, evitando erros de requisição ou ambiguidade com rotas base (`/api/pets`). A injeção via construtor garante imutabilidade (`final`) e facilita testes unitários.

```java
// Trecho organizado em PetController.java
private final PetRepository repository;

public PetController(PetRepository repository) {
    this.repository = repository;
}

@DeleteMapping("/{id}")
public String deletar(@PathVariable Long id) {
    repository.deleteById(id);
    return "Este pet foi removido.";
}
```

### 2. Mapeamento da Entidade e Enums (`Pet.java`, `Porte.java`, `Sexo.java`)

* **O que foi mudado:** Configuração das anotações `@Entity`, `@Id`, `@GeneratedValue(strategy = GenerationType.IDENTITY)` e `@Enumerated(EnumType.STRING)` nos atributos `sexo` e `porte`, acompanhados de construtores (vazio e parametrizado) e métodos *Getters/Setters*.
* **Por que essa mudança foi feita:**
  * O **JPA/Hibernate** exige um construtor padrão vazio (`public Pet()`) para instanciar a entidade ao realizar consultas no banco de dados.
  * O uso de `@Enumerated(EnumType.STRING)` garante que os valores dos Enums (`Macho`, `Fêmea`, `Pequeno`, `Médio`, `Grande`) sejam salvos como texto legível no MySQL, evitando problemas de ordenação caso novos valores sejam adicionados no futuro (o que ocorreria com o padrão `ORDINAL`).

```java
// Trecho organizado em Pet.java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

@Enumerated(EnumType.STRING)
private Sexo sexo;

@Enumerated(EnumType.STRING)
private Porte porte;
```

### 3. Camada de Persistência (`PetRepository.java`)

* **O que foi mudado:** Criação da interface `PetRepository` estendendo `JpaRepository<Pet, Long>` com a anotação `@Repository`.
* **Por que essa mudança foi feita:** Fornece automaticamente os métodos essenciais de CRUD (`findAll()`, `save()`, `deleteById()`, etc.) sem a necessidade de escrever consultas SQL manuais, conectando a entidade `Pet` ao banco de dados MySQL.

```java
@Repository
public interface PetRepository extends JpaRepository<Pet, Long> {
}
```

## 🧪 Evidências de Testes e Funcionamento

Os testes foram realizados utilizando o **Postman**, validados diretamente no banco de dados **MySQL (`petmania_db`)** e verificados na interface web da aplicação **PetMania**.

### 1. Cadastro de Pet (`POST /api/pets`)

Envio do payload JSON contendo os dados do pet (incluindo os valores mapeados pelos Enums `Sexo` e `Porte`). A API retornou status `200 OK` com o objeto persistido e seu `id` gerado automaticamente.

![Teste POST no Postman](img/Captura%20de%20tela%202026-11-11%20165445.png)

### 2. Estrutura do Banco de Dados (`petmania_db`)

Validação da criação automática da tabela `pet` e suas respectivas colunas no MySQL pelo Hibernate, refletindo exatamente os atributos definidos na classe `Pet.java`.

![Estrutura da Tabela no MySQL](img/Captura%20de%20tela%202026-11-11%20165525.png)

### 3. Validação da Rota de Exclusão (`DELETE /api/pets/{id}`)

Durante os testes no Postman, verificou-se a necessidade de informar o ID diretamente na URI para o correto funcionamento do `@DeleteMapping("/{id}")`:

* **Ajuste de Endpoint:** Configuração da requisição `DELETE` apontando para a rota base (`/api/pets`) e, em seguida, especificando o parâmetro `/2` na URL (`http://localhost:8080/api/pets/2`), retornando a confirmação `"Este pet foi removido."` com status `200 OK`.

| Rota Base (`/api/pets`) | Rota com PathVariable (`/api/pets/{id}`) | Confirmação da Remoção (`200 OK`) |
| :---: | :---: | :---: |
| ![DELETE Rota Base](img/Captura%20de%20tela%202026-11-11%20165903.png) | ![DELETE com ID](img/Captura%20de%20tela%202026-11-11%20165917.png) | ![Confirmação DELETE](img/Captura%20de%20tela%202026-11-11%20170054.png) |

### 4. Integração com a Interface Web (Front-end)

Validação visual no portal **PetMania**, demonstrando a exibição do registro cadastrado na API (com todos os campos mapeados corretamente) e a atualização da tela após a exclusão do registro.

| Pet Cadastrado Listado na Tela | Tela Atualizada Após Remoção |
| :---: | :---: |
| ![Pet Listado no Site](img/Captura%20de%20tela%202026-11-11%20170211.png) | ![Tela Vazia Após Exclusão](img/Captura%20de%20tela%202026-11-11%20170331%20-%20Copia.png) |

## 🛠️ Tecnologias Utilizadas

* **Java 25**
* **Spring Boot 4.1.2** (Spring Web MVC & Spring Data JPA)
* **MySQL** (Driver `mysql-connector-j`)
* **Maven** (Gerenciamento de dependências)
* **Postman** (Testes de integração da API)

## 🚀 Como Executar o Projeto

1. Certifique-se de ter o **MySQL** rodando localmente e crie o banco de dados:

   ```sql
   CREATE DATABASE petmania_db;
   ```

2. Configure as credenciais do banco no arquivo `src/main/resources/application.properties`.

3. Execute a aplicação via Maven:

   ```bash
   ./mvnw spring-boot:run
   ```

4. Utilize o Postman ou a interface web para consumir os endpoints disponíveis em `http://localhost:8080/api/pets`:

   * `GET /api/pets` — Lista todos os pets cadastrados.
   * `POST /api/pets` — Cadastra um novo pet.
   * `DELETE /api/pets/{id}` — Remove um pet pelo ID.