# Implementação de Service Mesh

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Mova as preocupações transversais da comunicação entre microsserviços — criptografia, retentativas, timeouts, divisão de tráfego e observabilidade — para fora de cada aplicação e para uma camada de infraestrutura dedicada. Você vai implantar um service mesh que injeta um proxy sidecar (ou um proxy por nó) ao lado dos seus serviços, de modo que o tráfego entre eles seja interceptado e governado por política em vez de por código espalhado entre times. O trabalho avançado não é "rodar o instalador"; é decidir o que pertence ao mesh versus ao app, habilitar TLS mútuo sem quebrar o tráfego existente, escrever política de autorização que seja negar-por-padrão, e prestar atenção honesta à latência e ao overhead de recursos que um proxy em cada salto introduz. Bem feito, segurança e confiabilidade viram propriedades da plataforma, não tarefa de casa de cada serviço.

## Pré-requisitos

- Um cluster Kubernetes rodando pelo menos dois serviços que chamam um ao outro
- Entendimento de TLS, certificados e autenticação mútua
- Familiaridade com conceitos L7: retentativas, timeouts, circuit breaking
- Observabilidade básica para medir o overhead que o mesh adiciona

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implantar um service mesh e entender a divisão sidecar/data-plane vs. control-plane
- Habilitar TLS mútuo entre serviços sem derrubar tráfego
- Escrever políticas de autorização negar-por-padrão entre cargas de trabalho
- Configurar gestão de tráfego: retentativas, timeouts, circuit breaking e divisão
- Quantificar e raciocinar sobre o overhead de latência e recursos do mesh

## Requisitos Funcionais

1. O tráfego entre serviços no mesh deve ser criptografado com TLS mútuo automaticamente.
2. A autorização serviço-a-serviço deve ser negar-por-padrão com regras de permissão explícitas.
3. O mesh deve impor retentativas, timeouts e circuit breaking configuráveis.
4. A divisão de tráfego entre versões de serviço deve ser controlável sem mudanças no app.
5. O mesh deve emitir telemetria L7 (latência por rota, taxa de erro) para observabilidade.
6. Habilitar o mesh não deve exigir reescrever o código de rede da aplicação.
7. A latência adicionada e o overhead de recursos devem ser medidos e documentados.

## Marcos Sugeridos

1. **Marco 1 — Mesh e sidecars:** Instale o mesh e injete sidecars para dois serviços que se comunicam.
2. **Marco 2 — mTLS e authz:** Ligue o TLS mútuo e escreva política de autorização negar-por-padrão.
3. **Marco 3 — Gestão de tráfego:** Adicione retentativas, timeouts, circuit breaking e divisão baseada em versão.
4. **Marco 4 — Observabilidade e custo:** Conecte a telemetria L7 e meça o overhead do mesh contra um baseline.

## Esboço de Dados e Interface

```text
                 ┌───────────────────────┐
                 │   Control Plane        │  (config, certs, política)
                 │  (istiod / linkerd)    │
                 └───────────┬───────────┘
           empurra config/certs│
        ┌──────────────┐      │      ┌──────────────┐
        │  Serviço A   │      │      │  Serviço B   │
        │ ┌──────────┐ │      │      │ ┌──────────┐ │
   ───▶ │ │  proxy   │◀┼──mTLS┼──────┼▶│  proxy   │ │ ───▶
        │ └────┬─────┘ │      │      │ └────┬─────┘ │
        │   app A      │      │      │   app B      │
        └──────────────┘      │      └──────────────┘
           data plane (sidecars impõem política em cada salto)

Exemplo de política (conceitual):
  authz: DENY por padrão
         allow  A -> B  na rota /orders  método GET
  tráfego: B retries=2 timeout=2s circuit-break em 50% 5xx
           split B: v1=90% v2=10%

Metas não-funcionais:
  latência p99 adicionada  medida por salto, mantida no orçamento
  overhead de proxy        CPU/mem por sidecar quantificado
  segurança                0 tráfego serviço-a-serviço em texto puro
```

## Desafios Extras

- Compare um mesh com sidecar com um modo sem sidecar/ambient e meça a diferença de overhead.
- Estenda a autorização para usar identidade de requisição (JWT), não apenas identidade de carga.
- Adicione injeção de falhas na camada do mesh para combinar com uma prática de chaos engineering.
- Federe o mesh entre dois clusters para mTLS e descoberta entre clusters.

## Definição de Pronto

- [ ] O tráfego serviço-a-serviço é criptografado com TLS mútuo sem mudanças no código do app.
- [ ] A autorização é negar-por-padrão com regras de permissão explícitas e testadas.
- [ ] Retentativas, timeouts e circuit breaking são impostos e disparam de forma demonstrável.
- [ ] O tráfego pode ser dividido entre versões puramente via config do mesh.
- [ ] A latência e o overhead de recursos do mesh são medidos e documentados contra um baseline.

## Armadilhas Comuns

- Ligar mTLS estrito globalmente de uma vez e cortar serviços que ainda não estão no mesh.
- Deixar a autorização em permitir-tudo, de modo que o mesh adiciona criptografia mas nenhum controle de acesso real.
- Ignorar o overhead do proxy até que caminhos sensíveis à latência regridam em produção.
- Configurar retentativas agressivas que amplificam carga e transformam uma oscilação em uma tempestade de retentativas.
- Colocar no mesh lógica que pertence ao app (ou vice-versa), borrando a titularidade.

## Recursos

- [Documentação do Istio](https://istio.io/latest/docs/) — um service mesh amplamente usado com política de tráfego rica.
- [Documentação do Linkerd](https://linkerd.io/2/overview/) — um mesh leve e focado em segurança.
- [SMI: Service Mesh Interface](https://smi-spec.io/) — uma especificação de API de mesh neutra em relação a fornecedor.
- [Google SRE Book: Handling Overload](https://sre.google/sre-book/handling-overload/) — retentativas, descarte de carga e circuit breaking feitos com segurança.
