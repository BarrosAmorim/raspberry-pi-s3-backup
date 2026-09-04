# 01 — Preparação do Raspberry Pi

## Objetivo

Preparar e validar o ambiente do Raspberry Pi 5 que será utilizado como servidor para o projeto de backup automático para o Amazon S3.

Nesta etapa foram verificadas informações do sistema operacional, arquitetura, armazenamento, memória, Git e AWS CLI.

Também foi realizado um teste de autenticação na AWS, no qual foi identificado um problema com credenciais antigas e posteriormente realizada a correção.

---

## 1. Identificação do sistema

### Comando utilizado

```bash
uname -a
```

### Resultado

```text
Linux rasp 6.18.39+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.18.39-1+rpt1 (2026-07-29) aarch64 GNU/Linux
```

### Informações identificadas

- Kernel: `6.18.39+rpt-rpi-2712`
- Arquitetura: `aarch64` (ARM64)
- Sistema: Linux

---

## 2. Identificação da distribuição Linux

### Comando utilizado

```bash
cat /etc/os-release
```

### Resultado

```text
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"
VERSION_CODENAME=trixie
DEBIAN_VERSION_FULL=13.6
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

### Informações identificadas

- Distribuição: Debian GNU/Linux
- Versão: 13.6
- Codinome: Trixie

---

## 3. Verificação do armazenamento

### Comando utilizado

```bash
df -h
```

### Resultado

```text
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           1.6G   22M  1.6G   2% /run
/dev/sdb2       469G  228G  223G  51% /
tmpfs           4.0G   32K  4.0G   1% /dev/shm
tmpfs           5.0M   48K  5.0M   0% /run/lock
tmpfs           4.0G   61M  3.9G   2% /tmp
/dev/sdb1       510M   67M  444M  14% /boot/firmware
/dev/zram1      224M   15M  192M   8% /var/log
tmpfs           1.0M     0     0   0% /run/credentials/systemd-journald.service
/dev/sda3       931G  849G   83G  92% /mnt/wdblue
tmpfs           807M  128K  807M   1% /run/user/1000
tmpfs           1.0M     0     0   0% /run/credentials/getty@tty1.service
tmpfs           1.0M     0     0   0% /run/credentials/serial-getty@ttyAMA10.service
```

### Observação

O sistema principal possui espaço disponível suficiente para a execução do projeto.

O armazenamento montado em `/mnt/wdblue` apresenta utilização elevada. Por isso, ele não será considerado como área principal para armazenamento temporário dos backups deste projeto.

---

## 4. Verificação da memória

### Comando utilizado

```bash
free -h
```

### Resultado

```text
               total        used        free      shared  buff/cache   available
Mem:           7.9Gi       1.6Gi       230Mi       218Mi       6.4Gi       6.2Gi
Swap:          2.0Gi          0B       2.0Gi
```

### Informações identificadas

- RAM total: `7.9 GiB`
- RAM disponível no momento do teste: `6.2 GiB`
- Swap total: `2.0 GiB`
- Swap utilizada: `0 B`

---

## 5. Identificação do usuário

### Comando utilizado

```bash
whoami
```

### Resultado

```text
rafael
```

O projeto será executado utilizando o usuário `rafael`, evitando o uso desnecessário do usuário `root`.

---

## 6. Diretório atual

### Comando utilizado

```bash
pwd
```

### Resultado

```text
/home/rafael
```

---

## 7. Verificação do Git

### Comando utilizado

```bash
git --version
```

### Resultado

```text
git version 2.47.3
```

O Git já estava instalado no Raspberry Pi e poderá ser utilizado para versionamento do projeto.

---

## 8. Verificação do AWS CLI

### Comando utilizado

```bash
aws --version
```

### Resultado

```text
aws-cli/2.23.6 Python/3.13.5 Linux/6.18.39+rpt-rpi-2712 source/aarch64.debian.13
```

O AWS CLI já estava instalado e será utilizado para realizar operações na AWS, principalmente no Amazon S3.

---

## 9. Verificação da configuração do AWS CLI

### Comando utilizado

```bash
aws configure list
```

### Resultado

```text
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************BYNW shared-credentials-file
secret_key     ****************66+a shared-credentials-file
    region                us-east-1      config-file    ~/.aws/config
```

### Observação de segurança

O AWS CLI encontrou credenciais configuradas no Raspberry Pi por meio do arquivo de credenciais compartilhadas.

As chaves de acesso não serão armazenadas na documentação, no código-fonte ou no GitHub.

A região configurada inicialmente era:

```text
us-east-1
```

---

## 10. Teste inicial de autenticação na AWS

### Comando utilizado

```bash
aws sts get-caller-identity
```

### Resultado

```text
An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid.
```

### Problema identificado

O AWS CLI estava utilizando credenciais antigas que haviam sido configuradas no Raspberry Pi durante um projeto anterior.

Essas credenciais não eram mais válidas, resultando no erro:

```text
InvalidClientTokenId
```

Esse problema precisou ser corrigido antes de continuar a configuração do backup.

---

## 11. Verificação da Access Key configurada

### Comando utilizado

```bash
aws configure get aws_access_key_id
```

### Resultado

O comando retornou a Access Key ID que estava configurada no AWS CLI.

Por segurança, o valor completo da chave não é registrado neste documento nem publicado no GitHub.

Essa verificação foi utilizada para confirmar qual credencial estava sendo utilizada pelo AWS CLI antes da correção.

---

## 12. Correção da autenticação AWS

Durante a configuração do IAM, foi criado um usuário dedicado ao projeto:

```text
pi-s3-backup
```

Também foi criada uma nova Access Key para utilização pelo AWS CLI.

As novas credenciais foram configuradas no Raspberry Pi utilizando:

```bash
aws configure
```

Foram configurados:

```text
Default region name:
us-east-1

Default output format:
json
```

As credenciais não são armazenadas neste projeto.

---

## 13. Validação da nova autenticação

Após a configuração das novas credenciais, foi executado novamente:

```bash
aws sts get-caller-identity
```

O comando confirmou a autenticação utilizando o usuário IAM:

```text
arn:aws:iam::...:user/pi-s3-backup
```

Isso confirmou que o Raspberry Pi estava autenticando corretamente na AWS.

---

## 14. Validação do acesso ao Amazon S3

Após a correção da autenticação, também foram realizados testes utilizando o bucket:

```text
raspberry-pi-s3-backup-lab-2026
```

Foi testada a listagem:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/
```

O bucket estava vazio inicialmente e o comando foi executado sem erro de permissão.

Posteriormente foram realizados testes de:

- upload;
- download;
- exclusão de objeto.

Todos os testes foram concluídos com sucesso.

Os detalhes desses testes estão documentados na etapa de configuração do Amazon S3.

---

# Status da preparação

### Concluído

- [x] Sistema operacional identificado
- [x] Arquitetura identificada
- [x] Kernel identificado
- [x] Armazenamento verificado
- [x] Memória verificada
- [x] Usuário identificado
- [x] Diretório de trabalho identificado
- [x] Git verificado
- [x] AWS CLI verificado
- [x] Configuração do AWS CLI verificada
- [x] Problema de autenticação identificado
- [x] Credenciais antigas substituídas
- [x] Nova autenticação AWS configurada
- [x] Autenticação AWS validada
- [x] Acesso ao Amazon S3 validado

---

## Conclusão

A preparação do Raspberry Pi 5 foi concluída.

O ambiente possui os recursos necessários para executar o projeto e o AWS CLI está instalado e configurado.

Durante a preparação foi identificado um problema real com credenciais AWS antigas. O problema foi corrigido através da criação de novas credenciais para o usuário IAM dedicado ao projeto.

A autenticação foi posteriormente validada com sucesso utilizando:

```bash
aws sts get-caller-identity
```

O Raspberry Pi está pronto para continuar a implementação do sistema de backup automático para o Amazon S3.

---

[README](../README.md) | [Próximo: Configuração IAM](02-configuracao-iam.md)
