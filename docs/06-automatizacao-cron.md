# 06 — Automatização com Cron

## Objetivo

Nesta etapa, o objetivo foi automatizar a execução do script de backup utilizando o **Cron**, permitindo que o Raspberry Pi execute o backup automaticamente sem necessidade de execução manual.

O script utilizado foi:

```text
/home/rafael/backup-lab/backup.sh
```

O fluxo implementado foi:

```text
Cron
  ↓
backup.sh
  ↓
Criação do arquivo .tar.gz
  ↓
AWS CLI
  ↓
Amazon S3
  ↓
Retenção dos 3 backups mais recentes
```

---

## 1. Verificação do serviço Cron

Antes de configurar a tarefa, foi verificado se o serviço Cron estava instalado, habilitado e em execução.

Comando utilizado:

```bash
systemctl status cron --no-pager
```

O resultado confirmou:

```text
cron.service - Regular background program processing daemon
Loaded: loaded
Active: active (running)
```

Isso confirmou que o serviço Cron estava funcionando no Raspberry Pi.

---

## 2. Verificação do caminho do script

Foi utilizado o comando `realpath` para obter o caminho absoluto do script:

```bash
realpath ~/backup-lab/backup.sh
```

Resultado:

```text
/home/rafael/backup-lab/backup.sh
```

O caminho absoluto é importante no Cron porque o ambiente de execução do Cron é diferente de uma execução manual no terminal.

---

## 3. Configuração inicial para teste

Antes de criar a programação definitiva, foi configurado um horário específico para testar se o Cron conseguiria executar o backup automaticamente.

Foi aberto o crontab do usuário:

```bash
crontab -e
```

Durante o teste foi utilizada a seguinte configuração:

```cron
52 11 4 9 * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

Essa configuração significa:

```text
52   → minuto
11   → hora
4    → dia do mês
9    → mês de setembro
*    → qualquer dia da semana
```

Portanto, a tarefa foi programada para:

```text
04/09/2026 às 11:52
```

O resultado da execução também foi direcionado para:

```text
/home/rafael/backup-lab/cron.log
```

A parte:

```text
>> /home/rafael/backup-lab/cron.log
```

faz com que a saída do script seja adicionada ao arquivo de log.

Já:

```text
2>&1
```

redireciona as mensagens de erro para o mesmo arquivo.

Dessa forma, tanto a saída normal quanto os erros ficam registrados no mesmo log.

---

## 4. Verificação do Crontab

Após salvar a configuração, foi utilizado:

```bash
crontab -l
```

O resultado confirmou que a tarefa estava registrada:

```cron
52 11 4 9 * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

---

## 5. Verificação antes do horário programado

Antes das 11:52 foi feita uma tentativa de consultar o arquivo de log:

```bash
cat ~/backup-lab/cron.log
```

Como o Cron ainda não havia executado a tarefa, o arquivo ainda não existia.

Resultado:

```text
cat: /home/rafael/backup-lab/cron.log: No such file or directory
```

Esse comportamento era esperado, pois o arquivo seria criado quando o Cron executasse o comando.

---

## 6. Validação da execução automática

Após o horário programado, o arquivo de log foi consultado novamente:

```bash
cat ~/backup-lab/cron.log
```

Resultado real:

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
delete: s3://raspberry-pi-s3-backup-lab-2026/09-03-231520.tar.gz
```

O resultado confirmou que o Cron executou o script automaticamente.

Também foi possível confirmar que:

- o backup foi criado;
- o arquivo foi enviado para o S3;
- a política de retenção foi executada;
- um backup antigo foi removido.

---

## 7. Verificação dos backups no S3

Depois da execução automática, foi verificado o conteúdo do bucket:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Resultado:

```text
2026-09-03 23:15:52        200 backup-2026-09-03-231550.tar.gz
2026-09-03 23:18:45        200 backup-2026-09-03-231843.tar.gz
2026-09-04 11:52:04        200 backup-2026-09-04-115202.tar.gz
```

A listagem confirmou que permaneceram somente os **3 backups mais recentes**.

Portanto, o teste comprovou a integração completa:

```text
Cron
 ↓
Script Bash
 ↓
AWS CLI
 ↓
IAM
 ↓
Amazon S3
 ↓
Retenção
```

---

## 8. Configuração permanente

Depois que o teste foi concluído com sucesso, a configuração temporária foi substituída por uma programação permanente.

Foi aberto novamente o crontab:

```bash
crontab -e
```

A configuração de teste:

```cron
52 11 4 9 * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

foi substituída por:

```cron
0 3 * * * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

Essa configuração significa:

```text
0    → minuto 00
3    → 03 horas
*    → qualquer dia
*    → qualquer mês
*    → qualquer dia da semana
```

Portanto:

> O Raspberry Pi executará o backup automaticamente todos os dias às 03:00.

---

## 9. Verificação da configuração permanente

Após salvar a configuração permanente, foi utilizado:

```bash
crontab -l
```

A tarefa permanente foi registrada no Crontab.

Configuração final:

```cron
0 3 * * * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

---

## 10. Como consultar o log

Sempre que for necessário verificar o resultado da execução automática:

```bash
cat ~/backup-lab/cron.log
```

Para acompanhar o log em tempo real:

```bash
tail -f ~/backup-lab/cron.log
```

Para sair do `tail`, pressione:

```text
Ctrl + C
```

---

## 11. Como verificar os backups no S3

Para verificar os backups armazenados:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Para contar quantos backups existem:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/ | grep 'backup-.*\.tar\.gz$' | wc -l
```

O projeto foi configurado para manter somente:

```text
3 backups
```

---

## 12. Troubleshooting

### O arquivo `cron.log` não existe

Comando:

```bash
cat ~/backup-lab/cron.log
```

Se aparecer:

```text
No such file or directory
```

verifique primeiro se o horário programado já passou.

O arquivo de log é criado quando o comando do Cron é executado pela primeira vez.

---

### Verificar se o Cron está funcionando

```bash
systemctl status cron --no-pager
```

O serviço deve apresentar:

```text
Active: active (running)
```

---

### Verificar a programação

```bash
crontab -l
```

A configuração permanente esperada é:

```cron
0 3 * * * /home/rafael/backup-lab/backup.sh >> /home/rafael/backup-lab/cron.log 2>&1
```

---

### Verificar os backups

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

---

## 13. Conceitos praticados

Nesta etapa foram praticados:

- Cron;
- Crontab;
- agendamento de tarefas Linux;
- execução automática de scripts Bash;
- redirecionamento de saída;
- redirecionamento de erros;
- arquivos de log;
- execução de AWS CLI pelo Cron;
- automação de backup;
- retenção de backups;
- integração Linux + AWS.

---

## Conclusão

A automatização do backup foi implementada utilizando o Cron do Linux.

Primeiramente foi realizado um teste programado para confirmar o funcionamento da automação. O teste foi executado com sucesso, criando o backup, enviando o arquivo para o Amazon S3 e executando corretamente a política de retenção.

Após a validação, a tarefa temporária foi substituída por uma programação permanente para execução diária às 03:00.

A solução final permite que o Raspberry Pi realize o processo de backup automaticamente, sem necessidade de execução manual.

---

[Anterior: Configuração da AWS CLI](04-configuracao-aws-cli.md) | [README](../README.md) | [Próximo: Processo de restauração](07-restauracao.md)
