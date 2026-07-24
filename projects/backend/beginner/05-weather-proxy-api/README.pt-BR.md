# API Proxy de Clima

> 🌐 [English](./README.md) · **Português**

**Domínio:** Backend · **Nível:** Iniciante · **Tempo estimado:** 5–8 horas

## Visão Geral

Construa uma API que fica na frente de um serviço público de clima, encaminhando requisições, simplificando a resposta e fazendo cache dos resultados para não sobrecarregar o provedor upstream. Este é seu primeiro contato com ser um *cliente* HTTP além de servidor — o padrão por trás de todo backend-for-frontend e API gateway. Um usuário pede a previsão de uma cidade ao seu serviço; seu serviço pergunta ao provedor, reduz o payload ao que importa e responde, idealmente do cache na segunda vez.

## Pré-requisitos

- Capacidade de construir endpoints HTTP e retornar JSON ([Servidor de API JSON Estático](../09-static-json-api-server/) é um precursor mais leve)
- Entendimento de variáveis de ambiente para segredos
- Uma chave de API gratuita de um provedor de clima (OpenWeatherMap, WeatherAPI ou Open-Meteo, que não exige chave)
- Familiaridade em fazer requisições HTTP de saída na sua linguagem

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Fazer requisições HTTP de saída a partir de um servidor e tratar suas respostas
- Manter uma chave de API fora do código-fonte usando configuração/variáveis de ambiente
- Transformar um payload verboso do upstream em uma resposta limpa e mínima
- Fazer cache de respostas com um tempo de vida (TTL) para reduzir latência e chamadas ao upstream
- Tratar falhas do upstream — timeouts, 404s, rate limits — sem expô-las cruas
- Traduzir condições de erro do upstream em códigos de status sensatos para seus clientes

## Requisitos Funcionais

1. O sistema deve expor um endpoint que aceita o nome de uma cidade e retorna o clima atual.
2. O sistema deve ler a chave da API upstream da configuração, nunca fixada no código.
3. O sistema deve transformar a resposta do upstream em um formato simplificado e documentado.
4. O sistema deve fazer cache do resultado de cada cidade por um TTL configurável e servir os acertos de cache sem chamar o upstream.
5. Uma cidade desconhecida deve retornar 404, distinto de uma indisponibilidade do upstream.
6. O sistema deve aplicar um timeout à chamada upstream e degradar graciosamente se ele for excedido.
7. O sistema nunca deve vazar a chave da API em respostas, logs ou mensagens de erro.

## Marcos Sugeridos

1. **Marco 1 — Repasse:** Chame o provedor upstream para uma cidade e retorne sua resposta crua.
2. **Marco 2 — Transformar e blindar:** Mapeie o payload para seu próprio formato e trate os casos de cidade desconhecida e timeout.
3. **Marco 3 — Cache:** Adicione cache em memória com TTL e confirme que os acertos de cache pulam a chamada upstream.

## Esboço de Dados e Interface

```text
GET /weather?city=Lisbon
  -> 200 {
       "city": "Lisbon",
       "tempC": 21.4,
       "condition": "Clear",
       "cachedAt": "ISO-8601",
       "source": "cache" | "upstream"
     }
  -> 404  { "error": "city not found" }
  -> 502  { "error": "weather provider unavailable" }
  -> 504  { "error": "weather provider timed out" }

Entrada de cache: chave = cidade normalizada, valor = { payload, expiresAt }
```

## Desafios Extras

- Adicione um endpoint de previsão de vários dias ao lado das condições atuais.
- Adicione retry com backoff exponencial para erros transitórios do upstream.
- Suporte unidades via um parâmetro `?units=metric|imperial`.
- Exponha métricas de cache (contagens de acerto/erro) em um endpoint de debug.

## Definição de Pronto

- [ ] Uma cidade válida retorna dados de clima simplificados com os campos corretos.
- [ ] A chave da API é carregada da configuração e nunca aparece em nenhuma saída.
- [ ] Uma requisição repetida dentro do TTL é servida do cache e não faz chamada upstream (verificável).
- [ ] Cidades desconhecidas retornam 404 e falhas do upstream retornam 502/504, não 500.
- [ ] Um timeout do upstream não trava seu endpoint indefinidamente.

## Armadilhas Comuns

- Commitar a chave da API ou devolvê-la em um erro — trate-a como segredo desde o primeiro dia.
- Repassar o corpo de erro cru do upstream, acoplando seus clientes a um terceiro.
- Fazer cache pela string crua (não normalizada) da cidade, de modo que "Lisbon" e "lisbon" não se encontram.
- Nenhum timeout na chamada de saída, deixando um provedor lento congelar seu serviço inteiro.

## Recursos

- [API Open-Meteo](https://open-meteo.com/en/docs) — uma API de clima gratuita que não exige chave, ótima para começar.
- [Documentação da API OpenWeatherMap](https://openweathermap.org/api) — um provedor amplamente usado com camada gratuita.
- [MDN: Cache](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Caching) — conceitos de cache HTTP, úteis mesmo para TTLs em memória.
- [The Twelve-Factor App: Config](https://12factor.net/pt_br/config) — por que segredos pertencem ao ambiente, não ao código.
