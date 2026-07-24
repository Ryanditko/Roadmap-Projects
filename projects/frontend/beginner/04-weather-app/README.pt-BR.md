# Aplicativo de Clima

> 🌐 [English](./README.md) · **Português**

**Domínio:** Frontend · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Construa um pequeno app onde o usuário digita uma cidade, e você busca e exibe o clima atual dela a partir de uma API pública. Este é o primeiro gostinho real da web assíncrona para iniciantes: uma requisição sai, o tempo passa, e algo — dados, um erro ou nada — volta. O trabalho interessante não é o layout, mas tratar os três estados que toda UI de rede precisa mostrar: carregando, sucesso e falha. Você vai ler JSON, mapeá-lo em uma visão limpa e garantir que o usuário nunca fique olhando para uma tela congelada sem saber o que aconteceu.

## Pré-requisitos

- Básico de JavaScript incluindo funções e objetos
- Promises e `async`/`await`, ou cadeias de `.then()`
- Como ler a documentação de uma API e inspecionar uma resposta JSON
- Uma chave de API gratuita de um provedor de clima (ex.: OpenWeather ou Open-Meteo, que não requer nenhuma)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Fazer requisições HTTP com `fetch` e analisar uma resposta JSON
- Modelar e renderizar estados explícitos de carregamento, sucesso e erro
- Tratar falhas graciosamente: cidade inválida, erro de rede, respostas não-200
- Manter uma chave de API fora do controle de versão e entender seus limites
- Transformar dados brutos da API em um formato pequeno e amigável à UI

## Requisitos Funcionais

1. O usuário pode buscar uma cidade pelo nome e disparar uma consulta de clima.
2. Enquanto a requisição está em andamento, um indicador de carregamento é exibido.
3. No sucesso, a UI mostra temperatura, uma descrição da condição e umidade.
4. Uma cidade desconhecida ou requisição falha mostra uma mensagem de erro clara e legível.
5. Envios repetidos e rápidos não devem deixar um resultado desatualizado na tela.
6. O campo de busca é acessível por teclado e enviável via Enter.
7. As unidades de temperatura são rotuladas explicitamente (°C ou °F).

## Marcos Sugeridos

1. **Marco 1 — Buscar e exibir:** Chame a API para uma cidade fixa e renderize o resultado.
2. **Marco 2 — Busca e estados:** Conecte o campo de entrada, adicione carregamento e tratamento de erro.
3. **Marco 3 — Robustez:** Trate entrada vazia, cidades inválidas e respostas fora de ordem.

## Esboço de Dados e Interface

```text
Estado da view (um de)
  { status: "idle" }
  { status: "loading" }
  { status: "success", data: Weather }
  { status: "error", message: string }

Weather (mapeado do JSON da API)
  city:        string
  tempC:       number
  condition:   string   ("Clouds", "Rain", ...)
  humidity:    number   (percentual)
  icon:        string   (código -> seu próprio ícone)

Layout
+--------------------------------------+
| [ buscar cidade....... ] [ Buscar ]  |
+--------------------------------------+
|  London           [ carregando... ]  |
|  18°C  Nublado   Umidade 72%         |
+--------------------------------------+
```

## Desafios Extras

- Adicione uma previsão de vários dias abaixo das condições atuais.
- Alterne entre Celsius e Fahrenheit sem refazer a busca.
- Use a API de Geolocalização para carregar o clima da localização atual do usuário.
- Faça cache do último resultado bem-sucedido para que um recarregamento mostre algo instantaneamente.

## Definição de Pronto

- [ ] Uma cidade válida mostra dados reais de clima com uma unidade rotulada.
- [ ] Os estados de carregamento, sucesso e erro renderizam distintamente.
- [ ] Uma cidade inexistente produz um erro amigável, não uma tela em branco ou travamento.
- [ ] A chave de API não é commitada no repositório.
- [ ] Enviar com Enter e clicar em Buscar se comportam de forma idêntica.

## Armadilhas Comuns

- Assumir que o `fetch` rejeita em erros HTTP — ele só rejeita em falha de rede; verifique `response.ok`.
- Esquecer de tratar o estado de carregamento, fazendo a UI parecer quebrada durante a requisição.
- Condições de corrida onde uma requisição anterior lenta sobrescreve um resultado mais novo.
- Fixar e commitar sua chave de API, e então atingir limites de taxa ou vazá-la.
- Renderizar o formato bruto do JSON da API diretamente, acoplando sua UI ao provedor.

## Recursos

- [MDN: Usando a Fetch API](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch) — requisições, respostas e verificação de erro.
- [MDN: Response.ok](https://developer.mozilla.org/en-US/docs/Web/API/Response/ok) — por que você deve verificar o status por conta própria.
- [Open-Meteo API](https://open-meteo.com/en/docs) — uma API de clima gratuita sem chave requerida.
- [web.dev: Loading states](https://web.dev/articles/optimize-cls) — comunicar progresso aos usuários.
