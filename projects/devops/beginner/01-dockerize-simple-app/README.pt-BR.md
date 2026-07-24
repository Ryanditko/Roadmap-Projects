# Dockerizar uma Aplicação Simples

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Pegue uma aplicação que você já tem — um pequeno servidor web, uma API ou uma CLI — e empacote-a para que rode de forma idêntica em qualquer máquina com Docker. O ponto não é apenas "sobe em um contêiner", mas uma imagem enxuta e reproduzível: uma base fixada, só os arquivos que você realmente precisa, um usuário não-root e uma porta exposta de forma limpa. No caminho você conhece o cache de camadas, a separação entre o estágio de build e o de runtime, e por que um `.dockerignore` vale o seu lugar. Ao terminar, `docker run` entrega a qualquer pessoa exatamente o ambiente que você tinha — e aposenta a desculpa do "na minha máquina funciona".

## Pré-requisitos

- Uma aplicação que inicia com um único comando (qualquer linguagem)
- Docker instalado localmente com o daemon em execução
- Conforto básico com a linha de comando (caminhos, variáveis de ambiente, portas)
- Uma ideia clara da versão do runtime e das dependências da sua app

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Escrever um Dockerfile que constrói uma imagem executável a partir do código-fonte
- Usar um build multi-estágio para manter o ferramental de build fora da imagem final
- Explicar o cache de camadas e ordenar instruções para aproveitá-lo
- Injetar configuração em um contêiner via variáveis de ambiente e portas
- Rodar o contêiner como usuário não-root e explicar por que isso importa

## Requisitos Funcionais

1. A imagem deve construir a partir de um único `docker build`, sem passos manuais.
2. O contêiner em execução deve servir ou executar a app em uma porta documentada.
3. O build deve ser multi-estágio para que ferramentas de build fiquem ausentes da imagem final.
4. Um `.dockerignore` deve excluir controle de versão, dependências e segredos do contexto de build.
5. O contêiner deve rodar como usuário não-root.
6. A imagem deve carregar uma tag de versão explícita, não apenas `latest`.
7. Configurações como porta e nível de log devem ser injetáveis via variáveis de ambiente.

## Marcos Sugeridos

1. **Marco 1 — Primeiro build:** Escreva um Dockerfile de estágio único, construa-o e rode a app em um contêiner.
2. **Marco 2 — Enxugue:** Divida em estágios de build e runtime, adicione `.dockerignore` e fixe a imagem base.
3. **Marco 3 — Endureça e versione:** Adicione um usuário não-root, config por ambiente, uma tag de versão e um comando de execução documentado.

## Esboço de Dados e Interface

```text
Layout do projeto
  Dockerfile
  .dockerignore
  <fonte da app>

Estrutura do Dockerfile (estágios, não o arquivo completo)
  estágio "build":
    FROM <base>:<versão-fixada>
    copia manifesto de dependências -> instala deps   (camada em cache)
    copia fonte -> compila/gera artefato
  estágio "runtime":
    FROM <base-enxuta>:<versão-fixada>
    cria + muda para usuário não-root
    copia artefato de "build"
    EXPOSE <porta>
    ENV APP_PORT / LOG_LEVEL
    ENTRYPOINT / CMD

Comandos
  docker build -t myapp:1.0.0 .
  docker run -p 8080:8080 -e LOG_LEVEL=info myapp:1.0.0
```

## Desafios Extras

- Envie a imagem para um registro (Docker Hub, GHCR) e baixe-a em outra máquina.
- Adicione uma instrução `HEALTHCHECK` para que o Docker reporte a saúde do contêiner.
- Escaneie a imagem com `docker scout` ou Trivy e corrija um achado real.
- Reduza ainda mais com uma base Alpine ou distroless e compare os tamanhos.

## Definição de Pronto

- [ ] `docker build` produz uma imagem sem erros a partir de um checkout limpo.
- [ ] A imagem final não contém compiladores nem dependências exclusivas de build.
- [ ] O contêiner roda como usuário não-root (verifique com `whoami` dentro dele).
- [ ] Valores de config mudam o comportamento via `-e` sem reconstruir.
- [ ] A imagem carrega uma tag de versão semântica explícita.

## Armadilhas Comuns

- Copiar todo o contexto (incluindo `node_modules` ou `.git`), inflando o tempo de build e o tamanho da imagem.
- Colocar `COPY . .` antes da instalação de dependências, invalidando o cache a cada mudança de fonte.
- Rodar como root por padrão, deixando o contêiner com privilégios em excesso.
- Fixar segredos ou valores específicos de ambiente na imagem em vez de injetá-los em tempo de execução.

## Recursos

- [Docker: Boas práticas de Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) — cache de camadas, multi-estágio e enxugamento.
- [Docker: Builds multi-estágio](https://docs.docker.com/build/building/multi-stage/) — o padrão oficial para imagens enxutas.
- [Docker: Referência do .dockerignore](https://docs.docker.com/build/concepts/context/#dockerignore-files) — controle o que entra no contexto de build.
- [Snyk: 10 boas práticas de segurança para imagens Docker](https://snyk.io/blog/10-docker-image-security-best-practices/) — usuários não-root e escaneamento.
