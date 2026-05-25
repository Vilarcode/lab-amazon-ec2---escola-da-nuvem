# Lab: Introduction to Amazon EC2

> Laboratório prático de introdução ao Amazon EC2 — provisionamento, monitoramento, configuração de rede, redimensionamento e encerramento seguro de instâncias na AWS.

---

## Sobre o Projeto

Este repositório documenta meu primeiro laboratório prático com **Amazon EC2**, cobrindo o ciclo de vida completo de uma instância: desde a criação até o encerramento, passando por monitoramento, configuração de segurança e redimensionamento.

**Plataforma:** AWS Console  
**Duração:** ~45 minutos  
**Nível:** Iniciante

---

## Objetivos Alcançados

-  Lançar uma instância EC2 com **proteção contra encerramento** ativada
-  Configurar um **servidor web Apache** via User Data (script de inicialização)
-  Monitorar a instância com **Amazon CloudWatch**
-  Atualizar o **Security Group** para permitir tráfego HTTP (porta 80)
-  **Redimensionar** o tipo de instância (t3.micro → t3.small)
-  **Ampliar volume EBS** (8 GiB → 10 GiB)
-  Testar e desativar a **proteção contra encerramento**
-  Encerrar a instância com segurança

---

## Arquitetura

```
Internet
    │
    ▼
┌─────────────────────────────────┐
│          AWS Cloud              │
│                                 │
│  ┌──────────────────────────┐   │
│  │        VPC (Lab VPC)     │   │
│  │                          │   │
│  │  ┌────────────────────┐  │   │
│  │  │  Security Group    │  │   │
│  │  │  • HTTP (80) ✅    │  │   │
│  │  │  • SSH (22)  ❌    │  │   │
│  │  │                    │  │   │
│  │  │  EC2 - Web Server  │  │   │
│  │  │  • t3.small        │  │   │
│  │  │  • Amazon Linux    │  │   │
│  │  │  • Apache httpd    │  │   │
│  │  │                    │  │   │
│  │  │  EBS Volume        │  │   │
│  │  │  • 10 GiB (root)   │  │   │
│  │  └────────────────────┘  │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘
```



##  Passo a Passo Executado

### Tarefa 1 — Lançar a Instância EC2

| Configuração | Valor |
|---|---|
| Nome | Web Server |
| AMI | Amazon Linux 2023 |
| Tipo | t3.micro |
| Par de chaves | Sem par de chaves |
| VPC | Lab VPC |
| Security Group | Web Server security group |
| Volume EBS | 8 GiB |
| Proteção contra encerramento | ✅ Habilitada |

**Script de User Data utilizado na inicialização:**

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

> Este script instala o Apache, o configura para iniciar automaticamente e cria uma página HTML simples.

---

### Tarefa 2 — Monitorar a Instância

- Verificado o status da instância: **2/2 verificações aprovadas**
- Exploradas as métricas do **Amazon CloudWatch** (CPU, rede, disco)
- Utilizado o recurso **"Obter captura de tela da instância"** — útil para diagnóstico quando não há acesso SSH/RDP

---

### Tarefa 3 — Configurar Security Group e Acessar o Servidor Web

**Problema identificado:** o servidor web não estava acessível via navegador, pois o Security Group não permitia tráfego HTTP.

**Solução:** adicionada regra de entrada no Security Group:

| Tipo | Protocolo | Porta | Origem |
|---|---|---|---|
| HTTP | TCP | 80 | 0.0.0.0/0 (IPv4 anywhere) |

Após a alteração, o servidor respondeu com:

```
Hello From Your Web Server!
```

---

### Tarefa 4 — Redimensionar a Instância

**Tipo de instância:** `t3.micro` → `t3.small` (dobro de memória: 1 GiB → 2 GiB)  
**Volume EBS:** `8 GiB` → `10 GiB`

>  Para redimensionar, a instância precisa ser **interrompida** antes. Não há perda de dados — apenas o estado de execução é suspenso.

---

### Tarefa 5 — Testar a Proteção Contra Encerramento

1. Tentativa de encerrar a instância → **bloqueada com erro** (proteção ativa)
2. Desabilitada a proteção via **Ações → Configurações de instância**
3. Instância encerrada com sucesso

---

##  Aprendizados

- **AMI (Amazon Machine Image):** template que define o SO e configurações base da instância
- **Security Groups** funcionam como firewall stateful — por padrão, bloqueiam todo tráfego de entrada
- **User Data** permite automatizar configurações na primeira inicialização da instância
- **EBS (Elastic Block Store)** persiste dados mesmo após parar a instância — diferente do armazenamento efêmero
- **Proteção contra encerramento** é uma camada de segurança importante para ambientes de produção
- O **CloudWatch** oferece métricas essenciais para monitoramento sem precisar acessar a instância diretamente

---

## 🔧 Tecnologias e Serviços AWS

- **Amazon EC2** — computação em nuvem
- **Amazon EBS** — armazenamento em bloco
- **Amazon CloudWatch** — monitoramento e métricas
- **VPC** — rede virtual privada
- **Security Groups** — controle de tráfego (firewall)
- **Apache HTTP Server (httpd)** — servidor web

---


Laboratório realizado como parte dos meus estudos em Cloud Computing e preparação para certificação AWS.
