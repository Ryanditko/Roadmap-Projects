# Configuração Básica de Nginx

> 🌐 [English](./README.md) · **Português**

**Domínio:** DevOps · **Nível:** Iniciante · **Tempo estimado:** 3–6 horas

## Visão Geral

Suba o Nginx e coloque-o na frente de uma aplicação, primeiro como um servidor web simples e depois como um reverse proxy que encaminha requisições à sua app rodando atrás dele. Esta é uma das formas mais comuns em produção: o Nginx termina o TLS, serve arquivos estáticos rapidamente e entrega requisições dinâmicas a um upstream. Você vai escrever um server block, habilitar HTTPS, adicionar alguns headers sensatos e provar que uma requisição à porta 443 alcança sua app e retorna corretamente. O objetivo é uma config que você entende linha por linha — não um trecho copiado que você tem medo de tocar.

## Pré-requisitos

- Uma aplicação em execução escutando em uma porta local (qualquer linguagem)
- Uma máquina (ou VM/contêiner) onde você pode instalar e rodar o Nginx
- Entendimento básico de HTTP, portas e DNS/hostnames
- Um certificado TLS (autoassinado para local, ou Let's Encrypt para um domínio real)

## Objetivos de Aprendizado

Ao final, você deve ser capaz de:

- Configurar um server block que serve um site em um hostname e porta
- Configurar o Nginx como reverse proxy para uma aplicação upstream
- Habilitar HTTPS e redirecionar HTTP para HTTPS
- Adicionar headers de segurança e cache e compressão gzip
- Ler os logs de acesso e de erro para depurar uma config problemática

## Requisitos Funcionais

1. O Nginx deve servir uma resposta para um hostname definido na porta 80.
2. Ele deve fazer reverse-proxy de requisições para uma app upstream e retornar a resposta da app.
3. O HTTPS deve estar habilitado na porta 443 com um certificado válido (ou autoassinado).
4. Requisições HTTP simples devem redirecionar para HTTPS.
5. A config deve definir headers encaminhados (host, IP real, protocolo) para o upstream.
6. Ativos estáticos devem ser servidos diretamente pelo Nginx com um header de cache.
7. `nginx -t` deve passar e um reload deve aplicar mudanças sem derrubar conexões.

## Marcos Sugeridos

1. **Marco 1 — Servir:** Escreva um server block que retorna uma página para seu hostname via HTTP.
2. **Marco 2 — Proxy e TLS:** Faça reverse-proxy para a app, habilite HTTPS e redirecione HTTP para HTTPS.
3. **Marco 3 — Endurecer e ajustar:** Adicione headers de segurança, gzip, cache de estáticos e valide com `nginx -t`.

## Esboço de Dados e Interface

```text
Layout de arquivos
  /etc/nginx/nginx.conf
  /etc/nginx/sites-available/app.conf   -> symlink em sites-enabled/

Estrutura do server block (diretivas-chave, não a config completa)
  server (porta 80):
    server_name <host>
    return 301 https://$host$request_uri     # redireciona para HTTPS
  server (porta 443, ssl):
    server_name <host>
    ssl_certificate / ssl_certificate_key
    location /static/  -> root <path>; header cache-control
    location /         -> proxy_pass http://upstream_app
                          proxy_set_header Host / X-Real-IP / X-Forwarded-Proto
  upstream upstream_app:
    server 127.0.0.1:8080

Validar e aplicar
  nginx -t          # testa a config
  nginx -s reload   # reload gracioso
```

## Desafios Extras

- Adicione um segundo servidor upstream e balanceie entre eles (round-robin).
- Adicione rate limiting básico a uma rota de login ou de API.
- Automatize a emissão e renovação de certificados com Certbot / Let's Encrypt.
- Adicione uma location `/healthz` que retorna 200 sem chegar ao upstream.

## Definição de Pronto

- [ ] Um navegador alcançando o hostname via HTTPS recebe a resposta da app.
- [ ] Requisições HTTP redirecionam para HTTPS.
- [ ] Arquivos estáticos são servidos pelo Nginx com um header de cache, não via proxy.
- [ ] Headers encaminhados chegam ao upstream (verifique que a app enxerga o protocolo/IP real do cliente).
- [ ] `nginx -t` passa e um reload aplica mudanças sem requisições derrubadas.

## Armadilhas Comuns

- Esquecer `proxy_set_header Host $host`, fazendo o upstream ver o host errado e gerar links quebrados.
- Deixar um cert autoassinado em produção e treinar usuários a ignorar avisos de TLS.
- Editar `nginx.conf` e recarregar sem rodar `nginx -t` antes, derrubando o site em um erro de digitação.
- Servir arquivos estáticos pelo proxy, desperdiçando o upstream em trabalho que o Nginx faz mais rápido.

## Recursos

- [Nginx: Guia do Iniciante](https://nginx.org/en/docs/beginners_guide.html) — o ponto de partida oficial.
- [Nginx: Guia de reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) — proxy_pass e headers encaminhados.
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) — configurações TLS seguras e atuais para Nginx.
- [Let's Encrypt / Certbot](https://certbot.eff.org/) — certificados gratuitos e renovação automática.
