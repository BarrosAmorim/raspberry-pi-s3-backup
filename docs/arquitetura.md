# Arquitetura — Raspberry Pi 5 + Amazon S3

## 1. Visão geral

Este projeto implementa um sistema de backup automatizado utilizando um Raspberry Pi 5 como servidor Linux e o Amazon S3 como armazenamento remoto.

O objetivo principal é demonstrar, de forma prática, como um servidor Linux pode realizar backups automatizados para a AWS utilizando:

- Linux;
- Bash;
- AWS CLI;
- IAM;
- Amazon S3;
- Cron.

Além do envio dos backups, o projeto também implementa uma política simples de retenção e possui um processo de restauração dos arquivos armazenados.

A arquitetura foi construída utilizando um usuário IAM dedicado ao projeto e seguindo o princípio de menor privilégio.

---

# 2. Arquitetura geral

A arquitetura simplificada do projeto é:

```text
                         AWS CLOUD
                    ┌─────────────────┐
                    │   Amazon S3     │
                    │                 │
                    │  Backups        │
                    │  .tar.gz        │
                    └────────┬────────┘
                             │
                             │
                         AWS CLI
                             │
                             │
                    ┌────────▼────────┐
                    │      IAM        │
                    │                 │
                    │ pi-s3-backup    │
                    └────────┬────────┘
                             │
                             │ autenticação
                             │ autorização
                             │
=============================│================================
                             │
                       INTERNET
                             │
=============================│================================
                             │
                    ┌────────▼────────┐
                    │  Raspberry Pi 5 │
                    │                 │
                    │      Linux      │
                    │                 │
                    │   backup.sh     │
                    └────────┬────────┘
                             │
                             │
                         Arquivos
                         locais
```

O Raspberry Pi executa o processo de backup.

O script `backup.sh` cria um arquivo compactado e utiliza o AWS CLI para enviá-lo para o Amazon S3.

O IAM controla quais operações o usuário utilizado pelo Raspberry Pi pode realizar.

O Amazon S3 é responsável pelo armazenamento dos arquivos de backup.

---

# 3. Componentes da arquitetura

## 3.1 Raspberry Pi 5

O Raspberry Pi 5 funciona como o servidor Linux utilizado neste laboratório.

É nele que ficam:

- os arquivos que serão submetidos ao backup;
- o script `backup.sh`;
- os arquivos `.tar.gz` gerados;
- os arquivos de log;
- a configuração do Cron;
- a configuração local do AWS CLI.

A máquina utilizada possui arquitetura:

```text
aarch64
```

O sistema operacional utilizado é:

```text
Debian GNU/Linux 13 (trixie)
```

O kernel utilizado durante o projeto foi:

```text
Linux 6.18.39+rpt-rpi-2712
```

---

# 4. Sistema operacional Linux

O Linux fornece o ambiente onde todos os componentes locais do projeto são executados.

Entre as funções utilizadas estão:

- gerenciamento de arquivos;
- criação de diretórios;
- execução de scripts;
- compactação;
- extração de arquivos;
- gerenciamento de processos;
- agendamento de tarefas;
- armazenamento dos logs.

Alguns comandos utilizados durante o projeto foram:

```bash
ls
```

```bash
cd
```

```bash
mkdir
```

```bash
tar
```

```bash
diff
```

```bash
systemctl
```

```bash
crontab
```

Esses comandos fazem parte das ferramentas utilizadas para administrar o servidor e validar o funcionamento do sistema.

---

# 5. Diretório utilizado pelo projeto

Durante os testes, foi utilizado o diretório:

```text
~/backup-lab/
```

A estrutura principal utilizada foi:

```text
/home/rafael/backup-lab/
│
├── backup.sh
│
├── backup-2026-09-03-225723.tar.gz
├── backup-2026-09-03-231443.tar.gz
├── backup-2026-09-03-231550.tar.gz
├── backup-2026-09-03-231843.tar.gz
├── backup-2026-09-04-115202.tar.gz
│
├── cron.log
│
├── teste/
│   ├── backup-01.txt
│   ├── backup-02.txt
│   └── backup-03.txt
│
└── restore/
    └── teste/
        ├── backup-01.txt
        ├── backup-02.txt
        ├── backup-03.txt
        └── backup-2026-09-04-115202.tar.gz
```

Os arquivos `.tar.gz` são os backups gerados durante os testes.

A pasta `teste` contém os arquivos utilizados como origem do backup.

A pasta `restore/teste` foi utilizada para testar a restauração sem alterar os arquivos originais.

---

# 6. Script de backup

O componente responsável pelo processo de backup é:

```text
backup.sh
```

Localização utilizada:

```text
/home/rafael/backup-lab/backup.sh
```

O caminho absoluto foi obtido utilizando:

```bash
realpath ~/backup-lab/backup.sh
```

Resultado:

```text
/home/rafael/backup-lab/backup.sh
```

O script realiza várias etapas.

---

# 7. Funcionamento do backup.sh

O fluxo principal do script é:

```text
Início
  │
  ▼
Define diretório de origem
  │
  ▼
Define destino S3
  │
  ▼
Gera timestamp
  │
  ▼
Cria arquivo .tar.gz
  │
  ▼
Verifica se a criação funcionou
  │
  ▼
Envia arquivo para S3
  │
  ▼
Verifica se o upload funcionou
  │
  ▼
Lista backups existentes
  │
  ▼
Verifica quantidade
  │
  ▼
Remove backups antigos
  │
  ▼
Fim
```

Essa sequência permite automatizar todo o processo sem necessidade de executar manualmente cada comando.

---

# 8. Diretório de origem

O diretório utilizado como origem foi:

```bash
SOURCE_DIR="$HOME/backup-lab/teste"
```

Isso corresponde a:

```text
/home/rafael/backup-lab/teste
```

Dentro dele estavam os arquivos:

```text
backup-01.txt
backup-02.txt
backup-03.txt
```

Esses arquivos foram utilizados para validar o funcionamento do sistema.

---

# 9. Criação do arquivo de backup

O script utiliza o comando:

```bash
tar -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .
```

O comando cria um arquivo compactado no formato:

```text
.tar.gz
```

Por exemplo:

```text
backup-2026-09-04-115202.tar.gz
```

A opção `-C` faz com que o `tar` entre no diretório de origem antes de adicionar os arquivos.

O ponto:

```text
.
```

representa o conteúdo do diretório atual.

Dessa forma, todo o conteúdo existente dentro da pasta de origem é incluído no backup.

---

# 10. Identificação dos backups

Para evitar sobrescrever backups anteriores, o script utiliza um timestamp:

```bash
TIMESTAMP=$(date +"%Y-%m-%d-%H%M%S")
```

O nome do arquivo é então construído utilizando esse timestamp:

```bash
BACKUP_FILE="$HOME/backup-lab/backup-$TIMESTAMP.tar.gz"
```

O resultado possui o seguinte padrão:

```text
backup-AAAA-MM-DD-HHMMSS.tar.gz
```

Exemplo real utilizado no projeto:

```text
backup-2026-09-04-115202.tar.gz
```

Essa estratégia permite identificar aproximadamente quando cada backup foi criado.

---

# 11. Amazon S3

O Amazon S3 é utilizado como armazenamento remoto dos backups.

O bucket utilizado no projeto é:

```text
raspberry-pi-s3-backup-lab-2026
```

A região utilizada é:

```text
us-east-1
```

O caminho lógico utilizado para os backups é:

```text
s3://raspberry-pi-s3-backup-lab-2026/backup/
```

A estrutura fica:

```text
Amazon S3
│
└── raspberry-pi-s3-backup-lab-2026
    │
    └── backup/
        │
        ├── backup-2026-09-03-231550.tar.gz
        ├── backup-2026-09-03-231843.tar.gz
        └── backup-2026-09-04-115202.tar.gz
```

---

# 12. AWS CLI

O AWS CLI é a ferramenta utilizada pelo Raspberry Pi para conversar com os serviços da AWS.

A versão utilizada durante o projeto foi:

```text
aws-cli/2.23.6
```

O AWS CLI permite executar operações no S3 diretamente pelo terminal Linux.

Por exemplo, para listar os objetos:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Para enviar um arquivo:

```bash
aws s3 cp arquivo.tar.gz s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Para baixar um arquivo:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/arquivo.tar.gz .
```

Para excluir um objeto:

```bash
aws s3 rm s3://raspberry-pi-s3-backup-lab-2026/backup/arquivo.tar.gz
```

---

# 13. IAM

O IAM é responsável pelo controle de identidade e permissões na AWS.

Para este projeto foi criado um usuário IAM específico:

```text
pi-s3-backup
```

ARN:

```text
arn:aws:iam::696537703431:user/pi-s3-backup
```

O Raspberry Pi utiliza as credenciais desse usuário para acessar o S3.

---

# 14. Princípio do menor privilégio

O usuário `pi-s3-backup` não possui permissões administrativas.

A política utilizada concede somente as operações necessárias para o funcionamento do backup.

As permissões são:

```text
s3:ListBucket
s3:PutObject
s3:GetObject
s3:DeleteObject
```

A divisão é:

```text
Bucket
│
└── s3:ListBucket

Objetos
│
├── s3:PutObject
├── s3:GetObject
└── s3:DeleteObject
```

Essa configuração segue o princípio de:

```text
Least Privilege
```

ou:

```text
Menor Privilégio
```

A ideia é conceder somente as permissões necessárias para que o sistema execute sua função.

---

# 15. Validação das permissões

Durante o projeto foi realizada uma tentativa de executar uma operação que não estava autorizada:

```bash
aws s3api get-bucket-location --bucket raspberry-pi-s3-backup-lab-2026
```

A AWS retornou:

```text
AccessDenied
```

O erro indicou que o usuário não possuía:

```text
s3:GetBucketLocation
```

Essa permissão não foi adicionada porque não era necessária para o funcionamento do sistema.

Esse teste demonstrou na prática que o IAM estava bloqueando uma operação que não fazia parte das permissões definidas.

---

# 16. Autenticação

A autenticação do AWS CLI foi validada utilizando:

```bash
aws sts get-caller-identity
```

O comando confirmou a identidade utilizada:

```text
arn:aws:iam::696537703431:user/pi-s3-backup
```

Isso permitiu confirmar que o Raspberry Pi estava utilizando o usuário IAM dedicado ao projeto.

---

# 17. Cron

O Cron é responsável por executar automaticamente o script de backup.

O serviço foi verificado com:

```bash
systemctl status cron --no-pager
```

O serviço estava:

```text
Active: active (running)
```

Isso confirmou que o serviço de agendamento estava funcionando.

---

# 18. Agendamento automático

O agendamento permanente configurado no Raspberry Pi foi:

```cron
0 3 * * * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

Essa configuração significa:

```text
0
│
└── minuto 0

3
│
└── hora 3

*
│
└── qualquer dia do mês

*
│
└── qualquer mês

*
│
└── qualquer dia da semana
```

Portanto:

```text
Todos os dias às 03:00
```

o Cron executará:

```text
/home/rafael/backup-lab/backup.sh
```

---

# 19. Logs

A saída do script é redirecionada para:

```text
/home/rafael/backup-lab/cron.log
```

A configuração utiliza:

```text
>> /home/rafael/backup-lab/cron.log
```

para adicionar novas execuções ao final do arquivo.

O:

```text
2>&1
```

faz com que mensagens de erro também sejam direcionadas para o mesmo arquivo.

Assim, o log recebe tanto a saída normal quanto os erros do processo.

---

# 20. Exemplo de execução registrada

Durante o teste real do Cron, o log apresentou:

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

Esse resultado demonstrou que o processo automatizado conseguiu:

1. iniciar o backup;
2. criar o arquivo local;
3. enviar o arquivo para o S3;
4. identificar a quantidade de backups;
5. aplicar a retenção;
6. remover um backup antigo.

---

# 21. Política de retenção

O script possui uma política simples de retenção:

```bash
BACKUP_LIMIT=3
```

Isso significa que o sistema deve manter somente os três backups mais recentes.

Quando existem mais de três backups, o script calcula quantos precisam ser removidos.

Exemplo:

```text
Backups encontrados: 4
Limite: 3

4 - 3 = 1
```

Portanto:

```text
1 backup antigo
```

deve ser removido.

Durante o teste real, isso aconteceu.

O script identificou:

```text
Quantidade de backups: 4
Removendo 1 backup(s) antigo(s)...
```

E removeu:

```text
backup-2026-09-03-231520.tar.gz
```

Depois permaneceram os três backups mais recentes.

---

# 22. Fluxo do backup

O fluxo completo de criação e armazenamento do backup é:

```text
                 RASPBERRY PI 5

        Arquivos em ~/backup-lab/teste
                       │
                       ▼
                  backup.sh
                       │
                       ▼
                  comando tar
                       │
                       ▼
              arquivo .tar.gz
                       │
                       ▼
                    AWS CLI
                       │
                       ▼
                     IAM
                       │
                 autorização
                       │
                       ▼
                  Amazon S3
                       │
                       ▼
              backup armazenado
                       │
                       ▼
                Retenção = 3
                       │
                       ▼
            Remove backups antigos
```

---

# 23. Fluxo de restauração

A restauração funciona no sentido inverso.

Primeiro o backup é localizado no S3.

Depois ele é baixado para o Raspberry Pi.

Em seguida o arquivo é extraído.

Finalmente os arquivos restaurados podem ser comparados com os arquivos originais.

O fluxo é:

```text
Amazon S3
    │
    ▼
AWS CLI
    │
    │ download
    ▼
arquivo .tar.gz
    │
    ▼
tar -xzf
    │
    ▼
arquivos restaurados
    │
    ▼
diff
    │
    ▼
validação
```

---

# 24. Teste de restauração

O backup utilizado no teste de restauração foi:

```text
backup-2026-09-04-115202.tar.gz
```

Foi criada a pasta:

```text
~/backup-lab/restore/teste
```

O arquivo foi baixado do S3 utilizando:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-04-115202.tar.gz ~/backup-lab/restore/teste/
```

Depois foi realizada a extração:

```bash
tar -xzf ~/backup-lab/restore/teste/backup-2026-09-04-115202.tar.gz -C ~/backup-lab/restore/teste/
```

Os arquivos restaurados foram:

```text
backup-01.txt
backup-02.txt
backup-03.txt
```

---

# 25. Validação da restauração

Para comparar os arquivos originais com os arquivos restaurados foi utilizado:

```bash
diff -r ~/backup-lab/teste ~/backup-lab/restore/teste --exclude='*.tar.gz'
```

O comando não apresentou nenhuma saída.

Isso significa que não foram encontradas diferenças entre os arquivos comparados.

Dessa forma, o processo de restauração foi validado dentro do cenário de teste utilizado.

---

# 26. Fluxo completo do projeto

A arquitetura completa pode ser representada da seguinte forma:

```text
                         ┌─────────────────────┐
                         │     Amazon S3       │
                         │                     │
                         │  Backup .tar.gz     │
                         └──────────▲──────────┘
                                    │
                                    │
                              AWS CLI│
                                    │
                         ┌──────────┴──────────┐
                         │        IAM          │
                         │                     │
                         │    pi-s3-backup     │
                         │                     │
                         │  Least Privilege    │
                         └──────────▲──────────┘
                                    │
                                    │
                              autenticação
                                    │
                                    │
                         ┌──────────┴──────────┐
                         │   Raspberry Pi 5    │
                         │                     │
                         │       Linux         │
                         │                     │
                         │      Cron           │
                         │        │            │
                         │        ▼            │
                         │   backup.sh         │
                         │        │            │
                         │        ▼            │
                         │      tar.gz         │
                         │        │            │
                         │        ▼            │
                         │   Arquivos locais   │
                         └─────────────────────┘
```

---

# 27. Ciclo de vida do backup

O ciclo de vida implementado no projeto é:

```text
1. Arquivos existem no Raspberry Pi
              │
              ▼
2. Cron executa backup.sh
              │
              ▼
3. backup.sh cria arquivo .tar.gz
              │
              ▼
4. AWS CLI envia arquivo para S3
              │
              ▼
5. S3 armazena o backup
              │
              ▼
6. Script verifica quantidade de backups
              │
              ▼
7. Backups antigos são removidos
              │
              ▼
8. Backup pode ser baixado posteriormente
              │
              ▼
9. Arquivo é extraído
              │
              ▼
10. Dados são restaurados
              │
              ▼
11. Arquivos podem ser validados
```

---

# 28. Segurança da arquitetura

A segurança do projeto foi baseada em alguns princípios simples.

## Usuário IAM dedicado

Foi utilizado:

```text
pi-s3-backup
```

em vez de utilizar o usuário Root.

---

## Menor privilégio

O usuário possui somente as permissões necessárias:

```text
s3:ListBucket
s3:PutObject
s3:GetObject
s3:DeleteObject
```

---

## Credenciais fora do código

As credenciais utilizadas pelo AWS CLI não fazem parte do código do projeto.

Nenhuma Access Key ou Secret Access Key deve ser publicada no GitHub.

---

## Bucket privado

O bucket utilizado no projeto possui o bloqueio de acesso público habilitado.

O armazenamento foi configurado com:

```text
Block Public Access
```

e:

```text
Bucket owner enforced
```

As ACLs foram desabilitadas.

---

## Criptografia

O bucket utiliza:

```text
SSE-S3
```

com:

```text
Bucket Key
```

habilitado.

Isso fornece criptografia no lado do servidor para os objetos armazenados.

---

# 29. O que acontece quando o Cron executa?

Quando chega o horário configurado:

```text
03:00
```

o Cron executa:

```text
/home/rafael/backup-lab/backup.sh
```

O script começa a execução:

```text
backup.sh
    │
    ▼
Define origem
    │
    ▼
Define destino S3
    │
    ▼
Gera timestamp
    │
    ▼
Cria .tar.gz
    │
    ▼
Upload para S3
    │
    ▼
Verifica backups
    │
    ▼
Aplica retenção
    │
    ▼
Finaliza
```

Toda a execução é registrada em:

```text
/home/rafael/backup-lab/cron.log
```

---

# 30. O que acontece se o upload falhar?

O script possui uma validação depois do comando de upload.

A estrutura utilizada é:

```bash
aws s3 cp "$BACKUP_FILE" "$S3_DESTINATION"

if [ $? -ne 0 ]; then
    echo "ERRO: falha ao enviar o backup para o S3."
    exit 1
fi
```

O código:

```bash
$?
```

representa o código de saída do comando executado anteriormente.

Quando:

```text
$? = 0
```

o comando anterior foi executado com sucesso.

Quando o valor é diferente de zero:

```text
$? != 0
```

o script considera que houve erro.

Nesse caso, o script apresenta uma mensagem de erro e encerra a execução.

---

# 31. O que acontece se a criação do backup falhar?

O mesmo princípio é utilizado na criação do arquivo:

```bash
tar -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .

if [ $? -ne 0 ]; then
    echo "ERRO: falha ao criar o backup."
    exit 1
fi
```

Se o `tar` falhar, o script:

```text
1. identifica o erro;
2. apresenta uma mensagem;
3. encerra a execução;
4. não continua para o upload.
```

Isso evita enviar para o S3 um backup que não foi criado corretamente.

---

# 32. Relação entre os componentes

Cada componente possui uma responsabilidade específica.

| Componente     | Responsabilidade                     |
| -------------- | ------------------------------------ |
| Raspberry Pi 5 | Servidor onde o processo é executado |
| Linux          | Sistema operacional                  |
| Bash           | Linguagem utilizada no script        |
| `backup.sh`    | Automatiza o processo de backup      |
| `tar`          | Compacta os arquivos                 |
| Cron           | Agenda a execução                    |
| AWS CLI        | Comunicação com a AWS                |
| IAM            | Controle de identidade e permissões  |
| Amazon S3      | Armazenamento dos backups            |
| `cron.log`     | Registro das execuções               |
| `diff`         | Comparação dos arquivos restaurados  |

Essa separação de responsabilidades torna a arquitetura simples de entender e manter.

---

# 33. Tecnologias utilizadas

O projeto utiliza:

```text
Hardware
└── Raspberry Pi 5

Sistema operacional
└── Debian GNU/Linux 13

Shell
└── Bash

Automação
└── Cron

Compactação
└── tar / gzip

Cloud
├── AWS IAM
├── Amazon S3
└── AWS CLI

Validação
└── diff

Versionamento
└── Git
```

---

# 34. Estrutura planejada do repositório

O repositório foi organizado da seguinte maneira:

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

A pasta `docs` concentra a documentação técnica.

A pasta `scripts` concentra os scripts utilizados pelo projeto.

A pasta `logs` possui um arquivo `.gitkeep` para manter o diretório no repositório.

---

# 35. Visão de infraestrutura

Do ponto de vista de infraestrutura, o projeto pode ser dividido em duas partes.

## Ambiente local

```text
Raspberry Pi 5
│
├── Linux
├── Bash
├── Cron
├── AWS CLI
├── backup.sh
└── Arquivos
```

## Ambiente AWS

```text
AWS
│
├── IAM
│   └── pi-s3-backup
│
└── Amazon S3
    └── raspberry-pi-s3-backup-lab-2026
```

A comunicação entre os dois ambientes ocorre através do AWS CLI.

---

# 36. Fluxo de comunicação

O fluxo de comunicação pode ser representado assim:

```text
Raspberry Pi 5
      │
      │
      │ AWS CLI
      ▼
   AWS API
      │
      ▼
     IAM
      │
      │ verifica identidade
      │ verifica permissões
      ▼
     S3
      │
      ▼
Objeto armazenado
```

O IAM não armazena os backups.

Sua função é controlar a identidade e as permissões utilizadas para acessar os recursos da AWS.

O armazenamento propriamente dito é realizado pelo Amazon S3.

---

# 37. Benefícios da arquitetura

A arquitetura escolhida apresenta algumas vantagens.

### Automação

O backup não depende de execução manual.

O Cron executa o script automaticamente.

### Armazenamento remoto

Os backups são enviados para o Amazon S3.

Isso permite manter uma cópia fora do armazenamento local do Raspberry Pi.

### Controle de acesso

O IAM controla as ações permitidas ao usuário utilizado pelo backup.

### Retenção

O script evita o crescimento indefinido da quantidade de backups no bucket.

### Restauração

O processo permite baixar e extrair um backup posteriormente.

### Baixa complexidade

A solução utiliza ferramentas relativamente simples:

```text
Linux
+
Bash
+
Cron
+
AWS CLI
+
S3
```

Isso torna o projeto adequado para estudar fundamentos de infraestrutura e DevOps.

---

# 38. Limitações do projeto

Apesar de funcionar para o objetivo deste laboratório, a solução possui algumas limitações.

A retenção é controlada pelo próprio script Bash.

Isso significa que a lógica depende da execução correta do script.

O projeto também utiliza credenciais do AWS CLI configuradas no Raspberry Pi.

Em um ambiente de produção, poderiam ser avaliadas outras formas de autenticação e gerenciamento de credenciais, dependendo da arquitetura adotada.

Além disso, este projeto foi desenvolvido como laboratório de estudo e não pretende substituir uma solução corporativa completa de backup.

---

# 39. Possíveis evoluções

O projeto pode ser evoluído futuramente.

Algumas possibilidades são:

```text
Projeto atual
     │
     ├── Backup automático
     ├── Retenção
     ├── Restauração
     └── Troubleshooting
              │
              ▼
       Possíveis melhorias
              │
              ├── Criptografia adicional
              ├── Monitoramento
              ├── Alertas
              ├── Validação automática
              ├── S3 Lifecycle
              ├── CloudWatch
              ├── Infraestrutura como código
              └── CI/CD
```

Essas melhorias podem ser implementadas em etapas futuras conforme o objetivo de aprendizado.

---

# 40. Conhecimentos demonstrados

Este projeto permitiu praticar conhecimentos relacionados a:

## Linux

- navegação no sistema;
- gerenciamento de arquivos;
- diretórios;
- permissões;
- processos;
- serviços;
- logs.

## Bash

- variáveis;
- comandos;
- condições;
- códigos de saída;
- automação;
- manipulação de arquivos.

## AWS

- IAM;
- S3;
- AWS CLI;
- autenticação;
- autorização;
- políticas;
- permissões.

## Automação

- Cron;
- execução programada;
- redirecionamento de logs;
- retenção automática.

## Troubleshooting

- análise de mensagens de erro;
- identificação de causas;
- validação de configurações;
- testes;
- documentação de problemas.

---

# 41. Resultado final da arquitetura

Ao final do projeto, o Raspberry Pi possui um processo automatizado capaz de:

```text
1. Localizar os arquivos de origem
          ↓
2. Criar um arquivo compactado
          ↓
3. Enviar o backup para o Amazon S3
          ↓
4. Manter somente os backups mais recentes
          ↓
5. Registrar a execução em log
          ↓
6. Permitir o download do backup
          ↓
7. Extrair os arquivos
          ↓
8. Validar os arquivos restaurados
```

A arquitetura utiliza componentes com responsabilidades bem definidas:

```text
Cron
 │
 ▼
backup.sh
 │
 ▼
tar
 │
 ▼
AWS CLI
 │
 ▼
IAM
 │
 ▼
Amazon S3
```

E o caminho de restauração é:

```text
Amazon S3
 │
 ▼
AWS CLI
 │
 ▼
.tar.gz
 │
 ▼
tar
 │
 ▼
Arquivos restaurados
 │
 ▼
diff
```

---

# 42. Conclusão

O projeto demonstra a construção de uma solução de backup automatizado utilizando um Raspberry Pi 5 como servidor Linux e o Amazon S3 como armazenamento remoto.

A arquitetura combina ferramentas fundamentais de infraestrutura:

```text
Linux
Bash
Cron
AWS CLI
IAM
Amazon S3
```

O Raspberry Pi é responsável pela execução do processo.

O Bash automatiza as tarefas.

O Cron agenda a execução.

O AWS CLI realiza a comunicação com a AWS.

O IAM controla a identidade e as permissões.

O Amazon S3 armazena os backups.

A solução também possui retenção de três backups e um processo de restauração validado durante o laboratório.

Um dos principais aprendizados foi perceber que uma solução de backup não termina no momento em que o arquivo é enviado para o armazenamento.

É necessário também verificar:

```text
O backup foi criado?
        ↓
O upload funcionou?
        ↓
O backup está armazenado?
        ↓
Os backups antigos estão sendo controlados?
        ↓
É possível baixar o arquivo?
        ↓
É possível restaurar os dados?
        ↓
Os dados restaurados estão corretos?
```

Esse ciclo representa uma visão mais próxima do trabalho real de infraestrutura e DevOps.

O projeto, portanto, não demonstra apenas a utilização do Amazon S3, mas a construção de um pequeno fluxo de infraestrutura automatizada, com autenticação, controle de acesso, armazenamento, automação, retenção, restauração e troubleshooting.

---

[Anterior: Troubleshooting](08-troubleshooting.md) | [README](../README.md)
