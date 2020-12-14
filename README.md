# Trabalho Individual - GCES - 2020/1

A Gestão de Configuração de Software é parte fundamental no curso de GCES, e dominar os conhecimentos de configuração de ambiente, containerização, virtualização, integração e deploy contínuo tem se tornado cada vez mais necessário para ingressar no mercado de trabalho.

Para exercitar estes conhecimentos, você deverá aplicar os conceitos estudados ao longo da disciplina no produto de software contido neste repositório.

O sistema se trata de uma aplicação Web, cuja funcionalidade consiste na pesquisa e exibição de perfis de usuários do GitHub, que é composta de:

- Front End escrito em Javascript, utilizando os frameworks Vue.JS e Quasar;
- Back End escrito em Ruby on Rails, utilizado em modo API;
- Banco de Dados PostgreSQL;

Para executar a aplicação na sua máquina, basta seguir o passo-a-passo descrito no arquivo [Descrição e Instruções](Descricao-e-Instrucoes.md).

## Critérios de avaliação

### 1. Containerização

A aplicação deverá ter seu ambiente completamente containerizado. Desta forma, cada subsistema (Front End, Back End e Banco de Dados) deverá ser isolado em um container individual.

Deverá ser utilizado um orquestrador para gerenciar comunicação entre os containers, o uso de credenciais, networks, volumes, entre outras configurações necessárias para a correta execução da aplicação.

Para realizar esta parte do trabalho, recomenda-se a utilização das ferramentas:

- Docker versão 17.04.0+
- Docker Compose com sintaxe na versão 3.2+

### 2. Integração contínua

Você deverá criar um 'Fork' deste repositório, onde será desenvolvida sua solução. Nele, cada commit submetido deverá passar por um sistema de integração contínua, realizando os seguintes estágios:

- Build: Construção completa do ambiente;
- Testes: Os testes automatizados da aplicação devem ser executados;
- Coleta de métricas: Deverá ser realizada a integração com algum serviço externo de coleta de métricas de qualidade;

O sistema de integração contínua deve exibir as informações de cada pipeline, e impedir que trechos de código que não passem corretamente por todo o processo sejam adicionados à 'branch default' do repositório.

Para esta parte do trabalho, poderá ser utilizada qualquer tecnologia ou ferramenta que o aluno desejar, como GitlabCI, TravisCI, CircleCI, Jenkins, CodeClimate, entre outras.

### 3. Deploy contínuo (Extra)

Caso cumpra todos os requisitos descritos acima, será atribuída uma pontuação extra para o aluno que configure sua pipeline de modo a publicar a aplicação automaticamente, sempre que um novo trecho de código seja integrado à branch default.


## Solução Implementada

Optou-se pela manutenção de um monorepo contendo os serviços api e client. Desta forma, os diretórios api e client foram utilizados como contexto para build de seus respectivos Dockerfiles.

As imagens foram construídas utilizando:
- Docker version 19.03.8, build afacb8b7f0
- Docker-compose 3.2

A construção da pipeline foi feita utilizando a ferramenta TravisCI, disponível no link:

https://travis-ci.com/github/Yoshida-Eduardo/Trabalho-Individual-2020-1

A coleta de métricas de análise estática do código foram feitas utilizando o Sonar, disponível no link:

https://sonarcloud.io/dashboard?id=yoshida-eduardo_Trabalho-Individual-2020-1


### 1. Containerização

#### 1.1 API

A imagem da Api foi baseada em uma imagem Alpine por possuir uma menor área de superfície. Foi realizada uma build multi-stage, pois algumas dependências são necessárias apenas para a fase de instalação das Gems, sendo desnecessárias na imagem final.

#### 1.2 Client

A imagem do Client também foi baseada em uma imagem Alpine, visando menor área de superfície. Também foi criada uma imagem para produção, utilizando o Nginx para servir os arquivos estáticos gerados pelo yarn build. No CI é realizada a build da imagem de produção com base na imagem de desenvolvimento, após a imagem de desenvolvimento ter seus artefatos testados. Desta forma garante-se que a imagem de produção seja gerada com artefatos testados.

### 2. Integração Contínua

A pipeline foi criada pensando em manter a arquitetura monorepo, há dois build stages, com um total de três jobs.

Os build stages são:
- Static Analysis, onde ocorre a coleta de métricas de análise estática de código pelo Sonar;
- Build, onde ocorrem, em jobs paralelos a build, testes e seria implementado o deploy dos serviços Api e Client, sendo os jobs independentes entre si;

Foi feita a escolha pela utilização de jobs paralelos na build, buscando que Api e Client fossem independentes entre si, uma vez que são serviços diferentes, commits realizados na Api não deveriam gerar uma build e deploy do Client e vice versa.

Para evitar builds e deploys desnecessários em cada serviço, foi adicionada na fase inicial de cada job o comando

```
git diff --name-only $TRAVIS_COMMIT_RANGE | grep -qE $REGEX || exit 0
```

Com o objetivo de verificar os arquivos modificados no range de commits analisados pela build, caso nenhum arquivo referente ao respectivo serviço seja alterado, o job deste serviço será abortado, evitando deploys desnecessários.

### 3. Deploy

Não foi realizado o deploy da aplicação.