# Guia de Laboratório: Gerenciamento Seguro de Configurações com AWS Parameter Store (SSM) e KMS

## 1. Visão Geral

Este laboratório ensina como armazenar e acessar configurações de aplicativos, como URLs de banco de dados e senhas, de forma segura. Para isso, usaremos o **AWS Systems Manager (SSM) Parameter Store** para o armazenamento e o **AWS KMS** para criptografar dados sensíveis. Finalmente, acessaremos esses parâmetros de forma segura utilizando o **AWS CloudShell**.

## 2. Objetivos de Aprendizagem

I. Criar parâmetros do tipo **String** (texto simples) e **SecureString** (criptografado) no SSM Parameter Store.
II. Utilizar uma estrutura de nomes hierárquica para organizar os parâmetros.
III. Criar uma chave de criptografia customizada no AWS Key Management Service (KMS).
IV. Associar a chave KMS a parâmetros do tipo **SecureString** para máxima segurança.
V. Acessar e descriptografar os parâmetros utilizando comandos da AWS CLI no CloudShell.

## 3. Cenário

Você é um desenvolvedor responsável por gerenciar as configurações de uma aplicação que possui ambientes de desenvolvimento (**dev**) e produção (**prod**). É crucial que as senhas do banco de dados sejam armazenadas de forma criptografada e que o acesso a todas as configurações seja controlado e auditável.

---

## 4. Preparação da Criptografia (AWS KMS)

Antes de armazenarmos segredos, precisamos criar a chave que irá protegê-los.

I. Acesse o Console de Gerenciamento da AWS e, na barra de busca, navegue até o serviço **KMS (Key Management Service)**.
II. No menu esquerdo, clique em **Chaves gerenciadas pelo cliente**.
III. Clique em **Criar chave**.
IV. **Configurar chave**: mantenha a opção *Simétrica* e *Criptografar e descriptografar* selecionada. Clique em **Próximo**.
V. **Adicionar etiquetas**:

* Alias: insira um nome único, por exemplo: `minha-chave-app-seunome-sobrenome`
* Descrição: *(opcional)* Chave para criptografar segredos do meu app.
* Clique em **Próximo**.
  VI. **Definir permissões de uso da chave**: para este laboratório, não precisamos adicionar outros usuários. Clique em **Próximo**.
  VII. **Revisar e finalizar**: clique em **Finalizar**.

✅ Sua chave de criptografia personalizada está criada e pronta para uso.

---

## 5. Criação dos Parâmetros no SSM Parameter Store

### 5.1. Parâmetros de Texto Simples (String)

I. Parâmetro 1: URL do Banco de Dados de Desenvolvimento

* Nome: `/meu-app-seunome-sobrenome/dev/db-url`
* Descrição: URL do banco de dados para o ambiente de dev.
* Tipo: **String**
* Valor: `dev.database.seunome.com:3306`

II. Parâmetro 2: URL do Banco de Dados de Produção

* Nome: `/meu-app-seunome-sobrenome/prod/db-url`
* Descrição: URL do banco de dados para o ambiente de prod.
* Tipo: **String**
* Valor: `prod.database.seunome.com:3306`

---

### 5.2. Parâmetros Criptografados (SecureString)

III. Parâmetro 3: Senha do Banco de Dados de Desenvolvimento

* Nome: `/meu-app-seunome-sobrenome/dev/db-password`
* Descrição: Senha do banco de dados para o ambiente de dev.
* Tipo: **SecureString**
* ID da chave KMS: alias criado na etapa 4.
* Valor: `SenhaSuperSecretaDeDesenvolvimento123`

IV. Parâmetro 4: Senha do Banco de Dados de Produção

* Nome: `/meu-app-seunome-sobrenome/prod/db-password`
* Descrição: Senha do banco de dados para o ambiente de prod.
* Tipo: **SecureString**
* ID da chave KMS: alias criado na etapa 4.
* Valor: `SenhaUltraSeguraDeProducao@2025`

✅ Ao final, você terá **4 parâmetros**: dois do tipo **String** e dois do tipo **SecureString**.

---

## 6. Acesso via Linha de Comando (CloudShell)

### 6.1. Buscando Parâmetros (Sem Descriptografia)

```bash
aws ssm get-parameters \
 --names /meu-app-seunome-sobrenome/dev/db-url /meu-app-seunome-sobrenome/dev/db-password
```

I. O parâmetro `db-url` será exibido em texto claro.
II. O parâmetro `db-password` aparecerá criptografado.

---

### 6.2. Buscando e Descriptografando Parâmetros

```bash
aws ssm get-parameters \
 --names /meu-app-seunome-sobrenome/dev/db-url /meu-app-seunome-sobrenome/dev/db-password \
 --with-decryption
```

I. Agora o valor do `db-password` será exibido em **texto claro** após descriptografia via KMS.

---

## 7. Limpeza dos Recursos (Etapa Crucial)

### 7.1. Excluir os Parâmetros do SSM

I. Volte ao **Parameter Store**.
II. Selecione os quatro parâmetros criados.
III. Clique em **Excluir** e confirme.

### 7.2. Agendar a Exclusão da Chave KMS

I. Vá para o serviço **KMS** → **Chaves gerenciadas pelo cliente**.
II. Selecione a chave criada (`minha-chave-app-...`).
III. Clique em **Ações da chave → Programar exclusão da chave**.
IV. Defina o período mínimo de espera: **7 dias**.
V. Confirme a exclusão.

⚠️ Importante: este tempo de espera é um mecanismo de segurança. Se a chave for excluída, **todos os dados criptografados com ela se tornam irrecuperáveis**.
