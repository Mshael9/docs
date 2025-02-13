---
title: Configurando o OpenID Connect no HashiCorp Vault
shortTitle: Configurando o OpenID Connect no HashiCorp Vault
intro: Use o OpenID Connect nos seus fluxos de trabalho para efetuar a autenticação com o HashiCorp Vault.
miniTocMaxHeadingLevel: 3
versions:
  fpt: '*'
  ghae: issue-4856
  ghec: '*'
  ghes: '>=3.5'
type: tutorial
topics:
  - Security
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Visão Geral

O OpenID Connect (OIDC) permite aos seus fluxos de trabalho de {% data variables.product.prodname_actions %} efetuar a autenticação com um HashiCorp Vault para recuperar segredos.

Este guia fornece uma visão geral sobre como configurar o cofre HashiCorp para confiar no OIDC de {% data variables.product.prodname_dotcom %} como uma identidade federada e mostra como usar essa configuração na ação [hashicorp/vault-action](https://github.com/hashicorp/vault-action) para recuperar segredos do cofre HashiCorp.

## Pré-requisitos

{% data reusables.actions.oidc-link-to-intro %}

{% data reusables.actions.oidc-security-notice %}

## Adicionando o provedor de identidade ao HashiCorp Vault

Para usar OIDC com oHashiCorp Vault, você deverá adicionar uma configuração de confiança ao provedor do OIDC de {% data variables.product.prodname_dotcom %}. Para obter mais informações, consulte a [documentação](https://www.vaultproject.io/docs/auth/jwt) do HashiCorp Vault.

Configure o cofre para aceitar tokens web do JSON (JWT) para a autenticação:
- For the `oidc_discovery_url`, use {% ifversion ghes %}`https://HOSTNAME/_services/token`{% else %}`https://token.actions.githubusercontent.com`{% endif %}
- For `bound_issuer`, use {% ifversion ghes %}`https://HOSTNAME/_services/token`{% else %}`https://token.actions.githubusercontent.com`{% endif %}
- Certifique-se de que `bound_subject` esteja corretamente definido para seus requisitos de segurança. Para obter mais informações, consulte ["Configurar a confiança do OIDC com a nuvem"](/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect#configuring-the-oidc-trust-with-the-cloud) e [`hashicorp/vault-action`](https://github.com/hashicorp/vault-action).

## Atualizar o seu fluxo de trabalho de {% data variables.product.prodname_actions %}

Para atualizar seus fluxos de trabalho para o OIDC, você deverá fazer duas alterações no seu YAML:
1. Adicionar configurações de permissões para o token.
2. Use a ação [`hashicorp/vault-ação`](https://github.com/hashicorp/vault-action) para trocar o token do OIDC (JWT) por um token de acesso na nuvem.


Para adicionar a integração do OIDC a seus fluxos de trabalho que lhes permitem acessar os segredos no Vault, você deverá adicionar as seguintes alterações de código:

- Conceder permissão para obter o token do provedor do OIDC de {% data variables.product.prodname_dotcom %}:
  - O fluxo de trabalho precisa de configurações de `permissions:` com o valor `id-token` definido como `write`. Isso permite obter o token do OIDC de cada trabalho do fluxo de trabalho.
- Solicite o JWT do provedor do OIDC {% data variables.product.prodname_dotcom %} e apresente-o ao HashiCorp Vault para receber um token de acesso:
  - Você pode usar o kit de ferramentas [Actions](https://github.com/actions/toolkit/) para buscar os tokens para o seu trabalho, ou você pode usar a ação [`hashicorp/vault-action`](https://github.com/hashicorp/vault-action) para buscar o JWT e receber o token de acesso do Vault.

Este exemplo demonstra como usar o OIDC com a ação oficial para solicitar um segredo ao HashiCorp Vault.

### Adicionando configurações de permissões

 {% data reusables.actions.oidc-permissions-token %}

### Solicitando o token de acesso

A ação `hashicorp/vault-ação` recebe um JWT do provedor de OIDC de {% data variables.product.prodname_dotcom %} e, em seguida, solicita um token de acesso da sua instância do HashiCorp Vault para recuperar segredos. Para obter mais informações, consulte a [documentação](https://github.com/hashicorp/vault-action) do HashiCorp Vault.

Este exemplo demonstra como criar um trabalho que solicita um segredo para o HashiCorp Vault.

- `<Vault URL>`: Substitua isso pela URL do seu HashiCorp Vault.
- `<Role name>`: Substitua isso pela função que você definiu no relacionamento de confiança do HashiCorp Vault.
- `<Audience>`: Substitua isso pela audiência que você definiu no relacionamento de confiança do HashiCorp Vault.
- `<Secret-Path>`: Substitua isso pelo caminho para o segredo que você está recuperando do HashiCorp Vault. Por exemplo: `secret/data/ci npmToken`.

```yaml{:copy}
jobs:
    retrieve-secret:
        steps:
            - name: Retrieve secret from Vault
              uses: hashicorp/vault-action@v2.4.0
              with:
                url: <Vault URL>
                role: <Role name>
                method: jwt
                jwtGithubAudience: <Audience>
                secrets: <Secret-Path>

            - name: Use secret from Vault
               run: |
                 # This step has access to the secret retrieved above; see hashicorp/vault-action for more details.
```
