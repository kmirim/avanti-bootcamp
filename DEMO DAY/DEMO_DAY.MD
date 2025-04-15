# Caso Real: O desastre da atualização não testada no time de Backend

## Personagens:
- **Carlos** (Tech Lead do Backend): Queria atualizar o servidor de pagamentos para Node.js 18.
- **Ana** (Dev Backend Jr): Fez a atualização direto no servidor de produção sem avisar.
- **Time de Infra** (Kalu e equipe): Descobriu o problema só quando o app quebrou.

## O Que Aconteceu?
Ana acessou o servidor de produção via SSH e rodou:

```bash
sudo apt-get install -y nodejs=18.0.0  # Atualizou manualmente
```

### O Resultado:
- O app parou de funcionar porque dependia de uma lib incompatível com Node 18.
- Ninguém sabia o que havia mudado (não havia registro da alteração).
- O time ficou 4 horas tentando descobrir o problema.

## Solução com Terraform + Git (Passo a Passo Humanizado)

### 1. Carlos (Tech Lead) Define a Infra como Código
Criou um arquivo chamado `servers.tf`:

```hcl
resource "aws_instance" "servidor_pagamentos" {
  ami           = "ami-0c55b159cbfafe1f0"  # Ubuntu 22.04
  instance_type = "t2.medium"
  user_data     = file("setup_node.sh")    # Script que instala Node.js 16

  tags = {
    Time      = "Backend"
    Responsavel = "Carlos"
  }
}
```

### 2. O Script Controlado (setup_node.sh)
```bash
#!/bin/bash
# Instala a versão CORRETA do Node.js
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 3. Ana quer atualizar? Agora é seguro!
- Ela edita o `setup_node.sh` para Node 18 e abre um Pull Request no GitHub.
- Isa e Kalu (Infra) revisam e testam em homologação.
- Se tudo ok, o merge no main dispara um terraform apply automático.

## Por que isso evita o caos?
- **Rastreabilidade**: Sabemos quem pediu a mudança (Ana) e quando (histórico do Git).
- **Testes**: A atualização é testada antes de ir para produção.
- **Padronização**: Todos os servidores usam a mesma versão do Node.js.

## 🔧 Etapas para resolver o config drift

### 1️⃣ Etapa 1: Identificar o problema
**Objetivo**: Entender onde está a inconsistência.

**Como fazer**:
Acesse servidores manualmente via SSH e compare:

```bash
# Verifique versões de pacotes:
python3 --version
nginx -v
# Verifique configurações críticas:
sudo cat /etc/nginx/nginx.conf
```

**Saída esperada**:
- Diferenças entre servidores (ex: Python 3.8 vs 3.10).

### 2️⃣ Etapa 2: Definir infraestrutura como código (Terraform)
**Objetivo**: Criar um template replicável.

**Arquivo**: `servers.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"  # Ubuntu 22.04
  instance_type = "t2.micro"
  user_data     = file("init_script.sh")   # Script de inicialização

  tags = {
    Name = "web-server-padrao"
  }
}
```

**Conteúdo de `init_script.sh`**:
```bash
#!/bin/bash
apt-get update
apt-get install -y nginx python3
systemctl start nginx
```

### 3️⃣ Etapa 3: Versionar no Git
**Objetivo**: Rastrear mudanças e permitir colaboração.

**Comandos**:
```bash
git init
git add servers.tf init_script.sh
git commit -m "Primeira versão da infraestrutura"
git remote add origin https://github.com/seu-user/repo.git
git push -u origin main
```

### 4️⃣ Etapa 4: Aplicar o Terraform
**Objetivo**: Criar servidores padronizados.

**Comandos**:
```bash
terraform init   # Inicializa o projeto
terraform plan   # Mostra o que será criado
terraform apply  # Executa a criação (digite "yes")
```

**Saída esperada**:
- Servidor AWS EC2 criado com Nginx e Python pré-instalados.

### 5️⃣ Etapa 5: Eliminar o Config Drift
**Objetivo**: Garantir que servidores existentes sigam o padrão.

**Como fazer**:
Para servidores já existentes:

Importe recursos manuais para o Terraform:
```bash
terraform import aws_instance.web_server i-0123456789abcdef0
```

Aplique o código para sincronizar:
```bash
terraform apply
```

### 6️⃣ Etapa 6: Monitorar e Manter
**Objetivo**: Evitar novas inconsistências.

**Ferramentas**:
- GitHub Actions: Pipeline para validar mudanças no Terraform.
- AWS Config: Monitora desvios na infraestrutura.
