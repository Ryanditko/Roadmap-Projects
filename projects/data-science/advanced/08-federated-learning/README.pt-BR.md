# Simulação de Aprendizado Federado

> 🌐 [English](./README.md) · **Português**

**Domínio:** Data Science · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Simule um sistema onde muitos clientes treinam um modelo compartilhado juntos sem nunca enviar seus dados brutos para um servidor central. Cada cliente treina localmente com seus próprios dados, envia apenas atualizações do modelo, e um servidor as agrega em um novo modelo global. Parece simples até as realidades baterem: os clientes têm dados não-IID, alguns são lentos ou caem, a comunicação é cara e as próprias atualizações podem vazar informação. Neste projeto você implementa o loop FedAvg e depois enfrenta heterogeneidade, stragglers, custo de comunicação e privacidade — as quatro coisas que separam um brinquedo de um sistema federado real.

## Pré-requisitos

- Entendimento sólido de treino baseado em gradiente e média de modelos
- Experiência com PyTorch ou TensorFlow para o passo de treino local
- Familiaridade com o conceito de privacidade diferencial
- Conforto para simular processos distribuídos (threads, processos ou um framework)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Implementar o loop de treino de média federada (FedAvg) entre clientes simulados
- Tratar partições de dados não-IID e medir seu efeito na convergência
- Lidar com stragglers e dropout de clientes sem travar uma rodada
- Reduzir o custo de comunicação via compressão de atualizações ou menos rodadas
- Adicionar um mecanismo de privacidade (privacidade diferencial ou agregação segura) e quantificar seu custo

## Requisitos Funcionais

1. A simulação deve rodar múltiplos clientes, cada um treinando localmente em uma partição de dados privada.
2. Um servidor deve agregar as atualizações dos clientes em um modelo global a cada rodada de comunicação.
3. As partições de dados devem suportar distribuições IID e não-IID entre clientes.
4. O sistema deve tolerar stragglers e clientes que caem dentro de uma rodada.
5. Deve implementar uma técnica de redução de comunicação (compressão ou épocas locais).
6. Deve oferecer um mecanismo de privacidade (ex.: DP-SGD) que possa ser ligado e medido.
7. Deve rastrear e reportar a convergência do modelo global ao longo das rodadas.

## Requisitos Não Funcionais

- **Convergência:** o modelo global deve convergir sob dados não-IID dentro de um orçamento de rodadas documentado.
- **Eficiência de comunicação:** reporte bytes-por-rodada e comunicação total versus uma baseline centralizada.
- **Privacidade:** quando a DP está ativa, reporte o orçamento de privacidade (epsilon) e seu custo de acurácia.
- **Robustez:** uma rodada deve completar mesmo que uma fração configurável de clientes caia.

## Marcos Sugeridos

1. **Marco 1 — Núcleo FedAvg:** Simule clientes, treino local e média no servidor em dados IID.
2. **Marco 2 — Heterogeneidade:** Introduza partições não-IID e meça o impacto na convergência.
3. **Marco 3 — Robustez e comunicação:** Trate stragglers/dropout e adicione compressão de atualizações.
4. **Marco 4 — Privacidade:** Adicione DP-SGD, varra o epsilon e quantifique o tradeoff privacidade–acurácia.

## Esboço de Dados e Interface

```text
              +-------------------- Servidor ------------------+
              |  modelo global w_t                             |
              |    broadcast w_t                               |
              +-----+-----------+-----------+------------------+
                    |           |           |
             +------v--+  +-----v---+  +----v----+  ... (K clientes)
             | Cliente1|  | Cliente2|  | ClienteK|
             | treino  |  | treino  |  | treino  |   dados privados (IID ou não-IID)
             | local   |  | local   |  | local   |
             | -> dw_1 |  | -> dw_2 |  | -> dw_K |   (+ ruído DP, compressão)
             +----+----+  +----+----+  +----+----+
                  |            |            |
                  +------ agrega (média ponderada) ------+
                                |
                          w_{t+1} = soma_k (n_k/n) * (w_t + dw_k)

 A rodada descarta clientes após um prazo (tratamento de straggler)
 Rastreie: acc global por rodada, bytes/rodada, epsilon (se DP ativo)
```

## Desafios Extras

- Adicione agregação segura para que o servidor nunca veja atualizações individuais em claro.
- Adicione personalização: uma base compartilhada com heads ajustadas por cliente.
- Simule um cliente adversário (envenenamento de modelo) e adicione um agregador robusto (ex.: média aparada).
- Compare FedAvg contra FedProx sob forte heterogeneidade.

## Definição de Pronto

- [ ] FedAvg converge em dados IID entre clientes simulados.
- [ ] Partições não-IID são suportadas e seu impacto na convergência é medido.
- [ ] As rodadas completam apesar de uma fração configurável de clientes lentos/caídos.
- [ ] Uma técnica de redução de comunicação é implementada e sua economia reportada.
- [ ] A DP pode ser ativada, com o epsilon e o custo de acurácia reportados.

## Armadilhas Comuns

- Testar apenas em dados IID e nunca ver os problemas de convergência que definem o aprendizado federado.
- Esperar sincronamente por cada cliente, fazendo um straggler travar a rodada inteira.
- Adicionar privacidade diferencial sem rastrear o epsilon real, tornando "privado" sem sentido.
- Fazer média das atualizações sem ponderação quando os clientes têm quantidades de dados muito diferentes.

## Recursos

- [Communication-Efficient Learning of Deep Networks from Decentralized Data (McMahan et al., 2017)](https://arxiv.org/abs/1602.05629) — o paper do FedAvg.
- [Advances and Open Problems in Federated Learning (Kairouz et al., 2019)](https://arxiv.org/abs/1912.04977) — o survey abrangente.
- [Documentação do TensorFlow Federated](https://www.tensorflow.org/federated) — um framework para computação federada.
- [Deep Learning with Differential Privacy (Abadi et al., 2016)](https://arxiv.org/abs/1607.00133) — DP-SGD e o orçamento de privacidade.
