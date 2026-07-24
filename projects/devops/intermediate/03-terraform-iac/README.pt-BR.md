# Infraestrutura como Código (Terraform)

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Intermediário · **Tempo estimado:** 1–2 dias

## Visão Geral

Provisione uma peça de infraestrutura pequena, mas realista — digamos uma rede, uma instância de computação ou serviço de contêineres e um banco de dados gerenciado — inteiramente a partir de código Terraform, sem cliques em console de nuvem. O ponto não é memorizar nomes de recursos, mas internalizar o ciclo de IaC: escrever configuração, `plan` para pré-visualizar o diff, `apply` para convergir a realidade ao estado desejado e guardar o estado remotamente para que um time possa colaborar. Acima de tudo, sua configuração deve ser idempotente: rodar `apply` duas vezes seguidas sem mudanças de código não deve produzir nenhuma alteração.

## Pré-requisitos

- Uma conta de nuvem (AWS, GCP, Azure) ou um provider local como o Docker
- Terraform (ou OpenTofu) instalado e autenticado no provider
- Entendimento dos recursos que você pretende criar (rede, computação, armazenamento)
- Conforto com a linha de comando e leitura de um diff

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Declarar recursos e entender o ciclo de vida plan/apply/destroy
- Parametrizar a configuração com variáveis de entrada e expor resultados como outputs
- Fatorar recursos repetidos em um módulo reutilizável
- Guardar o estado remotamente com locking para que applies concorrentes não o corrompam
- Detectar e reconciliar drift entre o código e a infraestrutura real

## Requisitos Funcionais

1. Toda a infraestrutura deve ser definida em arquivos Terraform versionados — nenhuma mudança manual no console.
2. `terraform plan` deve mostrar uma pré-visualização precisa antes de qualquer mudança ser aplicada.
3. A configuração deve ser idempotente: um segundo `apply` sem mudança de código reporta zero alterações.
4. Valores específicos de ambiente (região, tamanhos, nomes) devem ser variáveis, não fixados no código.
5. Ao menos um grupo de recursos deve ser extraído em um módulo reutilizável.
6. O estado deve ser guardado em um backend remoto com locking habilitado.
7. `terraform destroy` deve remover de forma limpa tudo o que a configuração criou.

## Marcos Sugeridos

1. **Marco 1 — Primeiro apply:** Defina alguns recursos, rode plan/apply, verifique que existem.
2. **Marco 2 — Variáveis e módulos:** Parametrize com variáveis/outputs e extraia um módulo.
3. **Marco 3 — Estado remoto e drift:** Mova o estado para um backend remoto com locking; altere um recurso à mão e reconcilie o drift.

## Esboço de Dados e Interface

```text
Layout do repositório (só estrutura):
  main.tf          composição raiz, chama o(s) módulo(s)
  variables.tf     entradas tipadas (region, instance_size, env)
  outputs.tf       valores exportados (ip, db_endpoint)
  backend.tf       config de estado remoto (bucket + tabela de lock)
  modules/network/ módulo reutilizável (vpc, subnets, ...)

Ciclo de vida:
  escreve .tf  ->  terraform plan  ->  revisa diff  ->  terraform apply
                                                          |
  drift (mudança manual)  <──────  terraform plan mostra delta  <──┘

Estado: backend remoto, travado durante o apply (sem escritores concorrentes)
```

## Desafios Extras

- Adicione múltiplos ambientes via workspaces ou arquivos de variáveis separados.
- Rode `terraform validate` e `fmt` na CI e poste a saída do `plan` nos pull requests.
- Adicione uma verificação de política (OPA/Sentinel ou tflint) que bloqueie configs de recursos proibidos.
- Importe para o estado do Terraform um recurso existente criado manualmente.

## Definição de Pronto

- [ ] Um clone novo consegue `plan`/`apply` e reproduzir a infraestrutura do zero.
- [ ] Um segundo `apply` sem mudanças reporta "No changes."
- [ ] Nenhuma credencial ou valor de ambiente está fixado nos arquivos .tf.
- [ ] O estado vive em um backend remoto e é travado durante o apply.
- [ ] `terraform destroy` não deixa recursos órfãos para trás.

## Armadilhas Comuns

- Editar a infraestrutura à mão após o apply, causando drift que o próximo apply reverte ou combate silenciosamente.
- Commitar o `terraform.tfstate` local (com segredos) no git em vez de usar um backend remoto.
- Construir config não idempotente (ex.: timestamps em nomes) fazendo cada apply mostrar mudanças espúrias.
- Pular o `plan` e aplicar às cegas, para então descobrir que ele queria destruir o banco de dados.
- Fixar valores de região/conta, tornando a config impossível de reutilizar entre ambientes.

## Recursos

- [Documentação do Terraform](https://developer.hashicorp.com/terraform/docs) — linguagem, fluxo de trabalho e providers.
- [Estado do Terraform](https://developer.hashicorp.com/terraform/language/state) — por que o estado existe e como backends remotos funcionam.
- [Estrutura padrão de módulo](https://developer.hashicorp.com/terraform/language/modules/develop/structure) — como organizar um módulo reutilizável.
- [O que é Infraestrutura como Código?](https://developer.hashicorp.com/terraform/intro) — o conceito e seus benefícios.
</content>
