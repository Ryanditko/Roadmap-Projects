# Gerenciador de Configuração por Ambiente

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa uma pequena ferramenta que carrega a configuração certa para o ambiente certo — `dev`, `staging`, `prod` — e entrega à aplicação um único conjunto de valores validados. A ideia central é a sobreposição em camadas: uma config base compartilhada, sobrescrita por arquivos específicos de ambiente, sobrescrita por variáveis de ambiente no topo. No caminho você separa segredos de configurações comuns, verifica que as chaves obrigatórias existem antes de a app iniciar e falha em alto e bom som diante de uma config ruim em vez de quebrar misteriosamente três requisições depois. Esta é a ferramenta humilde que torna a config "twelve-factor" real: uma base de código, muitos deploys, sem `if (env === 'prod')` espalhado pelo código.

## Pré-requisitos

- Uma linguagem com biblioteca de config ou de parsing de arquivos (qualquer stack)
- Entender variáveis de ambiente e como processos as leem
- Um formato de arquivo para trabalhar: JSON, YAML ou TOML
- Ciência de que segredos nunca devem ser commitados no controle de versão

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Sobrepor fontes de configuração com uma ordem de precedência clara
- Separar configurações não-secretas de segredos e carregar cada uma adequadamente
- Validar a configuração contra um schema antes de a app usá-la
- Fornecer padrões sensatos permitindo sobrescritas por ambiente
- Falhar cedo com uma mensagem clara quando uma config obrigatória está ausente

## Requisitos Funcionais

1. A ferramenta deve carregar uma config base e mesclar sobrescritas específicas de ambiente sobre ela.
2. Variáveis de ambiente devem ter precedência sobre valores de arquivo para a mesma chave.
3. Chaves obrigatórias devem ser validadas no carregamento; uma chave obrigatória ausente deve abortar a inicialização com um erro claro.
4. Ela deve suportar ao menos três ambientes selecionados por uma única variável (ex.: `APP_ENV`).
5. Segredos devem vir de uma fonte separada (do ambiente ou de um arquivo de segredos ignorado pelo git).
6. A config resolvida deve ser exposta à app como um único objeto estruturado.
7. Ela nunca deve imprimir valores de segredos em logs ou mensagens de erro.

## Marcos Sugeridos

1. **Marco 1 — Carregar e mesclar:** Leia um arquivo base e um de ambiente, mesclando-os com precedência definida.
2. **Marco 2 — Sobrescritas por env e segredos:** Deixe variáveis de ambiente sobrescreverem valores de arquivo e carregue segredos separadamente.
3. **Marco 3 — Validar e falhar cedo:** Adicione um schema de chaves obrigatórias e aborte com uma mensagem útil diante de uma config ruim.

## Esboço de Dados e Interface

```text
Precedência (menor -> maior)
  padrões  <  config.base.yml  <  config.<env>.yml  <  variáveis de ambiente

Layout de arquivos
  config/
    base.yml
    development.yml
    production.yml
  .env            (ignorado pelo git, segredos)
  .gitignore      (exclui .env)

Schema de config (nomes de chave + tipos, não valores reais)
  app:
    port:        inteiro   (obrigatório)
    log_level:   enum[debug|info|warn|error]  (padrão info)
  database:
    host:        string    (obrigatório)
    password:    segredo    (obrigatório, só do env)
  feature_flags: map<string, boolean>

Resolução
  APP_ENV=production -> mescla base + production + env vars -> valida -> objeto Config
```

## Desafios Extras

- Adicione um comando `config validate` que checa uma config sem iniciar a app.
- Imprima um dump censurado da config resolvida (segredos mostrados como `***`).
- Suporte hot-reload: detecte um arquivo alterado e revalide sem reiniciar.
- Gere um `.env.example` listando toda chave obrigatória com valores de placeholder.

## Definição de Pronto

- [ ] Trocar `APP_ENV` carrega uma configuração diferente e corretamente mesclada.
- [ ] Uma variável de ambiente sobrescreve a mesma chave definida em um arquivo.
- [ ] Remover uma chave obrigatória aborta a inicialização com uma mensagem nomeando a chave ausente.
- [ ] Segredos não são commitados e nunca aparecem em logs ou no dump censurado.
- [ ] A aplicação recebe um objeto de config estruturado, não buscas espalhadas.

## Armadilhas Comuns

- Commitar um arquivo `.env` ou de segredos — adicione-o ao `.gitignore` antes do primeiro commit.
- Inverter a precedência, de forma que um padrão base silenciosamente sobrescreva uma variável de ambiente.
- Ler config ad hoc por todo o código em vez de resolver uma vez na inicialização.
- Logar o objeto de config inteiro no boot e vazar uma senha para os logs.

## Recursos

- [The Twelve-Factor App: Config](https://12factor.net/config) — por que a config pertence ao ambiente.
- [OWASP: Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) — lidando com segredos com segurança.
- [Especificação do TOML](https://toml.io/en/) — um formato de config projetado para ser inequívoco.
- [Especificação do YAML](https://yaml.org/spec/) — o formato e suas armadilhas.
