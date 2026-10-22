# Biblioteca Cidadã - Sistema de Gestão de Empréstimos

## Integrantes do grupo

- Adam Caldas Lopes - 01640350
- Márcio da Costa Ferreira Junior  - 01596976
- Wesley Ruan de Lima Silva - 01555915

> ⚠️ **Estado atual do repositório**: nesta branch existe apenas este `README.md`. O código-fonte (`src/`) e o `pom.xml` descritos abaixo **não estão versionados aqui**; os comandos de build/execução valem para o projeto completo, depois que esses arquivos forem adicionados.

## 📚 Sobre o Projeto

Sistema de gestão de empréstimos da Biblioteca Municipal desenvolvido em Java, implementando todos os pilares da Orientação a Objetos (OO): Encapsulamento, Herança, Polimorfismo e Abstração, além de demonstrar os conceitos de Agregação e Composição.

## 🎯 Objetivos de Aprendizagem

O projeto demonstra em código Java os pilares de OO:

- ✅ **Encapsulamento**: Atributos privados com getters/setters e validações
- ✅ **Herança**: Hierarquias de classes (Usuario → Aluno/Servidor/Visitante, Recurso → Livro/Revista/MidiaDigital)
- ✅ **Polimorfismo**: Sobrescrita de métodos com comportamento específico por tipo
- ✅ **Abstração**: 1 classe abstrata (Recurso) + 1 interface (Emprestavel)
- ✅ **Agregação**: Biblioteca agrega Recursos e Usuários (ciclos de vida independentes)
- ✅ **Composição**: Livro compõe Capítulos (dependência de vida)

## 🏗️ Arquitetura e Estrutura

### Estrutura de pastas esperada

```
.
├── pom.xml                      # Build Maven (Java 21, JUnit 5)
├── README.md
└── src
    ├── main/java/br/recife/biblioteca/   # Código (pacotes abaixo)
    └── test/java/br/recife/biblioteca/modelo/BibliotecaTest.java
```

### Packages

```
br.recife.biblioteca
├── modelo/              # Entidades do domínio
│   ├── Emprestavel.java        (Interface)
│   ├── Usuario.java            (Classe Abstrata)
│   ├── Aluno.java
│   ├── Servidor.java
│   ├── Visitante.java
│   ├── Recurso.java            (Classe Abstrata)
│   ├── Livro.java
│   ├── Revista.java
│   ├── MidiaDigital.java
│   ├── Capitulo.java           (Composição)
│   └── Emprestimo.java
├── servico/            # Lógica de negócio
│   └── Biblioteca.java
├── excecao/            # Exceções customizadas
│   ├── RecursoJaEmprestadoException.java
│   ├── RecursoNaoEncontradoException.java
│   └── UsuarioNaoEncontradoException.java
└── ui/                 # Interface do usuário
    └── BibliotecaCLI.java
```

## 🔍 Pilares de OO Implementados

### 1. Encapsulamento

**Localização**: Todas as classes do pacote `modelo`

**Como foi implementado**:
- Todos os atributos são `private`
- Acesso controlado via getters e setters públicos
- Validações nos setters (ex: título não vazio, ano > 0, ID positivo)
- Coleções internas não são expostas diretamente (retorna `Collections.unmodifiableList()`)

**Exemplos**:
```java
// Em Usuario.java
private String nome;

public void setNome(String nome) {
    if (nome == null || nome.trim().isEmpty()) {
        throw new IllegalArgumentException("Nome não pode ser vazio");
    }
    this.nome = nome.trim();
}

// Em Livro.java - não expõe lista interna
public List<Capitulo> getCapitulos() {
    return Collections.unmodifiableList(capitulos);
}
```

### 2. Herança

**Hierarquias implementadas**:

1. **Usuario** (abstrata) → **Aluno**, **Servidor**, **Visitante**
2. **Recurso** (abstrata) → **Livro**, **Revista**, **MidiaDigital**

**Benefícios**:
- Reuso de código comum (atributos e métodos da classe base)
- Especialização de comportamento nas subclasses
- Facilita manutenção e extensão

### 3. Polimorfismo

**Polimorfismo por Sobrescrita de Métodos**:

**Localização**: `Usuario.java` e suas subclasses

Cada tipo de usuário define seu próprio prazo e fator de multa:
```java
// Aluno.java
@Override
public int prazoDiasPadrao() { return 14; }
@Override
public double fatorMulta() { return 1.0; }

// Servidor.java
@Override
public int prazoDiasPadrao() { return 21; }
@Override
public double fatorMulta() { return 0.7; }

// Visitante.java
@Override
public int prazoDiasPadrao() { return 7; }
@Override
public double fatorMulta() { return 1.5; }
```

**Localização**: `Recurso.java` e suas subclasses

Cada tipo de recurso calcula multa de forma diferente:
```java
// Livro: R$ 2,00 por dia
@Override
public double calcularMulta(long diasAtraso) {
    return diasAtraso * 2.0 * getUsuarioAtual().fatorMulta();
}

// Revista: R$ 1,50 por dia após 3 dias de carência
@Override
public double calcularMulta(long diasAtraso) {
    long diasMultaveis = diasAtraso > 3 ? diasAtraso - 3 : 0;
    return diasMultaveis * 1.5 * getUsuarioAtual().fatorMulta();
}

// MidiaDigital: R$ 1,00 por dia
@Override
public double calcularMulta(long diasAtraso) {
    return diasAtraso * 1.0 * getUsuarioAtual().fatorMulta();
}
```

**Ligação Dinâmica**: O método correto é chamado em tempo de execução baseado no tipo real do objeto.

### 4. Abstração

#### Classe Abstrata: `Recurso.java`

**Por que abstrata?**: Não faz sentido ter um "Recurso genérico", apenas tipos específicos (Livro, Revista, MídiaDigital).

**Características**:
- Método abstrato: `abstract double calcularMulta(long diasAtraso)`
- Método concreto: `public String getDescricao()` (pode ser sobrescrito)
- Define estrutura comum para todos os recursos

#### Interface: `Emprestavel.java`

**Por que interface?**: Define um contrato para objetos que podem ser emprestados.

**Métodos**:
```java
boolean emprestar(Usuario u);
void devolver();
LocalDate getDataPrevistaDevolucao();
boolean isDisponivel();
```

**Implementação**: A classe `Recurso` implementa `Emprestavel`, fornecendo comportamento padrão que pode ser especializado.

### 5. Agregação

**Localização**: `Biblioteca.java`

**O que é**: Biblioteca **agrega** Recursos e Usuários, mas eles podem existir independentemente.

**Implementação**:
```java
public class Biblioteca {
    // AGREGAÇÃO: Recursos e Usuarios podem existir fora da Biblioteca
    private List<Recurso> acervo;
    private List<Usuario> usuarios;
    
    public void adicionarRecurso(Recurso recurso) { ... }
    public void adicionarUsuario(Usuario usuario) { ... }
}
```

**Por que agregação?**: 
- Um Recurso pode ser criado antes de ser adicionado à Biblioteca
- Um Usuario pode existir mesmo se a Biblioteca for destruída
- A Biblioteca apenas **referencia** esses objetos, não os **possui**

### 6. Composição

**Localização**: `Livro.java` e `Capitulo.java`

**O que é**: Livro **compõe** Capítulos - um Capítulo não existe sem o Livro.

**Implementação**:
```java
public class Livro extends Recurso {
    // COMPOSIÇÃO: Capítulos pertencem exclusivamente ao Livro
    private List<Capitulo> capitulos;
    
    public void adicionarCapitulo(Capitulo capitulo) {
        this.capitulos.add(capitulo);
    }
    
    public boolean removerCapitulo(int numero) {
        return capitulos.removeIf(c -> c.getNumero() == numero);
    }
}
```

**Por que composição?**:
- Capítulos são criados e gerenciados pelo Livro
- Se o Livro for destruído, os Capítulos deixam de existir
- Capítulos não têm sentido fora do contexto de um Livro
- Relação parte-todo com dependência de vida

## 🚀 Como Compilar e Executar

### Pré-requisitos

- Java 21 ou superior
- Maven 3.6 ou superior

### Compilar o projeto

```bash
mvn clean compile
```

### Executar a aplicação CLI

```bash
mvn exec:java -Dexec.mainClass="br.recife.biblioteca.ui.BibliotecaCLI"
```

Ou, após compilar, executar diretamente:

```bash
java -cp target/classes br.recife.biblioteca.ui.BibliotecaCLI
```

### Executar os testes

```bash
mvn test
```

### Empacotar o projeto

```bash
mvn package
```

## 🧪 Testes Unitários

O projeto inclui 7 testes JUnit 5 que cobrem:

1. ✅ **Prazo por tipo de usuário** (Polimorfismo)
2. ✅ **Fator de multa por tipo de usuário** (Polimorfismo)
3. ✅ **Cálculo de multa por tipo de recurso** (Polimorfismo)
4. ✅ **Composição de capítulos** (Composição)
5. ✅ **Validação de atributos** (Encapsulamento)
6. ✅ **Empréstimo e devolução** (Interface Emprestavel)
7. ✅ **Herança e toString()** (Herança)

**Localização**: `src/test/java/br/recife/biblioteca/modelo/BibliotecaTest.java`

## 📋 Funcionalidades Implementadas

### Menu Principal

1. **Gerenciar Usuários**
   - Cadastrar (Aluno, Servidor, Visitante)
   - Listar todos
   - Buscar por ID
   - Remover

2. **Gerenciar Acervo**
   - Cadastrar (Livro com capítulos, Revista, Mídia Digital)
   - Listar todos
   - Listar disponíveis
   - Listar emprestados
   - Buscar por ID
   - Remover

3. **Realizar Empréstimo**
   - Com validação de disponibilidade
   - Calcula data prevista de devolução baseada no tipo de usuário

4. **Realizar Devolução**
   - Calcula dias de atraso
   - Calcula multa baseada no tipo de recurso e usuário

5. **Relatórios**
   - Empréstimos ativos
   - Empréstimos em atraso (com cálculo de multa)
   - Histórico por usuário

## 🎮 Casos de Uso Implementados

Todos os casos de uso mínimos foram implementados:

1. ✅ Cadastrar 3 usuários (1 Aluno, 1 Servidor, 1 Visitante)
2. ✅ Cadastrar 3 itens (1 Livro com 3+ capítulos, 1 Revista, 1 MidiaDigital)
3. ✅ Emprestar Livro para Aluno; devolver com atraso e exibir multa
4. ✅ Emprestar MidiaDigital para Visitante (verificar prazo e multa)
5. ✅ Relatório de itens emprestados e em atraso
6. ✅ Impedir empréstimo de item já emprestado (com exceção)
7. ✅ Listar capítulos do Livro (composição visível no relatório)

## 🔒 Exceções Customizadas

- `RecursoJaEmprestadoException`: Lançada ao tentar emprestar recurso já emprestado
- `RecursoNaoEncontradoException`: Lançada quando recurso não é encontrado
- `UsuarioNaoEncontradoException`: Lançada quando usuário não é encontrado

## 📊 Regras de Negócio

### Prazos de Empréstimo
- **Aluno**: 14 dias
- **Servidor**: 21 dias
- **Visitante**: 7 dias

### Cálculo de Multa

**Fator de Multa por Usuário**:
- **Aluno**: 1.0 (multa normal)
- **Servidor**: 0.7 (30% de desconto)
- **Visitante**: 1.5 (50% a mais)

**Multa por Tipo de Recurso**:
- **Livro**: R$ 2,00 por dia × fator do usuário
- **Revista**: R$ 1,50 por dia (após 3 dias de carência) × fator do usuário
- **Mídia Digital**: R$ 1,00 por dia × fator do usuário

**Exemplo**:
- Aluno com Livro 5 dias atrasado: 5 × R$ 2,00 × 1.0 = R$ 10,00
- Servidor com Revista 5 dias atrasado: 2 × R$ 1,50 × 0.7 = R$ 2,10 (carência de 3 dias)
- Visitante com Mídia 5 dias atrasado: 5 × R$ 1,00 × 1.5 = R$ 7,50

## 🛠️ Tecnologias Utilizadas

- **Java 21**: Linguagem de programação
- **Maven**: Gerenciamento de dependências e build
- **JUnit 5**: Framework de testes unitários
- **Java SE**: Apenas bibliotecas padrão (sem frameworks)

## 👨‍💻 Autor

Grupo listado em "Integrantes do grupo", no início deste arquivo.

Sistema desenvolvido como atividade prática para demonstração dos pilares de Orientação a Objetos.

## 📝 Decisões de Projeto

### Por que Agregação para Biblioteca?
A Biblioteca **agrega** Recursos e Usuários porque:
- Esses objetos podem ser criados independentemente
- Podem existir antes ou depois da Biblioteca
- A Biblioteca apenas gerencia referências a eles
- Não há dependência de ciclo de vida

### Por que Composição para Livro e Capítulos?
O Livro **compõe** Capítulos porque:
- Capítulos não fazem sentido fora do contexto de um Livro específico
- São criados e gerenciados exclusivamente pelo Livro
- Se o Livro for removido, os Capítulos deixam de existir
- Relação parte-todo com dependência de vida forte

### Por que uma Classe Abstrata E uma Interface?
- **Classe Abstrata (Recurso)**: Quando há comportamento e estado comum a compartilhar
- **Interface (Emprestavel)**: Quando queremos definir um contrato sem implementação obrigatória

Isso demonstra flexibilidade: interfaces para contratos, classes abstratas para código comum.

## 📚 Conceitos Avançados Aplicados

- **Streams API**: Para filtros e buscas eficientes
- **Optional**: Para tratamento de valores ausentes
- **LocalDate**: Para manipulação de datas
- **Collections.unmodifiableList()**: Para encapsulamento seguro
- **Javadoc**: Documentação completa das classes principais
- **Exception Handling**: Tratamento robusto de erros
- **Scanner**: Para entrada de dados do usuário

## 🎓 Aprendizados

Este projeto demonstra de forma prática e completa:

1. Como aplicar os 4 pilares de OO em um sistema real
2. A diferença entre Agregação e Composição
3. Como usar classes abstratas e interfaces adequadamente
4. A importância do Encapsulamento para manter invariantes
5. O poder do Polimorfismo para comportamentos especializados
6. Como escrever testes unitários eficazes
7. Boas práticas de organização de código em packages

---

**Biblioteca Cidadã** - Um exemplo completo de Orientação a Objetos em Java! 🚀📚
