# Raspberry Pi 5 → AWS S3 Backup

Solução de **backup automatizado de arquivos de um Raspberry Pi 5 para o Amazon S3**, utilizando Linux, Bash, AWS CLI, IAM e Cron.

> Projeto prático desenvolvido para aplicar conceitos de **Cloud, Linux, automação, segurança e backup** em um ambiente real.

---

## Navegação

- [Sobre o projeto](#sobre-o-projeto)
- [Objetivos](#objetivos)
- [Arquitetura](#arquitetura)
- [Tecnologias utilizadas](#tecnologias-utilizadas)
- [Estrutura do projeto](#estrutura-do-projeto)
- [Documentação](#documentação)
- [Resultados](#resultados)
- [Segurança](#segurança)
- [O que estou aprendendo](#o-que-estou-aprendendo)
- [Status do projeto](#status-do-projeto)
- [Conclusão](#conclusão)

---

## Sobre o projeto

Este projeto consiste na implementação de uma solução de **backup automatizado de arquivos de um Raspberry Pi 5 para o Amazon S3**.

A proposta é utilizar o Raspberry Pi como ambiente Linux e integrar recursos da AWS para criar uma rotina de backup com armazenamento remoto, controle de permissões, automação e possibilidade de restauração.

O projeto foi desenvolvido como um laboratório prático para transformar conhecimentos teóricos em prática, trabalhando com **Linux, AWS, Bash, AWS CLI, IAM, Amazon S3 e automação**.

Durante a implementação, também foram realizados testes de autenticação, acesso ao bucket, permissões IAM, envio de arquivos, retenção de backups e restauração.

---

## Objetivos

- Criar uma rotina de backup automatizado.
- Armazenar os backups no Amazon S3.
- Utilizar o AWS IAM para controle de permissões.
- Utilizar a AWS CLI para interação com os serviços AWS.
- Automatizar a execução dos backups utilizando Cron.
- Implementar uma política de retenção dos backups.
- Testar o processo de restauração dos arquivos.
- Registrar os procedimentos e comandos utilizados durante a implementação.
- Documentar problemas encontrados e suas respectivas soluções.

---

## Arquitetura

```text
┌──────────────────────────────┐
│         Raspberry Pi 5       │
│                              │
│            Linux             │
│                              │
│       Script de Backup       │
└──────────────┬───────────────┘
               │
               │ AWS CLI
               ▼
┌──────────────────────────────┐
│             IAM              │
│                              │
│      Controle de acesso      │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│          Amazon S3           │
│                              │
│           Backups            │
└──────────────────────────────┘
```

### Fluxo do backup

```text
Arquivos do Raspberry Pi
          │
          ▼
Script de Backup
          │
          ▼
Criação do arquivo .tar.gz
          │
          ▼
AWS CLI
          │
          ▼
IAM valida as permissões
          │
          ▼
Amazon S3
          │
          ▼
Retenção dos backups
```

### Fluxo da restauração

```text
Amazon S3
    │
    ▼
AWS CLI
    │
    ▼
Download do backup
    │
    ▼
Arquivo .tar.gz
    │
    ▼
Extração dos arquivos
    │
    ▼
Arquivos restaurados
```

---

## Tecnologias utilizadas

- **Raspberry Pi 5**
- **Linux**
- **Bash**
- **Git**
- **GitHub**
- **AWS CLI**
- **AWS IAM**
- **Amazon S3**
- **Cron**

---

## Estrutura do projeto

```text
raspberry-pi-s3-backup/
│
├── .gitignore
├── README.md
│
└── docs/
    ├── 01-preparacao-raspberry.md
    ├── 02-configuracao-iam.md
    ├── 03-configuracao-s3.md
    ├── 04-configuracao-aws-cli.md
    ├── 05-script-backup-retencao.md
    ├── 06-automatizacao-cron.md
    ├── 07-restauracao.md
    ├── 08-troubleshooting.md
    └── arquitetura.md
```

---

## Documentação

A implementação foi documentada passo a passo, incluindo **comandos utilizados no terminal, configurações realizadas, testes, resultados e procedimentos de troubleshooting**.

| Etapa | Documentação                                                                        |
| ----- | ----------------------------------------------------------------------------------- |
| 01    | [Preparação do Raspberry Pi](docs/01-preparacao-raspberry.md)                       |
| 02    | [Configuração do IAM](docs/02-configuracao-iam.md)                                  |
| 03    | [Configuração do S3](docs/03-configuracao-s3.md)                                    |
| 04    | [Configuração da AWS CLI](docs/04-configuracao-aws-cli.md)                          |
| 05    | [Desenvolvimento do script de backup e retenção](docs/05-script-backup-retencao.md) |
| 06    | [Automatização com Cron](docs/06-automatizacao-cron.md)                             |
| 07    | [Processo de restauração](docs/07-restauracao.md)                                   |
| 08    | [Troubleshooting](docs/08-troubleshooting.md)                                       |
| 09    | [Arquitetura do projeto](docs/arquitetura.md)                                       |

---

## Resultados

Os principais testes do projeto foram realizados com sucesso.

| Validação                         | Resultado |
| --------------------------------- | --------- |
| Preparação do Raspberry Pi        | Concluída |
| Configuração do IAM               | Concluída |
| Configuração do Amazon S3         | Concluída |
| Configuração da AWS CLI           | Concluída |
| Autenticação na AWS               | Validada  |
| Envio de backup para o S3         | Validado  |
| Download de backup do S3          | Validado  |
| Restauração dos arquivos          | Validada  |
| Política de retenção              | Validada  |
| Automatização com Cron            | Validada  |
| Teste de permissão não autorizada | Validado  |
| Documentação técnica              | Concluída |

### Retenção de backups

Foi configurada uma política para manter somente os **3 backups mais recentes**.

Durante os testes, o script identificou 4 backups no bucket e removeu o mais antigo.

Resultado final:

```text
backup-2026-09-03-231550.tar.gz
backup-2026-09-03-231843.tar.gz
backup-2026-09-04-115202.tar.gz
```

### Restauração

Foi realizado o download do backup mais recente e a extração dos arquivos.

A comparação entre os arquivos originais e os restaurados não apresentou diferenças:

```bash
diff -r ~/backup-lab/teste ~/backup-lab/restore/teste --exclude='*.tar.gz'
```

Esse resultado confirmou que os arquivos comparados foram restaurados corretamente.

### Automatização

O Cron foi configurado para executar o backup diariamente às **03:00**:

```cron
0 3 * * * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

O teste temporário do Cron também foi executado com sucesso, gerando o arquivo de log e realizando o backup automaticamente.

---

## Segurança

O projeto utiliza um usuário IAM dedicado:

```text
pi-s3-backup
```

As permissões foram configuradas seguindo o princípio de **Least Privilege**, concedendo somente as ações necessárias para o funcionamento do backup:

```text
s3:ListBucket
s3:PutObject
s3:GetObject
s3:DeleteObject
```

As credenciais não fazem parte do código-fonte e não devem ser publicadas no GitHub.

Também foi realizado um teste com uma operação não autorizada:

```bash
aws s3api get-bucket-location --bucket raspberry-pi-s3-backup-lab-2026
```

O comando retornou `AccessDenied`, demonstrando que o usuário não possui permissão para executar essa ação.

---

## O que estou aprendendo

Este projeto está sendo utilizado para transformar conhecimentos teóricos em prática, explorando:

- Administração de sistemas Linux.
- Gerenciamento de permissões e segurança com IAM.
- Armazenamento de objetos com Amazon S3.
- Utilização da AWS CLI.
- Automação com Bash.
- Agendamento de tarefas com Cron.
- Estratégias de retenção de backups.
- Processos de restauração.
- Troubleshooting.
- Documentação técnica.
- Versionamento com Git e GitHub.

---

## Status do projeto

**Concluído — primeira versão funcional.**

O Raspberry Pi consegue realizar backups para o Amazon S3, a retenção foi validada, a restauração foi testada e a execução automática com Cron foi configurada.

---

## Conclusão

Este projeto permitiu colocar em prática conceitos de **Linux, AWS, IAM, S3, Bash, AWS CLI, Cron, backup, restauração e troubleshooting**.

Além de configurar os serviços, foi possível testar o funcionamento das permissões, automatizar a execução e validar a recuperação dos arquivos.

O objetivo principal foi transformar o estudo em prática e construir uma solução que pudesse ser documentada e apresentada como projeto de aprendizado em **Cloud e DevOps**.

---

[Documentação](docs/01-preparacao-raspberry.md) | [Arquitetura](docs/arquitetura.md)
