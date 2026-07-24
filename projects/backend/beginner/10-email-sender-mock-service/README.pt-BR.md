# Serviço Mock de Envio de E-mails

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 5–8 horas

## Visão Geral

Construa uma API de envio de e-mails que nunca envia de fato — ela valida cada requisição, a enfileira, a "processa" contra um transporte fictício e registra o resultado em um log ou arquivo. Isso modela os serviços de e-mail transacional (SendGrid, SES, Postmark) que ficam por trás de confirmações de cadastro e redefinições de senha, mas sem dependências externas nem o risco de enviar spam a caixas de entrada reais. O valor está na maquinaria ao redor: validação de requisição, uma fila de jobs, rastreamento de status e retries.

## Pré-requisitos

- Capacidade de construir endpoints HTTP que aceitam e retornam JSON ([API REST Simples de Tarefas](../01-simple-rest-api-task-management/) recomendada)
- Entendimento de persistência em arquivo ou memória (a [API de Notas](../04-notes-api-file-persistence/) ajuda)
- Familiaridade com a ideia de uma fila e processamento assíncrono
- Consciência das regras de formato de endereço de e-mail

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Projetar uma API que aceita um job e retorna imediatamente com um id de rastreamento
- Validar entrada estruturada (destinatário, assunto, corpo) e rejeitar requisições malformadas
- Modelar o ciclo de vida de um job com estados e transições explícitos
- Processar uma fila e simular sucesso e falha contra um transporte fictício
- Implementar lógica de retry com um número limitado de tentativas
- Renderizar uma mensagem a partir de um template com variáveis substituídas

## Requisitos Funcionais

1. O sistema deve aceitar uma requisição de e-mail com destinatário, assunto e corpo e retornar um id de job com 202.
2. O sistema deve rejeitar um endereço de destinatário inválido ou um campo ausente com 400.
3. Cada job deve ter um status que se move por `queued → sending → sent | failed`.
4. O sistema deve expor um endpoint para consultar o status atual de um job pelo id.
5. Em vez de enviar, o sistema deve registrar cada mensagem processada em um arquivo ou log com um timestamp.
6. Jobs falhos devem ser retentados até um máximo configurado e então marcados como permanentemente falhos.
7. Jobs e seus status devem persistir para que um reinício não perca trabalho enfileirado ou em andamento.

## Marcos Sugeridos

1. **Marco 1 — Aceitar e validar:** Receba uma requisição de e-mail, valide-a, armazene como `queued` e retorne um id de job.
2. **Marco 2 — Processar a fila:** Mova os jobs por seus estados, "entregando" a um transporte fictício e registrando o resultado.
3. **Marco 3 — Confiabilidade e templates:** Adicione retries limitados para falhas e suporte corpos de mensagem por template.

## Esboço de Dados e Interface

```text
Job
  id:        string
  to:        string   (e-mail validado)
  subject:   string
  body:      string
  status:    "queued" | "sending" | "sent" | "failed"
  attempts:  inteiro
  updatedAt: string ISO-8601

POST /emails         body: { to, subject, body | { template, vars } }
                     -> 202 { id, status: "queued" } | 400
GET  /emails/{id}    -> 200 { id, status, attempts } | 404

Transporte fictício: sucesso/falha aleatório (ou por regra); registra cada tentativa
Retry: em falha, reenfileira até attempts == MAX, então status=failed
```

## Desafios Extras

- Adicione templates nomeados com interpolação de variáveis e um erro de template-não-encontrado.
- Adicione envio agendado: aceite um timestamp `sendAt` e segure o job até lá.
- Adicione envio em lote: aceite muitos destinatários em uma requisição, rastreados como jobs individuais.
- Exponha métricas da fila (contagens por status) em um endpoint de estatísticas.

## Definição de Pronto

- [ ] Uma requisição válida retorna 202 com um id de job e cria um job `queued`.
- [ ] Destinatários inválidos ou campos ausentes retornam 400 antes de enfileirar.
- [ ] O status do job é consultável e reflete transições reais ao longo do ciclo de vida.
- [ ] Toda mensagem processada é registrada (logada/gravada), nunca de fato enviada por e-mail.
- [ ] Uma falha simulada retenta até o máximo e então se estabelece como `failed` — sem loop infinito.

## Armadilhas Comuns

- Fazer o "envio" de forma síncrona dentro do handler do POST, anulando o propósito de uma fila.
- Validar e-mail com uma checagem ingênua que rejeita endereços válidos ou aceita lixo óbvio.
- Retentar para sempre em falha sem um limite de tentativas, girando a fila infinitamente.
- Perder jobs em andamento no reinício porque o status só era mantido em memória.

## Recursos

- [Wikipedia: Fila de mensagens](https://en.wikipedia.org/wiki/Message_queue) — o conceito do qual sua fila é uma pequena instância.
- [RFC 5321: SMTP](https://datatracker.ietf.org/doc/html/rfc5321) — como a entrega real de e-mail funciona, para contexto.
- [regex101](https://regex101.com/) — construa e teste um padrão de validação de e-mail interativamente.
- [Referência da API SendGrid](https://www.twilio.com/docs/sendgrid/api-reference) — uma API real de e-mail transacional para inspirar sua interface.
