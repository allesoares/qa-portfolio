# Portfólio de Qualidade de Software — Alexandre Soares

Aqui reuni os projetos que desenvolvi durante o curso de Engenharia de Qualidade de Software da EBAC. Ao longo das aulas e dos exercícios, fui colocando em prática diferentes formas de testar uma aplicação, desde os primeiros testes de interface até automação de API, pipelines, containers e testes mobile.

Organizei os trabalhos por assunto para facilitar a navegação e mostrar um pouco dessa evolução. Em cada seção deixei os links para o código, os documentos e as instruções de execução. Alguns projetos foram crescendo ao longo do curso, por isso aparecem em mais de uma etapa do portfólio.

Para uma primeira leitura, a tabela abaixo leva direto à área de interesse. O [estudo de IA](estudo-ia/estudo-de-caso.md) também pode ser lido aqui no GitHub, com a história, os critérios, o plano de testes e o registro das conversas com o agente.

## Visão geral

| Área | Tecnologias e práticas |
| --- | --- |
| [Automação de UI](#1-automação-de-ui-com-cypress) | Cypress, JavaScript, Node.js, testes end-to-end |
| [API REST](#2-testes-automatizados-de-api-rest) | Cypress, Postman, autenticação e operações CRUD |
| [DevOps](#3-devops-e-integração-contínua) | GitHub Actions, Jenkins e relatórios Allure |
| [Containers](#4-projeto-com-docker) | Dockerfile e Docker Compose |
| [Automação de UI avançada](#5-automação-de-ui-com-cypress--avançado) | BDD com Cucumber, Intercept e App Actions |
| [Performance](#6-testes-de-performance) | JMeter e K6 |
| [Mobile](#7-automação-mobile-com-appium) | Appium, WebdriverIO, Mocha e Android |
| [Testes de unidade](#8-testes-de-unidade-com-javascript) | Jest, valores limite e cobertura de condições e decisões |
| [Inteligência Artificial](#9-testes-com-inteligência-artificial) | Análise de requisitos, critérios de aceitação e planejamento de testes |

## 1. Automação de UI com Cypress

Comecei a automação de interface com o **Hub de Leitura**, aplicação que usei em vários exercícios do curso. Nesse projeto, automatizei cenários de cadastro, login, contato e consulta ao catálogo, verificando tanto os fluxos com dados válidos quanto as mensagens de erro.

Conforme avancei nas aulas, passei a usar fixtures, comandos customizados, dados gerados com Faker e Page Objects para organizar melhor os testes e reaproveitar o código.

**Tecnologias:** Cypress, JavaScript, Node.js e Faker.

- [Projeto principal — hub-de-leitura-teste-ui](https://github.com/allesoares/hub-de-leitura-teste-ui/tree/main)
- [Exercício complementar — cadastro e login end-to-end](https://github.com/allesoares/hub-de-leitura-e2e)

Também fiz um exercício separado para testar o fluxo completo de cadastro e login: crio um usuário, valido o acesso, faço logout e entro novamente com as mesmas credenciais.

## 2. Testes automatizados de API REST

Nos testes de API, trabalhei com o catálogo de livros do **Hub de Leitura**. Usei o Cypress para enviar as requisições e conferir o status HTTP, os dados retornados e o comportamento dos endpoints, incluindo as operações que precisam de autenticação.

Os testes passam pelos seguintes cenários:

- Listagem de livros com filtros e paginação.
- Consulta dos detalhes de um livro.
- Cadastro com dados válidos e rejeição de dados inválidos.
- Atualização e exclusão de livros.
- Preparação e limpeza dos dados criados pelos testes.

**Tecnologias:** Cypress, JavaScript e API REST.

Deixei também a collection do Postman como referência dos testes manuais da API.

- [Projeto de automação — hub-de-leitura-api-teste-task](https://github.com/allesoares/hub-de-leitura-api-teste-task)
- [Collection do Postman — referência dos testes manuais](https://github.com/allesoares/hub-de-leitura-api/blob/main/postman/Hub-de-Leitura-Catalogo-Livros.postman_collection.json)
- [Aplicação e API utilizadas nos testes](https://github.com/allesoares/hub-de-leitura-api)

## 3. DevOps e integração contínua

Nesta etapa, configurei a execução dos testes em pipelines com GitHub Actions e Jenkins, junto com a geração dos relatórios.

### GitHub Actions

No projeto de testes de UI, configurei um workflow que prepara a aplicação e o banco de dados, executa o Cypress e publica o relatório Allure no GitHub Pages. Assim, consigo consultar o resultado da execução pelo próprio GitHub.

- [Workflow — Cypress e Allure](https://github.com/allesoares/hub-de-leitura-teste-ui/blob/main/.github/workflows/teste-ci.yml)

### Jenkins

Com o Jenkins, montei um pipeline para executar os testes de interface no Windows. Ele baixa os projetos, instala as dependências, inicia o Hub de Leitura e roda os testes no Chrome. Configurei as credenciais no Jenkins e a publicação dos resultados com Allure, screenshots e vídeos da execução.

- [Projeto — hub-de-leitura-teste-ui2](https://github.com/allesoares/hub-de-leitura-teste-ui2/tree/main)
- [Pipeline Jenkins](https://github.com/allesoares/hub-de-leitura-teste-ui2/blob/main/jenkinsfile)
- [Documentação de configuração do Jenkins](https://github.com/allesoares/hub-de-leitura-teste-ui2/blob/main/JENKINS.md)

## 4. Projeto com Docker

Usei o projeto de testes de UI para praticar duas formas de executar o Cypress com Docker: primeiro com um Dockerfile e depois separando a aplicação e os testes com Docker Compose.

### Dockerfile

Preparei uma imagem com a aplicação Hub de Leitura, as dependências dos testes, os navegadores e o Java usado pelo Allure. Com isso, o ambiente necessário para rodar os testes fica dentro do container.

- [Dockerfile — branch main](https://github.com/allesoares/hub-de-leitura-teste-ui/blob/main/Dockerfile)

### Docker Compose

Depois, organizei o ambiente em dois serviços: um para a aplicação e outro para os testes. Configurei uma verificação para que o Cypress só comece quando a aplicação estiver pronta, além dos volumes para acessar os relatórios e as evidências fora dos containers.

- [Projeto — branch exercicio-20-docker-compose](https://github.com/allesoares/hub-de-leitura-teste-ui/tree/exercicio-20-docker-compose)
- [Configuração do Docker Compose](https://github.com/allesoares/hub-de-leitura-teste-ui/blob/exercicio-20-docker-compose/docker-compose.yml)

## 5. Automação de UI com Cypress — avançado

### BDD com Cucumber

Neste projeto, pratiquei a escrita de cenários em Gherkin e a implementação dos steps em JavaScript com Cypress e Cucumber. Trabalhei com login válido e inválido, cadastro, edição e exclusão de livros. Também usei massa de dados e esquema de cenário para executar o cadastro com exemplos diferentes.

**Tecnologias:** Cypress, Cucumber, Gherkin e JavaScript.

- [Projeto — cucumber-cypress-exemplo](https://github.com/allesoares/cucumber-cypress-exemplo)

### Intercept e App Actions

Aqui explorei como controlar as respostas da API durante os testes de interface e preparar a aplicação antes de começar o fluxo que quero validar.

- Com **Intercept**, simulei o resultado de uma busca de livros e uma resposta de erro 500 para conferir como a interface se comporta em cada situação.
- Com **App Actions**, fiz a autenticação pela API para começar o teste com a sessão preparada e, em seguida, adicionar um livro à cesta pela interface.

**Tecnologias:** Cypress e JavaScript.

- [Projeto — exercicio-cy-avancado, branch main](https://github.com/allesoares/exercicio-cy-avancado/tree/main)

## 6. Testes de performance

### JMeter

No JMeter, montei um teste para o **BlazeDemo**, passando pela busca de voos, seleção de um voo e confirmação de uma compra fictícia. Deixei no repositório o plano de teste, a massa de dados e o relatório com os resultados da execução.

- [Projeto — exercicio-21-jmeter](https://github.com/allesoares/exercicio-21-jmeter)

### K6

Com o K6, fiz testes de **stress** e **spike** na consulta de livros do Hub de Leitura, variando a quantidade de usuários para observar o comportamento da aplicação. Além de validar as respostas, defini limites para o tempo de resposta e a taxa de falhas. Os relatórios, resumos e evidências das execuções estão no repositório.

- [Projeto — hub-de-leitura-k6](https://github.com/allesoares/hub-de-leitura-k6)

## 7. Automação mobile com Appium

Para praticar automação mobile, usei o **WDIO Demo App** em um emulador Android com Appium, WebdriverIO e Mocha. Automatizei o login demonstrativo do aplicativo, conferindo a mensagem de sucesso e o fechamento do alerta. Organizei os seletores e as ações em Screen Objects para separar essa parte das validações do teste.

**Tecnologias:** Appium, WebdriverIO, Mocha, UiAutomator2, JavaScript e Android.

- [Projeto — teste-wdio](https://github.com/allesoares/teste-wdio)

## 8. Testes de unidade com JavaScript

Neste projeto, escrevi testes com Jest para as funções de validação de cadastro que implementei no módulo de JavaScript. As regras verificam se o usuário é maior de idade, se o e-mail contém `@` e se o cadastro está ativo.

Ao todo, são **19 cenários**. Testei as idades de 17, 18 e 19 anos para trabalhar os valores próximos ao limite, os diferentes valores do campo ativo e as oito combinações das regras do cadastro. Também configurei o Jest para medir a cobertura e exigir 100% de cobertura de instruções, branches, funções e linhas.

**Tecnologias:** JavaScript, Node.js e Jest.

- [Projeto — modulo-28-jest](https://github.com/allesoares/modulo-28-jest)
- [Exercício de JavaScript que originou as funções — módulo 27](https://github.com/allesoares/exercicio-js)

## 9. Testes com Inteligência Artificial

Neste estudo do módulo 30, usei um agente de IA para analisar uma história do Hub de Leitura sobre bloqueio de usuários por atraso na devolução de livros. Pedi ajuda para identificar ambiguidades, propor critérios de aceitação e gerar cenários de teste. Depois revisei as respostas, solicitei complementos e identifiquei cenários repetidos.

O material é um estudo documental de análise e planejamento. Organizei a apresentação começando pela história, pelos 14 critérios refinados e pelo plano de testes. Na sequência, mantive os prompts e as respostas registrados no exercício, incluindo a proposta de consolidação de 30 para 25 cenários ou esquemas.

Os esclarecimentos com produto foram simulados, e quatro cenários continuam marcados como condicionais. Os cenários foram planejados; este estudo não inclui implementação nem evidência de execução.

- [Ler o estudo e as conversas no GitHub](estudo-ia/estudo-de-caso.md)
- [Versão para apresentação em PDF](estudo-ia/analise-de-requisitos-e-testes-com-ia.pdf)
- [Versão editável em Word](estudo-ia/analise-de-requisitos-e-testes-com-ia.docx)
- [Documento original do exercício](Modulo%2030%20-%20Cria%C3%A7%C3%A3o%20crit%C3%A9rios%20e%20cen%C3%A1rios%20de%20teste%20com%20IA.docx)

## Como explorar os projetos

Para conhecer melhor ou executar algum dos projetos, é só abrir o repositório e seguir as instruções do README. Nos exercícios com Docker, os links já apontam para a branch correspondente.

Os trabalhos fazem parte dos meus estudos na EBAC e usam aplicações, exemplos e projetos-base fornecidos ao longo do curso. As referências estão nos respectivos repositórios.
