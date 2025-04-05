# 🚀 Provisionamento AWS com Terraform

## 📋 Índice
- [Pré-requisitos](#-pré-requisitos)
- [Configuração da Conta AWS](#-1-configuração-da-conta-aws)
- [Instalação Local](#-2-instalação-local)
- [Autenticação AWS](#-3-autenticação-aws)
- [Preparação do Projeto](#-4-preparação-do-projeto)
- [Execução do Terraform](#-5-execução-do-terraform)
- [Verificações](#-6-verificações)
- [Tratativa de Erros](#-7-tratativa-de-erros-comuns)
- [Destruição dos Recursos](#-8-destruição-dos-recursos)
- [Observações](#-observações)

## ✅ Pré-requisitos

- [ ] Conta AWS criada e ativa  
- [ ] Acesso root ou IAM com permissões administrativas temporárias  

---

## 🔐 1. Configuração da Conta AWS

### Criar conta AWS

1. Acesse: [https://console.aws.amazon.com](https://console.aws.amazon.com)  
2. Complete o cadastro  
3. Ative a **verificação em duas etapas**

### Criar usuário IAM com permissões específicas (recomendado)

1. Acesse **IAM Console** > **Users**  
2. Selecione seu usuário > **Add permissions** > **Attach policies directly**

#### Permissões mínimas recomendadas:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:CreateSecurityGroup",
        "ec2:Describe*",
        "ec2:TerminateInstances",
        "iam:CreateKeyPair"
      ],
      "Resource": "*"
    }
  ]
}
```

> **Nota**: Alternativamente, você pode usar a política gerenciada `AmazonEC2FullAccess` (menos segura, mas rápida para testes).

## 💻 2. Instalação Local

### Terraform

#### Linux (Debian/Ubuntu)
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update && sudo apt install terraform
```

#### Windows (via Chocolatey)
```powershell
choco install terraform
```

#### Solução para erro de repositório
Se ocorrer erro do tipo:
```
E: Malformed entry 1 in list file /etc/apt/sources.list.d/hashicorp.list (Component)
E: The list of sources could not be read.
```

Execute:
```bash
wget https://releases.hashicorp.com/terraform/1.6.4/terraform_1.6.4_linux_amd64.zip
unzip terraform_*.zip
sudo mv terraform /usr/local/bin/
terraform --version
```

### AWS CLI

#### Linux
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Windows
```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi /quiet
```
> **Nota**: Esse comando é usado para instalar silenciosamente a AWS CLI no Windows sem intervenção do usuário.

## 🔑 3. Autenticação AWS

1. Configure suas credenciais:
```bash
aws configure
```

2. Preencha as informações:
- AWS Access Key ID
- AWS Secret Access Key
- Default region: us-east-1
- Default output: json

3. Teste o acesso:
```bash
# Listar buckets S3
aws s3 ls

# Verificar identidade
aws sts get-caller-identity
```

> **Dica**: Se receber erro "Unable to locate credentials", verifique se a AWS CLI está funcionando corretamente.

## 📂 4. Preparação do Projeto

### Estrutura de arquivos
```plaintext
/
├── main.tf          # Configuração do provider
├── ec2.tf           # Instância EC2
├── security-group.tf # Regras de segurança
├── script.sh        # User data (Apache/Nginx)
└── outputs.tf       # IP público e outros outputs
```

## 🚀 5. Execução do Terraform

### Inicialização
```bash
terraform init
```

### Planejamento
```bash
terraform plan
```
> Verifique se 1 EC2 instance e 1 Security Group serão criados

### Aplicação
```bash
terraform apply
```
> Confirme com 'yes' quando solicitado

## ✔️ 6. Verificações

1. Acesse o Console AWS > EC2
2. Confirme:
   - Instância com tag Name = Web Server
   - Status running
3. Teste de conexão:
```bash
curl http://$(terraform output -raw public_ip)
```

## 💥 7. Tratativa de Erros Comuns

### InvalidKeyPair.NotFound
```bash
aws ec2 create-key-pair --key-name terraform-key --query 'KeyMaterial' --output text > ~/.ssh/terraform-key.pem
chmod 400 ~/.ssh/terraform-key.pem
```

### UnauthorizedOperation
Adicione ao usuário IAM:
```json
{
    "Effect": "Allow",
    "Action": "ec2:CreateSecurityGroup",
    "Resource": "*"
}
```

## 🧹 8. Destruição dos Recursos

```bash
terraform destroy
```
> ⚠️ Execute `terraform destroy` SOMENTE após registrar todas as evidências

## 📝 Observações

1. **Key Pair**: Substitua `terraform-key` pelo nome da sua chave existente ou use o bloco `aws_key_pair` no Terraform  
2. **Região**: Verifique se todos os recursos estão na mesma região (`us-east-1`)  
3. **Segurança**: Para ambientes reais, restrinja o acesso SSH no Security Group  

---

## 🔗 Links Úteis

- [Repositório do Exercício](https://github.com/luizcarlos16/bt-dvp-v2-terraform)
- [Documentação AWS](https://docs.aws.amazon.com/)
- [Documentação Terraform](https://www.terraform.io/docs)
