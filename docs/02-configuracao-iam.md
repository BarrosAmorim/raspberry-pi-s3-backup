# Configuração do IAM

## Objetivo

Nesta etapa foi configurado o acesso seguro do Raspberry Pi 5 ao Amazon S3.

A ideia foi criar um usuário IAM específico para o projeto, concedendo somente as permissões necessárias para:

- listar o conteúdo do bucket;
- enviar backups;
- baixar arquivos para restauração;
- excluir backups antigos.

O acesso não foi configurado com permissões administrativas.

---

## 1. Criar o grupo IAM

A primeira etapa foi criar um grupo específico para o projeto.

No console da AWS:

**IAM → User groups → Create group**

Foi criado o grupo:

```text
pi-s3-backup
```

### Por que utilizar um grupo?

O grupo permite centralizar as permissões.

Em vez de colocar políticas diretamente no usuário, a estrutura utilizada foi:

```text
Grupo IAM
└── pi-s3-backup
       │
       └── Política
           RaspberryPiS3BackupPolicy
               │
               └── Usuário do projeto
```

Isso facilita a administração das permissões e permite adicionar outros usuários ao grupo futuramente, caso seja necessário.

---

## 2. Criar o usuário IAM

Depois foi criado um usuário IAM específico para o projeto.

Esse usuário foi associado ao grupo:

```text
pi-s3-backup
```

A intenção é que o Raspberry Pi utilize esse usuário exclusivamente para executar as operações necessárias no S3.

### Princípio do menor privilégio

Não foi utilizada uma política administrativa como `AdministratorAccess`.

O usuário recebeu somente as permissões necessárias para trabalhar com o bucket utilizado pelo projeto.

---

## 3. Criar a política personalizada

Foi criada uma política IAM personalizada chamada:

```text
RaspberryPiS3BackupPolicy
```

No console:

**IAM → Policies → Create policy**

Foi escolhida a opção:

**JSON**

A política utilizada foi:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListarBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::raspberry-pi-s3-backup-lab-2026"
    },
    {
      "Sid": "GerenciarBackups",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::raspberry-pi-s3-backup-lab-2026/*"
    }
  ]
}
```

### Entendendo a política

A política possui dois níveis de acesso.

#### Acesso ao bucket

```json
"s3:ListBucket"
```

Permite listar os objetos existentes no bucket.

O recurso utilizado é:

```text
arn:aws:s3:::raspberry-pi-s3-backup-lab-2026
```

#### Acesso aos objetos

As seguintes ações foram permitidas:

```text
s3:PutObject
s3:GetObject
s3:DeleteObject
```

**`s3:PutObject`**

Permite enviar arquivos para o S3.

No projeto, será utilizado para enviar os backups do Raspberry Pi.

**`s3:GetObject`**

Permite obter/baixar arquivos armazenados no S3.

No projeto, será utilizado para o processo de restauração.

**`s3:DeleteObject`**

Permite excluir objetos do bucket.

No projeto, será utilizado posteriormente para a política de retenção, removendo backups antigos.

As permissões dos objetos estão limitadas a:

```text
arn:aws:s3:::raspberry-pi-s3-backup-lab-2026/*
```

O `/*` representa os objetos existentes dentro desse bucket.

---

## 4. Associar a política ao grupo

Depois de criada a política, ela foi associada ao grupo:

```text
pi-s3-backup
```

No console:

**IAM → User groups → pi-s3-backup → Permissions → Add permissions → Attach policies**

Foi selecionada:

```text
RaspberryPiS3BackupPolicy
```

O resultado ficou:

```text
pi-s3-backup
└── RaspberryPiS3BackupPolicy
```

---

## 5. Criar uma Access Key para o AWS CLI

Para permitir que o Raspberry Pi utilize o AWS CLI, foi criada uma chave de acesso para o usuário IAM.

No usuário:

**IAM → Users → [usuário do projeto] → Security credentials**

Na seção **Access keys**, foi selecionado:

**Create access key**

Na escolha do caso de uso foi selecionado:

**Command Line Interface (CLI)**

A Access Key e a Secret Access Key foram geradas e armazenadas de forma segura.

> Nunca coloque a Secret Access Key em arquivos do projeto, GitHub, README ou mensagens públicas.

---

## 6. Configurar o AWS CLI no Raspberry Pi 5

Com as credenciais criadas, foi configurado o AWS CLI no Raspberry Pi.

Comando:

```bash
aws configure
```

O comando solicita:

```text
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Foi utilizada a região:

```text
us-east-1
```

E o formato de saída:

```text
json
```

As credenciais foram inseridas diretamente no terminal.

---

## 7. Validar a autenticação

Depois da configuração, foi executado:

```bash
aws sts get-caller-identity
```

Esse comando consulta o AWS Security Token Service e retorna a identidade associada às credenciais utilizadas pelo AWS CLI.

A resposta confirmou que o Raspberry Pi estava autenticando como o usuário IAM do projeto:

```text
arn:aws:iam::...:user/pi-s3-backup
```

Isso confirmou que as novas credenciais estavam funcionando.

---

## 8. Testar a permissão de listagem

Foi executado:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/
```

Como o bucket estava vazio naquele momento, nenhum arquivo foi exibido.

O comando, porém, foi executado sem apresentar erro de `AccessDenied`.

Isso confirmou que o usuário possuía permissão para listar o conteúdo do bucket.

---

## 9. Testar `s3:PutObject`

Foi criado um arquivo de teste:

```bash
echo "Teste de backup Raspberry Pi" > ~/teste-backup.txt
```

Depois foi realizado o upload:

```bash
aws s3 cp ~/teste-backup.txt s3://raspberry-pi-s3-backup-lab-2026/
```

O upload foi concluído com sucesso.

O arquivo também foi conferido diretamente no console do Amazon S3.

Isso confirmou o funcionamento da permissão:

```text
s3:PutObject
```

---

## 10. Testar `s3:GetObject`

Foi criada uma pasta para receber o arquivo restaurado:

```bash
mkdir -p ~/teste-restore
```

Depois foi realizado o download:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/teste-backup.txt ~/teste-restore/
```

O download foi concluído com sucesso.

Isso confirmou o funcionamento da permissão:

```text
s3:GetObject
```

---

## 11. Testar `s3:DeleteObject`

Por fim, foi testada a exclusão do objeto:

```bash
aws s3 rm s3://raspberry-pi-s3-backup-lab-2026/teste-backup.txt
```

O arquivo foi removido com sucesso.

A remoção também foi conferida diretamente no bucket S3.

Isso confirmou o funcionamento da permissão:

```text
s3:DeleteObject
```

---

## 12. Resultado da validação

Ao final dos testes, todas as permissões necessárias foram validadas:

| Permissão         | Função                   | Resultado |
| ----------------- | ------------------------ | --------- |
| `s3:ListBucket`   | Listar objetos do bucket | OK        |
| `s3:PutObject`    | Enviar arquivos          | OK        |
| `s3:GetObject`    | Baixar arquivos          | OK        |
| `s3:DeleteObject` | Excluir arquivos         | OK        |

A autenticação do AWS CLI também foi validada com:

```bash
aws sts get-caller-identity
```

O acesso do Raspberry Pi ao bucket S3 está funcionando corretamente.

---

## 13. Troubleshooting — credenciais antigas

Antes da criação das novas credenciais, o AWS CLI do Raspberry Pi possuía credenciais utilizadas em um projeto anterior.

Ao executar:

```bash
aws sts get-caller-identity
```

foi retornado:

```text
An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation:
The security token included in the request is invalid.
```

O problema ocorreu porque as credenciais antigas já não eram válidas.

A solução foi criar uma nova Access Key para o usuário IAM utilizado neste projeto e executar novamente:

```bash
aws configure
```

Após a configuração das novas credenciais, o comando:

```bash
aws sts get-caller-identity
```

passou a identificar corretamente o usuário:

```text
user/pi-s3-backup
```

Esse problema demonstrou a importância de validar as credenciais antes de iniciar a automação do backup.

---

## Conclusão

A configuração do IAM foi concluída e testada.

O Raspberry Pi 5 possui agora um usuário IAM dedicado ao projeto, com permissões limitadas ao bucket:

```text
raspberry-pi-s3-backup-lab-2026
```

As operações de listagem, upload, download e exclusão foram executadas com sucesso.

A próxima etapa será documentar e finalizar a configuração do bucket Amazon S3.

---

[Anterior: Preparação do Raspberry Pi](01-preparacao-raspberry.md) | [README](../README.md) | [Próximo: Configuração do S3](03-configuracao-s3.md)
