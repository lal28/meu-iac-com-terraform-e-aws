# IAC com Terraform e AWS

Este repositório contém a resolução do desafio de Infraestrutura como Código (IAC) proposto no curso de DEVOPS, utilizando Terraform e AWS para provisionar recursos na nuvem e Ansible para automação da configuração.

Documentos de instruções disponível em [formato PDF](docs/README.pdf) e [formato HTML](docs/README.html)


# Erros Encontrados durante o acompanhamento e Soluções - Tutorial IAC Terraform + AWS

---

## Erro 1: Reference to undeclared resource

### Mensagem de erro:
```
Error: Reference to undeclared resource
│ 
│   on ec2.tf line 3, in resource "aws_instance" "web_server":
│    3:     ami           = data.aws_ami.amazon_linux.id
│ 
│ A data resource "aws_ami" "amazon_linux" has not been declared in the root module.
```

### Causa:
Inconsistência no nome do data source entre `data.tf` e `ec2.tf`

### Solução:
No arquivo `ec2.tf`, alterar:

**Antes:**
```hcl
ami = data.aws_ami.amazon_linux.id
```

**Depois:**
```hcl
ami = data.aws_ami.amazon_linux_2023.id
```

---

## Erro 2: Instance type not eligible for Free Tier

### Mensagem de erro:
```
Error: creating EC2 Instance: operation error EC2: RunInstances, https response error StatusCode: 400, 
RequestID: 4d44943a-cab5-4ee2-883d-a02b974cdba4, api error InvalidParameterCombination: 
The specified instance type is not eligible for Free Tier. For a list of Free Tier instance types, 
run 'describe-instance-types' with the filter 'free-tier-eligible=true'.
│ 
│   with aws_instance.web_server,
│   on ec2.tf line 2, in resource "aws_instance" "web_server":
│    2: resource "aws_instance" "web_server" {
```

### Causa:
O tipo `t2.micro` não está mais disponível no Free Tier em algumas regiões

### Solução:
No arquivo `ec2.tf`, alterar:

**Antes:**
```hcl
instance_type = "t2.micro"
```

**Depois:**
```hcl
instance_type = "t3.micro"
```

---

## Erro 3: amazon-linux-extras command not found (Ansible)

### Mensagem de erro:
```
TASK [Install Nginx on Amazon Linux 2] **************************************************************
fatal: [13.221.111.45]: FAILED! => {"changed": false, "cmd": "amazon-linux-extras install -y nginx1", 
"msg": "[Errno 2] No such file or directory: b'amazon-linux-extras'", "rc": 2, "stderr": "", 
"stderr_lines": [], "stdout": "", "stdout_lines": []}
```

### Causa:
Amazon Linux 2023 não possui o comando `amazon-linux-extras` (presente apenas no Amazon Linux 2)

### Solução:
No arquivo `playbook.yml`, fazer as seguintes alterações:

**1. Substituir a task de instalação do Nginx:**

**Antes:**
```yaml
- name: Install Nginx on Amazon Linux 2
  command: amazon-linux-extras install -y nginx1
  args:
    creates: /usr/sbin/nginx
```

**Depois:**
```yaml
- name: Install Nginx on Amazon Linux 2023
  dnf:
    name: nginx
    state: present
```

**2. Alterar a primeira task de `yum` para `dnf`:**

**Antes:**
```yaml
- name: Ensure all packages are up to date
  yum:
    name: '*'
    state: latest
```

**Depois:**
```yaml
- name: Ensure all packages are up to date
  dnf:
    name: '*'
    state: latest
```

**3. Alterar a instalação do Git:**

**Antes:**
```yaml
- name: Install Git
  yum:
    name: git
    state: present
```

**Depois:**
```yaml
- name: Install Git
  dnf:
    name: git
    state: present
```

---

## Informações do Ambiente

- **Sistema Operacional:** WSL2 (Ubuntu) + Windows
- **Versão AMI:** Amazon Linux 2023
- **Região AWS:** Variável (ajustar conforme disponibilidade do Free Tier)
- **Terraform:** Versão compatível com provider AWS ~> 5.0
- **Ansible:** Versão 2.16+

---

## Observações Importantes

1. **Amazon Linux 2 vs 2023:** O Amazon Linux 2023 usa `dnf` como gerenciador de pacotes, substituindo o `yum` e removendo o `amazon-linux-extras`

2. **Free Tier:** Verifique sempre quais tipos de instância estão disponíveis no Free Tier da sua região usando:
   ```bash
   aws ec2 describe-instance-types \
     --filters "Name=free-tier-eligible,Values=true" \
     --query "InstanceTypes[].InstanceType" \
     --output table
   ```

3. **IP Público:** Não esqueça de atualizar o arquivo `inventory` com o IP público da instância EC2 após o `terraform apply`

4. **Security Group SSH:** Configure seu IP público no arquivo `variables.tf` antes de executar o Terraform
## Saiba mais

- [Documentação do Terraform](https://developer.hashicorp.com/terraform)
- [Documentação do Provider AWS do Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Lista de Providers do Terraform](https://registry.terraform.io/browse/providers)
- [Documentação da AWS](https://docs.aws.amazon.com/pt_br/)

