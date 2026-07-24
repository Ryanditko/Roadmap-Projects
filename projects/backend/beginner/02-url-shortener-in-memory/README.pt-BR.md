# Encurtador de URLs (Em Memória)

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 4–7 horas

## Visão Geral

Construa um serviço como o Bitly ou o TinyURL que transforma um link longo em um código curto e redireciona os visitantes de volta ao original quando o acessam. Tudo vive em um mapa em memória, então reseta ao reiniciar — o que mantém o foco nas partes interessantes: gerar códigos curtos livres de colisão e emitir redirecionamentos HTTP corretos. É um projeto pequeno que ensina discretamente codificação, roteamento e a diferença entre um 301 e um 302.

## Pré-requisitos

- Conforto para construir endpoints HTTP básicos ([API REST Simples de Tarefas](../01-simple-rest-api-task-management/) é um bom aquecimento)
- Entender do que uma URL é feita (esquema, host, caminho)
- Um framework web à sua escolha (Node, Python ou Go)
- Familiaridade com mapas/dicionários como armazenamento chave-valor

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Gerar códigos curtos e únicos e raciocinar sobre a probabilidade de colisão
- Implementar redirecionamentos HTTP e escolher entre 301 (permanente) e 302 (temporário)
- Validar e normalizar URLs fornecidas pelo usuário antes de armazená-las
- Usar uma estrutura chave-valor para busca bidirecional (código ↔ URL)
- Rastrear e expor uma análise simples de uso por link

## Requisitos Funcionais

1. O sistema deve aceitar uma URL longa e retornar um código curto único.
2. O sistema deve rejeitar entradas que não sejam URLs `http`/`https` sintaticamente válidas com 400.
3. Visitar um código curto deve redirecionar o cliente à URL original com um status 3xx apropriado.
4. Requisitar um código desconhecido deve retornar 404.
5. O sistema deve garantir que duas URLs longas diferentes nunca recebam o mesmo código.
6. O sistema deve contar quantas vezes cada código curto foi visitado.
7. O sistema deve expor um endpoint para recuperar a URL original e a contagem de acessos sem redirecionar.

## Marcos Sugeridos

1. **Marco 1 — Encurtar e armazenar:** Aceite uma URL, gere um código, armazene o mapeamento e retorne o link curto.
2. **Marco 2 — Redirecionar:** Resolva um código para sua URL e emita o redirecionamento, tratando o caso 404.
3. **Marco 3 — Validação e análise:** Valide URLs na entrada e rastreie contagens de visitas por código.

## Esboço de Dados e Interface

```text
Link
  code:      string   (ex.: "aZ3kQ")
  longUrl:   string
  hits:      inteiro
  createdAt: string ISO-8601

POST /shorten        body: { "url": "https://..." }
                     -> 201 { "code": "aZ3kQ", "shortUrl": ".../aZ3kQ" }
GET  /{code}         -> 302 Location: <longUrl>  | 404
GET  /api/{code}     -> 200 { longUrl, hits, createdAt } | 404

Opções de geração de código: base62 aleatório, contador incremental -> base62
```

## Desafios Extras

- Permita um alias personalizado escolhido pelo usuário, rejeitando um que já esteja em uso.
- Adicione expiração opcional para que um código pare de resolver após um tempo definido.
- Retorne o link já encurtado para uma URL que já foi encurtada, em vez de um novo código.
- Adicione uma página de estatísticas listando os links mais acessados.

## Definição de Pronto

- [ ] Um link encurtado redireciona exatamente para a URL original, incluindo a query string.
- [ ] URLs inválidas ou não-HTTP são rejeitadas com 400 antes de qualquer código ser gerado.
- [ ] Códigos desconhecidos retornam 404, não um redirecionamento para lugar nenhum.
- [ ] Códigos gerados são seguros para URL e verificados como únicos no armazenamento.
- [ ] As contagens de acesso incrementam no redirecionamento e ficam visíveis pelo endpoint de estatísticas.

## Armadilhas Comuns

- Armazenar a URL sem validá-la e depois redirecionar para `javascript:` ou um alvo malformado.
- Usar um hash da URL como código sem tratar a (rara) colisão — decida uma estratégia e teste-a.
- Confundir 301 e 302: navegadores fazem cache agressivo do 301, então um redirecionamento permanente errado é difícil de desfazer.
- Esquecer de preservar a query string ou o fragmento da URL original no redirecionamento.

## Recursos

- [MDN: Redirecionamentos em HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Redirections) — 301 vs 302 vs 307/308 explicados.
- [RFC 3986: Sintaxe Genérica de URI](https://datatracker.ietf.org/doc/html/rfc3986) — a autoridade sobre como é uma URL válida.
- [Wikipedia: Base62](https://en.wikipedia.org/wiki/Base62) — a codificação comum para códigos curtos e legíveis.
- [MDN: API URL](https://developer.mozilla.org/pt-BR/docs/Web/API/URL) — uma forma nativa de analisar e validar URLs.
