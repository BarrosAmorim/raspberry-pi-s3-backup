# Configuração do Amazon S3

## Objetivo

Nesta etapa foi criado e configurado o bucket Amazon S3 que será utilizado para armazenar os backups do Raspberry Pi 5.

O bucket foi configurado com foco em segurança, controle de acesso e utilização exclusiva pelo projeto.

A arquitetura utilizada é:

```text
Raspberry Pi 5
      │
      │ AWS CLI
      ▼
Usuário IAM
      │
      │ Permissões específicas
      ▼
Amazon S3
      │
      ▼
raspberry-pi-s3-backup-lab-2026
```

---

## 1. Criar o bucket S3

No console da AWS:

**Amazon S3 → Buckets → Create bucket**

Foi criado o bucket:

```text
raspberry-pi-s3-backup-lab-2026
```

O nome do bucket é globalmente único dentro do Amazon S3.

A região utilizada foi:

```text
us-east-1
```

### Por que escolher uma região específica?

A escolha da região determina onde os dados do bucket serão armazenados e também influencia custos e latência.

Neste projeto foi utilizada a região `us-east-1`, que também está configurada no AWS CLI do Raspberry Pi.

---

## 2. Bloqueio de acesso público

Na configuração do bucket foi mantida a opção de bloquear todo acesso público.

Foi utilizada a configuração:

```text
Block all public access
```

### Por que bloquear o acesso público?

Os backups podem conter arquivos importantes do sistema.

Não existe nenhuma necessidade de tornar esses arquivos acessíveis pela internet.

A arquitetura desejada é:

```text
Internet
   │
   X
   │
Amazon S3
   │
   └── Acesso autorizado pelo IAM
           │
           └── Raspberry Pi
```

Dessa forma, o acesso ao bucket ocorre por meio das credenciais e permissões IAM configuradas para o projeto.

---

## 3. ACLs

O bucket foi configurado com as ACLs desabilitadas.

Foi utilizada a configuração baseada em:

```text
Bucket owner enforced
```

### O que são ACLs?

ACL significa **Access Control List**.

As ACLs são um mecanismo antigo de controle de acesso do S3.

Neste projeto, o controle de acesso é realizado principalmente por meio das políticas IAM.

Por isso, não foi necessário utilizar ACLs para controlar os arquivos do backup.

---

## 4. Versionamento

O versionamento do bucket foi mantido:

```text
Disabled
```

### O que é versionamento?

O versionamento permite manter diferentes versões do mesmo objeto.

Por exemplo:

```text
backup.tar.gz
     │
     ├── versão 1
     ├── versão 2
     └── versão 3
```

Neste projeto, o sistema de retenção será responsável por manter uma quantidade definida de backups.

Por isso, inicialmente o versionamento não foi habilitado.

---

## 5. Criptografia do lado do servidor

Foi habilitada a criptografia:

```text
Server-side encryption
SSE-S3
```

O Amazon S3 utiliza chaves gerenciadas pelo próprio serviço para realizar a criptografia dos objetos armazenados.

### Por que utilizar criptografia?

O backup pode conter informações importantes do servidor.

A criptografia adiciona uma camada de proteção aos dados armazenados no S3.

O fluxo fica:

```text
Raspberry Pi
     │
     │ Backup
     ▼
Amazon S3
     │
     │ Criptografia SSE-S3
     ▼
Objeto armazenado
```

---

## 6. S3 Bucket Key

A opção:

```text
Bucket Key
```

foi habilitada.

O Bucket Key pode reduzir a quantidade de solicitações necessárias ao AWS KMS quando a criptografia utiliza SSE-KMS.

Neste projeto foi utilizado **SSE-S3**, portanto essa configuração não é o ponto principal da segurança adotada.

Ela foi mantida conforme a configuração apresentada pelo S3 durante a criação do bucket.

---

## 7. Estado inicial do bucket

Após a criação, o bucket estava vazio.

A estrutura inicial era:

```text
raspberry-pi-s3-backup-lab-2026/
└── vazio
```

Os arquivos de backup serão adicionados posteriormente pelo script executado no Raspberry Pi.

---

# Testes de acesso pelo Raspberry Pi

Depois da configuração do bucket e do IAM, foram realizados testes utilizando o AWS CLI no Raspberry Pi 5.

O objetivo foi verificar se o usuário IAM possuía realmente as permissões necessárias para trabalhar com o bucket.

---

## 8. Testar listagem do bucket

Foi executado:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/
```

Como o bucket estava vazio naquele momento, nenhum arquivo foi exibido.

O comando foi executado sem retornar erro de permissão.

Isso confirmou que o usuário possuía acesso para listar o conteúdo do bucket.

Permissão relacionada:

```text
s3:ListBucket
```

---

## 9. Testar upload

Foi criado um arquivo de teste no Raspberry Pi:

```bash
echo "Teste de backup Raspberry Pi" > ~/teste-backup.txt
```

Depois foi realizado o upload:

```bash
aws s3 cp ~/teste-backup.txt s3://raspberry-pi-s3-backup-lab-2026/
```

O upload foi concluído com sucesso.

O arquivo também foi visualizado no console do Amazon S3.

Isso confirmou que o Raspberry Pi consegue enviar objetos para o bucket.

Permissão utilizada:

```text
s3:PutObject
```

---

## 10. Testar download

Para testar a recuperação de um objeto, foi criada uma pasta:

```bash
mkdir -p ~/teste-restore
```

Depois foi executado:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/teste-backup.txt ~/teste-restore/
```

O arquivo foi baixado com sucesso.

Isso confirmou que o Raspberry Pi consegue recuperar objetos armazenados no bucket.

Permissão utilizada:

```text
s3:GetObject
```

---

## 11. Testar exclusão

Por último, foi testada a exclusão do objeto:

```bash
aws s3 rm s3://raspberry-pi-s3-backup-lab-2026/teste-backup.txt
```

O comando foi executado com sucesso.

O arquivo também foi conferido no console do S3 e não estava mais presente.

Isso confirmou que o usuário possui permissão para excluir objetos.

Permissão utilizada:

```text
s3:DeleteObject
```

Essa permissão será importante posteriormente para implementar a política de retenção dos backups.

---

# Resultado dos testes

Os testes realizados confirmaram que o Raspberry Pi consegue executar as operações necessárias no bucket.

| Operação        | Permissão         | Resultado |
| --------------- | ----------------- | --------- |
| Listar objetos  | `s3:ListBucket`   | OK        |
| Enviar arquivo  | `s3:PutObject`    | OK        |
| Baixar arquivo  | `s3:GetObject`    | OK        |
| Excluir arquivo | `s3:DeleteObject` | OK        |

---

# Configuração final do bucket

Ao final desta etapa, o bucket ficou configurado da seguinte forma:

```text
Nome:
raspberry-pi-s3-backup-lab-2026

Região:
us-east-1

Acesso público:
Bloqueado

ACLs:
Desabilitadas

Versionamento:
Desabilitado

Criptografia:
SSE-S3

Bucket Key:
Habilitada
```

O bucket está pronto para receber os backups automatizados do Raspberry Pi 5.

---

# Conclusão

A configuração do Amazon S3 foi concluída e validada.

O bucket foi criado com acesso público bloqueado e criptografia habilitada.

Também foram realizados testes reais utilizando o Raspberry Pi 5, comprovando as operações de:

```text
Listagem
   ↓
Upload
   ↓
Download
   ↓
Exclusão
```

Com o S3 e o IAM funcionando corretamente, a próxima etapa será configurar e validar o **AWS CLI no Raspberry Pi 5** de forma documentada para utilização no script de backup.

---

[Anterior: Configuração IAM](02-configuracao-iam.md) | [README](../README.md) | [Próximo: Configuração AWS CLI](04-configuracao-aws-cli.md)
