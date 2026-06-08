# ORBIT API

**Plataforma inteligente de apoio à decisão para operações críticas em ambientes extremos**

FIAP Global Solution 2026/1 — Tema: Economia Espacial

---

## Equipe

| Nome | RM |
|------|-----|
| Gabriely Bonfim Silva | RM566242 |
| Henrique Rodrigues Vespasiano | RM562917 |
| Mirelly Sousa Alves | RM566299 |
| Ruan Luca Feliciano | RM562218 |

---

## Sobre o Projeto

O ORBIT é uma API REST que auxilia equipes operacionais a tomar decisões críticas em missões de campo — terrestres, remotas ou futuras missões orbitais/espaciais.

Quando uma missão entra em estado crítico, o sistema analisa:

- Autonomia e nível de combustível do veículo
- Distância até cada ponto de apoio disponível (cálculo com **fórmula de Haversine**)
- Recursos disponíveis no ponto (combustível, medicina, suprimentos, energia)
- Nível de risco da rota
- Condição física média dos operadores  

E retorna uma lista **rankeada com justificativa** de qual ponto de apoio deve ser acionado primeiro, usando um **algoritmo de score ponderado**.

---

## Stack Tecnológica

- **Java 21** (compatível com Java 24 via Lombok 1.18.38)
- **Spring Boot 3.3.5**
- **Spring Data JPA** com Oracle (produção) / H2 (desenvolvimento)
- **Spring Security + JWT** (jjwt 0.12.6)
- **Spring HATEOAS**
- **Springdoc OpenAPI 2.6.0** (Swagger UI)
- **Lombok 1.18.38**
- **Oracle JDBC** (ojdbc11 23.4.0.24.05)

---

## Requisitos Java Advanced Atendidos

| Requisito | Implementação |
|-----------|---------------|
| Spring Boot + JPA | `spring-boot-starter-data-jpa`, entidades com `@Entity`, `@Table`, sequences Oracle |
| Herança JPA | `Veiculo` abstrata com `@Inheritance(JOINED)` + `VeiculoTerrestre`, `VeiculoAereo` |
| Embeddable | `Coordenadas` com `@Embeddable` + `@AttributeOverrides` em `Missao` |
| Chave composta | `MissaoRecursoId` com `@EmbeddedId` + `@MapsId` |
| Paginação | `Page<T>` + `Pageable` em todos os GETs principais |
| HATEOAS | `EntityModel` com `linkTo`/`methodOn` em todos os controllers |
| Swagger/OpenAPI | Documentação completa em `/swagger-ui.html` |
| Validação | `@Valid`, `@NotBlank`, `@DecimalMin/Max`, `@Min`, `@Max`, `@Email` nos DTOs |
| Spring Security + JWT | Autenticação stateless com Bearer token em todos os endpoints |
| Java Records | Todos os DTOs de request/response são records |
| Exception Handler | `@RestControllerAdvice` com `GlobalExceptionHandler` |
| Perfis | `application.properties` (Oracle prod) + `application-dev.properties` (H2 dev) |
| DataInitializer | Dados de exemplo carregados no startup (perfil dev) |

---

## Executar Localmente (Perfil Dev — H2)

```bash
# Clonar e entrar no projeto
cd /Users/riquezada/Desktop/OBITS/orbit-api

# Compilar e rodar com H2 em memória
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Ou com Java direto após build
./mvnw package -DskipTests
java -jar target/orbit-api-1.0.0.jar --spring.profiles.active=dev
```

A aplicação sobe em `http://localhost:8080`

---

## Executar com Oracle (Produção)

Configure as variáveis de ambiente e execute sem o perfil dev:

```bash
export ORACLE_URL=jdbc:oracle:thin:@oracle.fiap.com.br:1521:ORCL
export ORACLE_USERNAME=rm562917
export ORACLE_PASSWORD=@rm562917
export JWT_SECRET=minha-chave-secreta-forte

java -jar target/orbit-api-1.0.0.jar
```

---

### Autenticação

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| POST | `/api/auth/registrar` | Registrar novo usuário |
| POST | `/api/auth/login` | Login e obtenção do JWT |

rm562917@fiap.com.br
orbit2026

> Todos os demais endpoints requerem o header: `Authorization: Bearer <token>`

### Missões

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/missoes` | Listar (paginado) |
| GET | `/api/missoes/{id}` | Buscar por ID |
| GET | `/api/missoes/status/{status}` | Filtrar por status |
| GET | `/api/missoes/criticas?nivelMinimo=7.0` | Missões de alto risco |
| POST | `/api/missoes` | Criar |
| PUT | `/api/missoes/{id}` | Atualizar |
| DELETE | `/api/missoes/{id}` | Deletar |

### Recomendação (endpoint principal)

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/recomendacoes/missao/{missaoId}` | Recomendar pontos de apoio para a missão |

### Veículos

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/veiculos` | Listar (paginado) |
| GET | `/api/veiculos/{id}` | Buscar por ID |
| POST | `/api/veiculos/terrestre` | Cadastrar veículo terrestre |
| POST | `/api/veiculos/aereo` | Cadastrar veículo aéreo |
| DELETE | `/api/veiculos/{id}` | Desativar |

### Pontos de Apoio

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/pontos-apoio` | Listar (paginado) |
| GET | `/api/pontos-apoio/{id}` | Buscar por ID |
| GET | `/api/pontos-apoio/ativos` | Listar ativos |
| POST | `/api/pontos-apoio` | Criar |
| PUT | `/api/pontos-apoio/{id}` | Atualizar |
| DELETE | `/api/pontos-apoio/{id}` | Desativar |

### Recursos, Pessoas

- `GET/POST/PUT/DELETE /api/recursos`
- `GET/POST/PUT/DELETE /api/pessoas`

---

## Documentação Interativa

Com a aplicação rodando, acesse:

- **Swagger UI:** `http://localhost:8080/swagger-ui.html`
- **OpenAPI JSON:** `http://localhost:8080/api-docs`
- **H2 Console (dev):** `http://localhost:8080/h2-console`
  - JDBC URL: `jdbc:h2:mem:orbitdb`

---

## Algoritmo de Recomendação

Score ponderado com os seguintes pesos:

| Fator | Peso |
|-------|------|
| Distância (Haversine) | 35% |
| Nível de combustível do veículo | 25% |
| Recursos disponíveis no ponto | 20% |
| Risco da rota (inverso do nivelRisco) | 10% |
| Condição física média dos operadores | 10% |

Pontos com `ativo = false` são excluídos automaticamente. A resposta retorna a lista rankeada com posição, distância, score total e justificativa textual.

---

## Estrutura do Projeto

```
orbit-api/src/main/java/br/com/fiap/orbit/
├── OrbitApplication.java
├── DataInitializer.java
├── config/
│   ├── OpenApiConfig.java
│   └── SecurityConfig.java
├── controller/
│   ├── AuthController.java
│   ├── MissaoController.java
│   ├── VeiculoController.java
│   ├── PessoaController.java
│   ├── PontoDeApoioController.java
│   ├── RecursoController.java
│   └── RecomendacaoController.java
├── dto/
│   ├── request/   (Java Records com validações)
│   └── response/  (Java Records)
├── exception/
│   ├── BusinessException.java
│   ├── ResourceNotFoundException.java
│   ├── ErrorResponse.java
│   └── GlobalExceptionHandler.java
├── model/
│   ├── enums/
│   ├── Coordenadas.java      (@Embeddable)
│   ├── Usuario.java
│   ├── Veiculo.java          (abstract, JOINED inheritance)
│   ├── VeiculoTerrestre.java
│   ├── VeiculoAereo.java
│   ├── Missao.java
│   ├── Pessoa.java
│   ├── PontoDeApoio.java
│   ├── Recurso.java
│   ├── MissaoRecurso.java    (@EmbeddedId + @MapsId)
│   └── MissaoRecursoId.java  (@Embeddable + Serializable)
├── repository/               (JpaRepository + Pageable + @Query)
├── security/
│   ├── JwtService.java
│   ├── JwtAuthenticationFilter.java
│   └── UserDetailsServiceImpl.java
└── service/
    ├── AuthService.java
    ├── MissaoService.java
    ├── VeiculoService.java
    ├── PessoaService.java
    ├── PontoDeApoioService.java
    ├── RecursoService.java
    └── RecomendacaoService.java
```

## LINK RENDER https://orbit-api-uo91.onrender.com/swagger-ui/index.html OU https://orbit-api-uo91.onrender.com

## github https://github.com/HenriqueRodriguesV/orbit_api/tree/main/orbit-api

## video yt https://youtu.be/_kELwq2UnOU