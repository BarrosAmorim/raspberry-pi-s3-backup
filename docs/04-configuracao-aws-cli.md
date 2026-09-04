# 04 — Configuração e Validação do AWS CLI

## Objetivo

Configurar e validar o AWS CLI no Raspberry Pi 5 para permitir que o servidor utilize os serviços da AWS por linha de comando.

Neste projeto, o AWS CLI será utilizado posteriormente pelo script de backup para enviar os arquivos do Raspberry Pi para o Amazon S3.

Nesta etapa foram validados:

- configuração das credenciais;
- região padrão;
- autenticação na AWS;
- identidade do usuário IAM;
- acesso ao bucket S3;
- comportamento das permissões IAM.

---

## 1. Verificação da configuração do AWS CLI

O primeiro passo foi verificar como o AWS CLI estava configurado no Raspberry Pi.

### Comando utilizado

```bash
aws configure list
```

### Resultado

```text
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************MF4Z shared-credentials-file
secret_key     ****************DMB3 shared-credentials-file
    region                us-east-1      config-file    ~/.aws/config
```

### Análise

O resultado confirmou que:

- não existe um profile nomeado configurado;
- o AWS CLI está utilizando o profile padrão;
- as credenciais estão sendo obtidas do arquivo de credenciais compartilhadas;
- a região configurada é `us-east-1`.

As chaves de acesso não são registradas neste documento por questões de segurança.

---

## 2. Validação da autenticação na AWS

Depois de confirmar a configuração local, foi necessário verificar se as credenciais realmente permitiam autenticação na AWS.

### Comando utilizado

```bash
aws sts get-caller-identity
```

### Resultado

O comando confirmou a identidade do usuário IAM utilizado pelo Raspberry Pi:

```text
Arn: arn:aws:iam::696537703431:user/pi-s3-backup
```

### O que esse comando faz?

O `aws sts get-caller-identity` consulta o AWS Security Token Service (STS) e informa qual identidade AWS está sendo utilizada pelas credenciais configuradas.

Essa é uma das principais formas de verificar se o AWS CLI está autenticando corretamente.

### Resultado

A autenticação foi realizada com sucesso utilizando o usuário:

```text
pi-s3-backup
```

Isso também confirma que o Raspberry Pi não está utilizando o usuário Root da conta AWS para realizar as operações do projeto.

---

## 3. Validação do acesso ao bucket S3

Depois de validar a identidade, foi realizado um teste de acesso ao bucket utilizado pelo projeto.

### Comando utilizado

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/
```

### Resultado

O comando foi executado sem apresentar erro.

Como o bucket estava vazio naquele momento, nenhum objeto foi listado.

### O que esse comando faz?

O comando `aws s3 ls` lista objetos armazenados em um bucket S3.

Neste caso:

```text
aws s3 ls
        ↓
s3://raspberry-pi-s3-backup-lab-2026/
```

O teste confirmou que o usuário IAM consegue realizar a operação de listagem no bucket.

Essa permissão corresponde à ação:

```text
s3:ListBucket
```

---

## 4. Teste de uma operação não autorizada

Para entender melhor como a política IAM está funcionando, foi realizado um teste utilizando a API `GetBucketLocation`.

### Comando utilizado

```bash
aws s3api get-bucket-location --bucket raspberry-pi-s3-backup-lab-2026
```

### Resultado

```text
An error occurred (AccessDenied) when calling the GetBucketLocation operation: User: arn:aws:iam::696537703431:user/pi-s3-backup is not authorized to perform: s3:GetBucketLocation on resource: "arn:aws:s3:::raspberry-pi-s3-backup-lab-2026" because no identity-based policy allows the s3:GetBucketLocation action
```

### O que aconteceu?

O AWS CLI estava funcionando corretamente e o usuário estava autenticado.

O erro ocorreu porque a política IAM criada para o projeto **não concede** a ação:

```text
s3:GetBucketLocation
```

A política possui somente as permissões necessárias para o funcionamento do backup:

```text
s3:ListBucket
s3:PutObject
s3:GetObject
s3:DeleteObject
```

Portanto, o AWS bloqueou corretamente uma operação que não foi autorizada.

---

## 5. Por que não adicionar `s3:GetBucketLocation`?

Neste projeto não existe necessidade de consultar a localização do bucket através dessa API.

A região do bucket já foi definida durante sua criação:

```text
us-east-1
```

E essa também é a região configurada no AWS CLI.

Por isso, não foi adicionada uma permissão extra apenas para fazer o teste funcionar.

Essa decisão segue o princípio de:

> **Least Privilege — Menor Privilégio**

A ideia é conceder ao usuário somente as permissões necessárias para executar sua função.

Neste projeto, o usuário `pi-s3-backup` precisa:

```text
Listar backups
      ↓
Enviar backups
      ↓
Baixar backups
      ↓
Excluir backups antigos
```

Não é necessário conceder permissões administrativas ou outras ações do S3 que não sejam utilizadas pelo processo.

---

## 6. Relação entre AWS CLI e o projeto

A estrutura planejada para o funcionamento do projeto é:

```text
Raspberry Pi 5
      │
      │
      ▼
Script de Backup
      │
      │
      ▼
AWS CLI
      │
      │
      ▼
IAM
      │
      │
      ▼
Amazon S3
```

O AWS CLI será o componente responsável por realizar as operações no S3 a partir do Raspberry Pi.

Posteriormente, o script `backup.sh` utilizará o AWS CLI para automatizar o envio dos arquivos.

---

## 7. Segurança das credenciais

As credenciais utilizadas pelo AWS CLI são mantidas no ambiente do Raspberry Pi e não fazem parte do código-fonte do projeto.

Nenhuma Access Key ou Secret Access Key será publicada no GitHub.

O projeto também utilizará um usuário IAM específico:

```text
pi-s3-backup
```

em vez de utilizar credenciais do usuário Root.

Essa separação reduz o impacto caso as credenciais do projeto sejam comprometidas.

---

## 8. Resultado da configuração

| Validação                     | Resultado                |
| ----------------------------- | ------------------------ |
| AWS CLI instalado             | OK                       |
| Credenciais configuradas      | OK                       |
| Região configurada            | `us-east-1`              |
| Autenticação AWS              | OK                       |
| Usuário IAM                   | `pi-s3-backup`           |
| Acesso ao bucket              | OK                       |
| Listagem do bucket            | OK                       |
| `GetBucketLocation`           | Negado conforme política |
| Princípio de menor privilégio | Aplicado                 |

---

## 9. Conclusão

A configuração e validação do AWS CLI no Raspberry Pi 5 foram concluídas.

O Raspberry Pi consegue autenticar na AWS utilizando o usuário IAM dedicado ao projeto e possui acesso ao bucket S3 necessário para o sistema de backup.

Também foi possível validar, na prática, o funcionamento das restrições de IAM. A tentativa de executar uma ação não autorizada resultou em `AccessDenied`, demonstrando que o usuário não possui permissões além das definidas na política.

A configuração está pronta para a próxima etapa: desenvolvimento do script responsável por realizar os backups automaticamente.

---

[Anterior: Configuração do S3](03-configuracao-s3.md) | [README](../README.md) | [Próximo: Script de Backup](05-script-backup-retencao.md)
