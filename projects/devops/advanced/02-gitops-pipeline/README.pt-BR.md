# Pipeline GitOps

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Avançado · **Tempo estimado:** 1–2 semanas

## Visão Geral

Faça do Git a única fonte de verdade sobre o que roda nos seus clusters e deixe um reconciliador — não um humano rodando `kubectl apply` — convergir a realidade ao estado declarado. Você vai construir um pipeline em que um merge em um repositório dispara um deploy automático e auditável, em que qualquer mudança manual no cluster é detectada como drift e revertida, e em que rollbacks são apenas `git revert`. As partes difíceis são justamente as que as pessoas pulam nas demos: manter segredos fora do Git em texto puro, estruturar repositórios para que múltiplos ambientes não se atropelem, adicionar portões de aprovação sem transformar GitOps de volta em ClickOps, e provar que "o repo descreve o cluster" é de fato verdade. Bem feito, seu histórico de deploy e seu histórico de Git viram a mesma coisa.

## Pré-requisitos

- Conhecimento funcional de Kubernetes e conforto com manifests declarativos
- Experiência com branches Git, pull requests e fluxos de revisão
- Familiaridade com Helm ou Kustomize para overlays de ambiente
- Entendimento básico de conceitos de gestão de segredos (criptografia em repouso, KMS)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Modelar o estado desejado no Git e ter um operador reconciliando-o continuamente
- Detectar e remediar drift de configuração automaticamente
- Estruturar repositórios para múltiplos ambientes e promoção progressiva
- Tratar segredos com segurança em um fluxo dirigido por Git
- Reverter um deploy puramente pelo histórico do Git e observar o reconciliador recuperar

## Requisitos Funcionais

1. Um merge no branch-alvo deve disparar um deploy sem passo manual de `kubectl`.
2. O operador deve reconciliar continuamente o estado do cluster com o repositório.
3. Mudanças manuais em recursos gerenciados devem ser detectadas como drift e reportadas ou revertidas.
4. Segredos nunca devem ser commitados em texto puro; devem ser criptografados ou referenciados externamente.
5. A promoção de um ambiente para o próximo deve ser uma ação explícita e revisável.
6. Um `git revert` de um commit de deploy deve reverter o cluster ao estado anterior.
7. Toda mudança no cluster deve ser rastreável até um commit, um autor e uma aprovação.

## Marcos Sugeridos

1. **Marco 1 — Reconciliar a partir do Git:** Instale um operador (Argo CD ou Flux), aponte-o para um repo e observe-o fazer deploy e autocorreção.
2. **Marco 2 — Drift e rollback:** Faça uma mudança manual e confirme a detecção de drift; depois reverta via `git revert`.
3. **Marco 3 — Multi-ambiente:** Estruture dev/staging/prod com overlays e adicione um fluxo de promoção com aprovação.
4. **Marco 4 — Segredos e política:** Adicione segredos selados/criptografados e uma verificação de política que bloqueia manifests não conformes antes do merge.

## Esboço de Dados e Interface

```text
   dev ──PR──▶ repo Git (fonte de verdade)
                    │  estado desejado (manifests / Helm / Kustomize)
                    ▼
              ┌────────────────┐   loop de reconciliação (a cada N s)
              │ Operador GitOps│──────────────┐
              │  (Argo / Flux) │              │
              └───────┬────────┘              ▼
                      │ apply / prune   ┌─────────────┐
                      └────────────────▶│  Cluster    │
                                        │ estado vivo │
                      drift? ◀──compara─┤             │
                                        └─────────────┘

Layout do repo (uma opção):
  apps/<nome>/base/          manifests compartilhados
  apps/<nome>/overlays/dev/  patches por ambiente
  apps/<nome>/overlays/prod/
  secrets/  -> apenas SealedSecrets / criptografado com SOPS

Metas não-funcionais:
  latência de sync  < 3 min do merge à convergência
  MTTR de drift     auto-revertido ou alertado < 5 min
  auditabilidade    100% das mudanças rastreáveis a um commit
```

## Desafios Extras

- Adicione entrega progressiva (Argo Rollouts / Flagger) para que promoções sejam canário automaticamente.
- Conecte política como código (OPA/Gatekeeper ou Kyverno) como portão de merge e portão de admissão.
- Suporte múltiplos clusters a partir de um repo de controle com direcionamento por cluster.
- Adicione notificações e um dashboard de deploy para que o time veja o status de sync sem abrir a CLI.

## Definição de Pronto

- [ ] Um merge faz deploy automaticamente; nenhum humano roda `kubectl apply` no caminho feliz.
- [ ] Drift manual é detectado e revertido ou claramente alertado.
- [ ] Segredos são criptografados ou externalizados — nada sensível fica em texto puro no Git.
- [ ] Múltiplos ambientes são promovidos por um fluxo explícito e revisável.
- [ ] Um `git revert` comprovadamente reverte o cluster.

## Armadilhas Comuns

- Commitar segredos em texto puro "temporariamente" — eles vivem para sempre no histórico.
- Um repo gigante sem separação de ambientes, de modo que uma mudança de dev pode chegar à prod.
- Desabilitar auto-sync/prune "por segurança", o que silenciosamente reintroduz drift manual.
- Tratar o operador como esqueça-e-pronto e nunca monitorar syncs que falharam.
- Portões de aprovação tão pesados que as pessoas contornam o GitOps por completo para correções "urgentes".

## Recursos

- [Documentação do Argo CD](https://argo-cd.readthedocs.io/) — GitOps declarativo para Kubernetes.
- [Documentação do Flux](https://fluxcd.io/flux/) — o toolkit GitOps da CNCF.
- [Princípios OpenGitOps](https://opengitops.dev/) — os quatro princípios centrais de GitOps.
- [SOPS](https://github.com/getsops/sops) — criptografando segredos para armazenamento seguro no Git.
