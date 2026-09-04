# 08 — Troubleshooting

## Objetivo

Durante a construção de um ambiente de infraestrutura, é normal encontrar erros de configuração, permissões ou automação.

Nesta etapa são documentados os principais problemas encontrados durante o desenvolvimento do projeto de backup do Raspberry Pi 5 para o Amazon S3.

O objetivo é registrar:

- o problema encontrado;
- o comportamento observado;
- a investigação realizada;
- a causa identificada;
- a solução aplicada;
- a validação realizada.

O troubleshooting é uma habilidade importante em ambientes de infraestrutura e DevOps, pois permite identificar e resolver problemas de forma organizada.

---

## 1. Problema com as credenciais do AWS CLI

### Sintoma

Durante a configuração inicial do AWS CLI, foi executado o comando:

```bash
aws sts get-caller-identity
```

O comando retornou um erro:

```text
InvalidClientTokenId
```

Esse erro indicava que as credenciais utilizadas pelo AWS CLI não estavam sendo aceitas pela AWS.

---

### Investigação

Primeiro foi verificada a configuração atual do AWS CLI:

```bash
aws configure list
```

O comando mostrou que existiam credenciais configuradas no Raspberry Pi.

Porém, essas credenciais estavam relacionadas a uma configuração anterior utilizada em outro projeto.

Dessa forma, o AWS CLI estava tentando autenticar utilizando credenciais que não eram mais válidas para o ambiente atual.

---

### Solução

As credenciais antigas foram substituídas pelas credenciais do usuário IAM criado especificamente para este projeto:

```text
pi-s3-backup
```

Depois da reconfiguração, o comando foi executado novamente:

```bash
aws sts get-caller-identity
```

O comando passou a retornar corretamente a identidade:

```text
arn:aws:iam::696537703431:user/pi-s3-backup
```

---

### Resultado

A autenticação foi validada com sucesso.

O Raspberry Pi passou a utilizar o usuário IAM correto para executar as operações do projeto.

---

## 2. Erro `AccessDenied` no S3

### Sintoma

Durante os testes de permissões IAM, foi executado:

```bash
aws s3api get-bucket-location --bucket raspberry-pi-s3-backup-lab-2026
```

A AWS retornou:

```text
An error occurred (AccessDenied) when calling the GetBucketLocation operation: User: arn:aws:iam::696537703431:user/pi-s3-backup is not authorized to perform: s3:GetBucketLocation on resource: "arn:aws:s3:::raspberry-pi-s3-backup-lab-2026" because no identity-based policy allows the s3:GetBucketLocation action
```

---

### Investigação

O usuário IAM estava autenticado corretamente.

Isso foi confirmado anteriormente com:

```bash
aws sts get-caller-identity
```

Também foi possível acessar o bucket utilizando:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/
```

Portanto, o problema não era de autenticação.

A mensagem `AccessDenied` indicava especificamente que a ação:

```text
s3:GetBucketLocation
```

não estava autorizada.

---

### Análise da política IAM

A política criada para o projeto possui as seguintes permissões:

```text
s3:ListBucket
s3:PutObject
s3:GetObject
s3:DeleteObject
```

A ação:

```text
s3:GetBucketLocation
```

não fazia parte da política.

---

### Solução

Neste caso, não foi adicionada uma nova permissão.

O comportamento foi considerado correto porque a ação não era necessária para o funcionamento do sistema de backup.

A política foi mantida seguindo o princípio de:

```text
Least Privilege
```

ou:

```text
Princípio do menor privilégio
```

O usuário IAM deve possuir somente as permissões necessárias para executar sua função.

---

### Resultado

O teste demonstrou que as restrições do IAM estavam funcionando corretamente.

Uma operação não autorizada foi bloqueada pela AWS.

Isso validou, na prática, o controle de permissões configurado para o projeto.

---

## 3. Arquivo `cron.log` inexistente antes da execução

### Sintoma

Durante o teste da automação com Cron, foi configurado o seguinte agendamento temporário:

```cron
52 11 4 9 * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

Antes do horário programado, foi executado:

```bash
cat ~/backup-lab/cron.log
```

O sistema retornou:

```text
cat: /home/rafael/backup-lab/cron.log: No such file or directory
```

---

### Investigação

O arquivo `cron.log` estava sendo criado através do redirecionamento utilizado no próprio Cron:

```text
>> /home/rafael/backup-lab/cron.log
```

Porém, o Cron ainda não havia executado o script.

Portanto, não havia motivo para o arquivo existir naquele momento.

---

### Solução

Foi aguardado o horário definido no agendamento.

Depois da execução programada, o arquivo foi consultado novamente:

```bash
cat ~/backup-lab/cron.log
```

O arquivo passou a existir e continha os registros da execução.

---

### Resultado

O log apresentou:

```text
Iniciando backup...
Origem: /home/rafael/backup-lab/teste
Arquivo: /home/rafael/backup-lab/backup-2026-09-04-115202.tar.gz
Backup local criado com sucesso.
upload: backup-lab/backup-2026-09-04-115202.tar.gz to s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-04-115202.tar.gz
Backup enviado para o S3 com sucesso.
Backup concluído.
Quantidade de backups: 4
Removendo 1 backup(s) antigo(s)...
delete: s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-03-231520.tar.gz
```

Isso confirmou que o Cron executou o script corretamente.

---

## 4. Retenção dos backups

### Sintoma

O script foi configurado para manter somente os três backups mais recentes.

A configuração utilizada foi:

```bash
BACKUP_LIMIT=3
```

Durante os testes, havia quatro backups disponíveis no bucket.

O script identificou:

```text
Quantidade de backups: 4
```

---

### Comportamento esperado

Como o limite definido era de três backups, o script precisava remover um backup antigo.

O cálculo utilizado pelo script é:

```text
Quantidade de backups = 4

Limite = 3

4 - 3 = 1
```

Portanto, um backup deveria ser removido.

---

### Resultado

O script apresentou:

```text
Quantidade de backups: 4
Removendo 1 backup(s) antigo(s)...
```

Em seguida, removeu:

```text
backup-2026-09-03-231520.tar.gz
```

Depois da execução, permaneceram no bucket os três backups mais recentes:

```text
backup-2026-09-03-231550.tar.gz
backup-2026-09-03-231843.tar.gz
backup-2026-09-04-115202.tar.gz
```

---

### Resultado

A política de retenção funcionou conforme planejado.

O script conseguiu:

1. identificar os backups existentes;
2. verificar a quantidade;
3. calcular quantos deveriam ser removidos;
4. excluir os backups mais antigos;
5. manter os três backups mais recentes.

---

## 5. Separação entre arquivos originais e arquivos restaurados

### Situação

Durante o teste de restauração, existiam duas áreas diferentes no Raspberry Pi:

```text
~/backup-lab/teste
```

e:

```text
~/backup-lab/restore/teste
```

A primeira continha os arquivos originais utilizados no teste.

A segunda foi utilizada para restaurar o backup.

---

### Por que essa separação é importante?

Restaurar os arquivos diretamente sobre a pasta original poderia alterar ou sobrescrever os dados utilizados para comparação.

Por isso, foi criada uma pasta específica:

```bash
mkdir -p ~/backup-lab/restore/teste
```

O backup foi baixado para essa pasta:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-04-115202.tar.gz ~/backup-lab/restore/teste/
```

Depois o conteúdo foi extraído nela.

---

### Validação

Os arquivos restaurados foram comparados com os arquivos originais utilizando:

```bash
diff -r ~/backup-lab/teste ~/backup-lab/restore/teste --exclude='*.tar.gz'
```

O comando não apresentou nenhuma saída.

Isso significa que não foram encontradas diferenças entre os arquivos comparados.

---

## 6. Método utilizado para troubleshooting

Durante o desenvolvimento do projeto, o processo de troubleshooting seguiu uma abordagem simples:

```text
Problema
   │
   ▼
Observar a mensagem de erro
   │
   ▼
Identificar o componente envolvido
   │
   ▼
Verificar a configuração
   │
   ▼
Identificar a causa
   │
   ▼
Aplicar a correção
   │
   ▼
Executar novamente o teste
   │
   ▼
Validar o resultado
   │
   ▼
Documentar
```

Essa abordagem evita simplesmente alterar configurações sem entender a causa do problema.

---

## 7. Resumo dos problemas encontrados

| Problema                 | Causa                                          | Solução                                          | Resultado              |
| ------------------------ | ---------------------------------------------- | ------------------------------------------------ | ---------------------- |
| `InvalidClientTokenId`   | Credenciais antigas/inválidas                  | Reconfiguração das credenciais do AWS CLI        | Resolvido              |
| `AccessDenied`           | `s3:GetBucketLocation` não estava autorizado   | Manter a permissão bloqueada por Least Privilege | Comportamento esperado |
| `cron.log` inexistente   | Cron ainda não havia executado                 | Aguardar a execução programada                   | Resolvido              |
| Mais de 3 backups        | Limite de retenção atingido                    | Script removeu o backup mais antigo              | Resolvido              |
| Separação da restauração | Necessidade de preservar os arquivos originais | Utilização de diretório separado                 | Validado               |

---

## 8. Principais aprendizados

Os problemas encontrados permitiram praticar conceitos importantes de infraestrutura.

### AWS CLI

Foi necessário entender de onde o AWS CLI estava obtendo suas credenciais e validar a identidade utilizada.

Comando:

```bash
aws sts get-caller-identity
```

---

### IAM

Foi possível observar na prática a diferença entre:

```text
Autenticação
```

e:

```text
Autorização
```

A autenticação confirmou quem estava acessando a AWS.

A autorização determinou quais ações esse usuário poderia executar.

---

### Amazon S3

Foram realizados testes de:

- listagem;
- upload;
- download;
- exclusão;
- controle de permissões.

---

### Linux

Foram utilizados comandos para:

- navegar entre diretórios;
- criar diretórios;
- listar arquivos;
- compactar;
- extrair arquivos;
- comparar diretórios;
- verificar logs.

---

### Cron

Foi possível configurar uma tarefa automatizada e validar sua execução através de um arquivo de log.

---

### Bash

O script foi responsável por:

- criar o backup;
- compactar os arquivos;
- enviar o arquivo para o S3;
- verificar erros;
- aplicar retenção;
- remover backups antigos.

---

## 9. Resultado final do troubleshooting

Os principais problemas encontrados durante a construção do projeto foram investigados e validados.

O ambiente passou a apresentar o seguinte fluxo:

```text
Raspberry Pi 5
      │
      ▼
backup.sh
      │
      ├── Cria backup
      │
      ├── Compacta arquivos
      │
      ├── Envia para S3
      │
      └── Aplica retenção
              │
              ▼
        Amazon S3
              │
              ▼
          Restauração
              │
              ▼
          Validação
```

A configuração de permissões também foi validada através de uma tentativa de executar uma operação não autorizada.

O resultado `AccessDenied` demonstrou que o IAM estava impedindo uma ação que não fazia parte das permissões necessárias ao projeto.

---

## 10. Conclusão

O troubleshooting realizado durante este projeto permitiu identificar e compreender problemas relacionados a:

- credenciais AWS;
- autenticação;
- autorização IAM;
- permissões S3;
- execução de tarefas com Cron;
- arquivos de log;
- retenção de backups;
- restauração de dados;
- organização de diretórios Linux.

Mais importante do que apenas corrigir os problemas foi entender o motivo de cada comportamento.

O projeto também demonstrou uma prática importante de infraestrutura:

> Antes de alterar uma configuração, é necessário entender o erro, identificar a causa e depois validar a solução.

Com isso, o projeto passou pelas principais etapas:

```text
Preparação
    ↓
IAM
    ↓
S3
    ↓
AWS CLI
    ↓
Script de Backup
    ↓
Retenção
    ↓
Cron
    ↓
Restauração
    ↓
Troubleshooting
```

Com a etapa de troubleshooting concluída, a documentação técnica do projeto está praticamente completa.

---

[Anterior: Restauração de Backups](07-restauracao.md) | [README](../README.md) | [Próximo: Arquitetura](arquitetura.md)
