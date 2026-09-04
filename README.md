# Raspberry Pi 5 → AWS S3 Backup

Solução de **backup automatizado de arquivos de um Raspberry Pi 5 para o Amazon S3**, utilizando Linux, Bash, AWS CLI, IAM e Cron.

> Projeto prático desenvolvido para aplicar conceitos de **Cloud, Linux, automação, segurança e backup** em um ambiente real.

---

## 📚 Navegação

- [Sobre o projeto](#-sobre-o-projeto)
- [Objetivos](#-objetivos)
- [Arquitetura](#-arquitetura)
- [Tecnologias utilizadas](#-tecnologias-utilizadas)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Documentação](#-documentação)
- [Status do projeto](#-status-do-projeto)
- [O que estou aprendendo](#-o-que-estou-aprendendo)

---

## 📌 Sobre o projeto

Este projeto consiste na implementação de uma solução de **backup automatizado de arquivos de um Raspberry Pi 5 para o Amazon S3**.

A proposta é utilizar o Raspberry Pi como ambiente Linux e integrar recursos da AWS para criar uma rotina de backup segura, automatizada e com possibilidade de restauração.

O projeto está sendo desenvolvido como um laboratório prático para transformar conhecimentos teóricos em prática, trabalhando com **Linux, AWS, Bash, AWS CLI, IAM, Amazon S3 e automação**.

---

## 🎯 Objetivos

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

## 🏗️ Arquitetura

```text
┌──────────────────────┐
│    Raspberry Pi 5    │
│                      │
│       Linux          │
│                      │
│   Script de Backup   │
└──────────┬───────────┘
           │
           │ AWS CLI
           ▼
┌──────────────────────┐
│         IAM          │
│                      │
│ Controle de acesso   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│      Amazon S3       │
│                      │
│      Backups         │
└──────────────────────┘
```

---

## 🛠️ Tecnologias utilizadas

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

## 📂 Estrutura do projeto

```text
raspberry-pi-s3-backup/
│
├── .gitignore
├── README.md
│
├── docs/
│   ├── 01-preparacao-raspberry.md
│   ├── 02-configuracao-iam.md
│   ├── 03-configuracao-s3.md
│   ├── 04-configuracao-aws-cli.md
│   ├── 05-script-backup-retencao.md
│   ├── 06-automatizacao-cron.md
│   ├── 07-restauracao.md
│   ├── 08-troubleshooting.md
│   └── arquitetura.md
│
├── scripts/
│   ├── backup.sh
│   └── restore.sh
│
└── logs/
    └── .gitkeep
```

---

## 📚 Documentação

A implementação será documentada passo a passo, incluindo **comandos utilizados no terminal, configurações realizadas, testes, resultados e procedimentos de troubleshooting**.

| Etapa | Documentação                                                    |
| ----- | --------------------------------------------------------------- |
| 01    | [Preparação do Raspberry Pi](docs/01-preparacao-raspberry.md)   |
| 02    | [Configuração do IAM](docs/02-configuracao-iam.md)              |
| 03    | [Configuração do S3](docs/03-configuracao-s3.md)                |
| 04    | [Configuração da AWS CLI](docs/04-configuracao-aws-cli.md)      |
| 05    | [Desenvolvimento do script de backup](docs/05-script-backup.md) |
| 06    | [Implementação da retenção](docs/06-retencao.md)                |
| 07    | [Automatização com Cron](docs/07-automatizacao-cron.md)         |
| 08    | [Processo de restauração](docs/08-restauracao.md)               |
| 09    | [Troubleshooting](docs/09-troubleshooting.md)                   |

---

## 🚧 Status do projeto

**Em desenvolvimento.**

O projeto está sendo implementado de forma incremental, com testes realizados a cada etapa e versionamento utilizando Git.

---

## 📖 O que estou aprendendo

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

## 📌 Resultado

Esta seção será atualizada ao final do projeto com os resultados dos testes realizados, incluindo:

- [ ] Backup realizado com sucesso.
- [ ] Arquivos armazenados no Amazon S3.
- [ ] Política de retenção funcionando.
- [ ] Backup automatizado.
- [ ] Processo de restauração testado.
- [ ] Documentação concluída.
