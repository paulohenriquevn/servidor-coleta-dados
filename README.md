# 📌 Tutorial Completo: Configurando um Servidor Hetzner com GitHub Actions

Este tutorial guiará você pelo processo de **criação, configuração e deploy** de uma aplicação em um servidor **Hetzner** usando **GitHub Actions**.

## 📖 **Sumário**
1. [Pré-requisitos](#pré-requisitos)
2. [Criando Chaves SSH](#criando-chaves-ssh)
3. [Configurando GitHub Secrets](#configurando-github-secrets)
4. [Criando o Arquivo `deploy.yml`](#criando-o-arquivo-deployyml)
5. [Testando a Conexão SSH](#testando-a-conexão-ssh)

---

## 🔹 **Pré-requisitos**
Antes de começar, você precisará de:
- Conta na [Hetzner Cloud](https://www.hetzner.com/cloud).
- GitHub Actions habilitado no repositório.
- Uma chave **SSH** para conexão remota.
- API Token da Hetzner.

---

## 🔑 **Criando Chaves SSH**
Se você ainda não tem um par de chaves SSH, crie uma nova:

```bash
ssh-keygen -t rsa -b 4096 -C "seu-email@example.com"
```

Isso criará dois arquivos:

```bash
    ~/.ssh/id_rsa → Chave privada (NÃO compartilhe!)
    ~/.ssh/id_rsa.pub → Chave pública (usada para acessar o servidor)
```
Copie sua chave pública:
```bash
cat ~/.ssh/id_rsa.pub
```

## 🔐 **Configurando GitHub Secrets**

Agora, adicione as seguintes chaves no seu repositório GitHub:
    Acesse Settings → Secrets and variables → Actions.
    Clique em New repository secret e adicione:

Nome	                Descrição
HETZNER_API_TOKEN	    Token da API da Hetzner Cloud
SSH_PRIVATE_KEY	        Conteúdo do arquivo id_rsa
SSH_PUBLIC_KEY	        Conteúdo do arquivo id_rsa.pub

## ⚙ **Criando o Arquivo deploy.yml**

Crie um arquivo .github/workflows/deploy.yml com o seguinte conteúdo:
```yaml
name: Deploy to Hetzner Cloud

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Configurar e Preparar Servidor
    runs-on: ubuntu-latest
    steps:
    - name: Criar novo servidor Hetzner
      id: setup_server
      uses: saitho/hetzner-cloud-action@master
      with:
        action: create
        server_name: tools-server
        server_image: ubuntu-24.04
        server_type: cx22
        server_location: hel1
        server_ssh_key_name: github-ci
        wait_for_ssh: 1
      env:
        API_TOKEN: ${{ secrets.HETZNER_API_TOKEN }}

    - name: Obter IP do servidor
      run: echo "SERVER_IP=${{ steps.setup_server.outputs.hcloud_server_created_ipv4 }}" >> $GITHUB_ENV

    - name: Configurar SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan -H ${{ env.SERVER_IP }} >> ~/.ssh/known_hosts

    - name: Instalar Docker, Git e Python 3.11
      run: |
        ssh -i ~/.ssh/id_rsa root@${{ env.SERVER_IP }} << 'EOF'
        apt update && apt upgrade -y
        apt install -y docker.io docker-compose git software-properties-common
        add-apt-repository -y ppa:deadsnakes/ppa
        apt update
        apt install -y python3.11 python3.11-venv python3.11-dev python3-pip
        systemctl enable --now docker
        EOF

```

## 🔍 **Testando a Conexão SSH**

Após rodar o pipeline do GitHub Actions, verifique se a conexão SSH funciona:

```bash
ssh deploy@IP_DO_SERVIDOR
```

```bash
ssh-keygen -R IP_DO_SERVIDOR
```

```bash
ssh root@IP_DO_SERVIDOR
```

## 🏆 **Conclusão**

Agora você tem um servidor Hetzner totalmente automatizado via GitHub Actions! 🎯
Esse setup permite criar servidores do zero, configurá-los e fazer deploy de aplicações de forma rápida e eficiente. 🚀