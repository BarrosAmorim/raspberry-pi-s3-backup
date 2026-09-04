# 05 — Script de Backup

## Objetivo

Nesta etapa foi desenvolvido e testado um script Bash responsável por realizar o backup de arquivos do Raspberry Pi 5 para um bucket Amazon S3.

O objetivo foi criar um processo que pudesse:

- selecionar uma pasta de origem;
- compactar os arquivos em `.tar.gz`;
- gerar um nome de backup utilizando data e hora;
- enviar o backup para o Amazon S3;
- verificar se o upload foi realizado;
- manter somente os 3 backups mais recentes;
- permitir o download e a restauração do backup.

O script foi desenvolvido e executado diretamente no Raspberry Pi 5.

---

# 1. Preparação do ambiente de testes

Para não utilizar arquivos grandes durante o laboratório, foi criada uma pequena estrutura de testes.

A pasta utilizada foi:

```text
/home/rafael/backup-lab/
```

E dentro dela:

```text
backup-lab/
└── teste/
    ├── backup-01.txt
    ├── backup-02.txt
    └── backup-03.txt
```

A utilização de arquivos pequenos permitiu testar o processo sem gerar armazenamento desnecessário no S3.

---

# 2. Criação dos arquivos de teste

Inicialmente foi criada a pasta de testes:

```bash
mkdir -p ~/backup-lab/teste
```

Também foi criado um primeiro arquivo de teste:

```bash
echo "Arquivo de teste do backup" > ~/backup-lab/teste/exemplo.txt
```

Posteriormente, para testar o backup de uma pasta contendo vários arquivos, esse arquivo foi removido:

```bash
cd ~/backup-lab/teste
rm -f exemplo.txt
```

Foram então criados três arquivos pequenos:

```bash
echo "Backup de teste 01" > backup-01.txt
echo "Backup de teste 02" > backup-02.txt
echo "Backup de teste 03" > backup-03.txt
```

A estrutura passou a ser:

```text
teste/
├── backup-01.txt
├── backup-02.txt
└── backup-03.txt
```

---

# 3. Teste inicial de upload para o S3

Antes de criar o script, foi realizado um teste manual utilizando o AWS CLI.

O objetivo era confirmar que uma pasta poderia ser enviada recursivamente para o bucket.

### Comando

```bash
aws s3 cp ~/backup-lab/teste/ s3://raspberry-pi-s3-backup-lab-2026/backup/teste/ --recursive
```

### O que o comando faz?

O comando:

```text
aws s3 cp
```

realiza uma cópia entre o sistema local e o Amazon S3.

A opção:

```text
--recursive
```

faz com que todos os arquivos encontrados dentro da pasta sejam enviados.

Neste teste, os três arquivos foram enviados para:

```text
s3://raspberry-pi-s3-backup-lab-2026/backup/teste/
```

---

# 4. Verificação dos arquivos no S3

Depois do upload, foi realizada a verificação:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/teste/
```

Resultado obtido:

```text
2026-09-03 22:48:28         19 backup-01.txt
2026-09-03 22:48:28         19 backup-02.txt
2026-09-03 22:48:28         19 backup-03.txt
```

Os três arquivos foram armazenados corretamente no S3.

---

# 5. Teste de download e restauração de um arquivo

Foi realizado também um teste de recuperação.

Primeiro foi criada uma pasta para restauração:

```bash
mkdir -p ~/backup-lab/restore
```

Depois, um arquivo foi baixado do S3:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/teste/backup-01.txt ~/backup-lab/restore/
```

O arquivo foi baixado com sucesso.

Para verificar seu conteúdo:

```bash
cat ~/backup-lab/restore/backup-01.txt
```

Resultado:

```text
Backup de teste 01
```

Esse teste confirmou que o arquivo armazenado no S3 podia ser recuperado.

---

# 6. Criação do script

Depois dos testes manuais, foi criado o arquivo responsável pelo backup automatizado:

```bash
touch ~/backup-lab/backup.sh
```

A existência do arquivo foi verificada:

```bash
ls -lh ~/backup-lab/backup.sh
```

Resultado inicial:

```text
-rw-rw-r-- 1 rafael rafael 0 Sep  3 22:53 /home/rafael/backup-lab/backup.sh
```

---

# 7. Shebang do script

O arquivo recebeu a primeira linha:

```bash
echo '#!/bin/bash' > ~/backup-lab/backup.sh
```

O conteúdo foi conferido:

```bash
cat ~/backup-lab/backup.sh
```

Resultado:

```text
#!/bin/bash
```

### O que significa `#!/bin/bash`?

Essa linha informa ao sistema que o arquivo deve ser interpretado pelo Bash.

Ela é chamada de **shebang**.

---

# 8. Script de backup

O script completo utilizado no laboratório foi:

```bash
#!/bin/bash

SOURCE_DIR="$HOME/backup-lab/teste"
S3_DESTINATION="s3://raspberry-pi-s3-backup-lab-2026/backup/"

TIMESTAMP=$(date +"%Y-%m-%d-%H%M%S")
BACKUP_FILE="$HOME/backup-lab/backup-$TIMESTAMP.tar.gz"

echo "Iniciando backup..."
echo "Origem: $SOURCE_DIR"
echo "Arquivo: $BACKUP_FILE"

tar -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .

if [ $? -ne 0 ]; then
    echo "ERRO: falha ao criar o backup."
    exit 1
fi

echo "Backup local criado com sucesso."

aws s3 cp "$BACKUP_FILE" "$S3_DESTINATION"

if [ $? -ne 0 ]; then
    echo "ERRO: falha ao enviar o backup para o S3."
    exit 1
fi

echo "Backup enviado para o S3 com sucesso."
echo "Backup concluído."

# Retenção: manter somente os 3 backups mais recentes

BACKUP_LIMIT=3

BACKUPS=$(aws s3 ls "$S3_DESTINATION" | sort | grep 'backup-.*\.tar\.gz$')
BACKUP_COUNT=$(echo "$BACKUPS" | grep -c 'backup-.*\.tar\.gz$')

if [ "$BACKUP_COUNT" -gt "$BACKUP_LIMIT" ]; then
    DELETE_COUNT=$((BACKUP_COUNT - BACKUP_LIMIT))

    echo "Quantidade de backups: $BACKUP_COUNT"
    echo "Removendo $DELETE_COUNT backup(s) antigo(s)..."

    echo "$BACKUPS" | head -n "$DELETE_COUNT" | while read -r DATE TIME SIZE FILE; do
        aws s3 rm "${S3_DESTINATION}${FILE}"
    done
fi
```

---

# 9. Entendendo as variáveis

## `SOURCE_DIR`

```bash
SOURCE_DIR="$HOME/backup-lab/teste"
```

Define a pasta que será utilizada como origem do backup.

No Raspberry Pi, `$HOME` corresponde ao diretório pessoal do usuário.

Como o usuário utilizado no laboratório é `rafael`:

```text
$HOME
↓
/home/rafael
```

Portanto:

```text
$HOME/backup-lab/teste
```

corresponde a:

```text
/home/rafael/backup-lab/teste
```

---

## `S3_DESTINATION`

```bash
S3_DESTINATION="s3://raspberry-pi-s3-backup-lab-2026/backup/"
```

Define o destino dos backups no Amazon S3.

---

## `TIMESTAMP`

```bash
TIMESTAMP=$(date +"%Y-%m-%d-%H%M%S")
```

Obtém a data e hora atual do Raspberry Pi.

O formato utilizado é:

```text
AAAA-MM-DD-HHMMSS
```

Por exemplo:

```text
2026-09-03-231843
```

Isso permite criar nomes diferentes para cada backup.

---

## `BACKUP_FILE`

```bash
BACKUP_FILE="$HOME/backup-lab/backup-$TIMESTAMP.tar.gz"
```

Define o nome e o local do arquivo compactado.

Um exemplo real gerado durante o teste foi:

```text
/home/rafael/backup-lab/backup-2026-09-03-225723.tar.gz
```

---

# 10. Criação do arquivo compactado

O comando responsável pela compactação é:

```bash
tar -czf "$BACKUP_FILE" -C "$SOURCE_DIR" .
```

### Significado das opções

```text
tar
```

Ferramenta utilizada para criar um arquivo que reúne vários arquivos.

```text
-c
```

Cria um novo arquivo.

```text
-z
```

Utiliza gzip para compactação.

```text
-f
```

Indica o nome do arquivo que será criado.

```text
-C "$SOURCE_DIR"
```

Faz o comando trabalhar a partir da pasta de origem.

```text
.
```

Indica todos os arquivos da pasta atual.

O resultado é um único arquivo:

```text
backup-2026-09-03-225723.tar.gz
```

---

# 11. Verificação da criação do backup

O script verifica se o comando `tar` foi executado corretamente:

```bash
if [ $? -ne 0 ]; then
    echo "ERRO: falha ao criar o backup."
    exit 1
fi
```

O `$?` representa o código de saída do último comando executado.

De forma simplificada:

```text
0      → sucesso
diferente de 0 → erro
```

Se a criação do backup falhar, o script encerra.

---

# 12. Upload para o Amazon S3

Depois da criação do arquivo, o script executa:

```bash
aws s3 cp "$BACKUP_FILE" "$S3_DESTINATION"
```

Esse comando utiliza o AWS CLI para enviar o arquivo `.tar.gz` para o S3.

A autenticação é feita utilizando o usuário IAM configurado anteriormente:

```text
pi-s3-backup
```

---

# 13. Verificação do upload

Após o upload, o script verifica novamente o código de saída:

```bash
if [ $? -ne 0 ]; then
    echo "ERRO: falha ao enviar o backup para o S3."
    exit 1
fi
```

Se o upload falhar, o script informa o erro e encerra.

Se funcionar:

```text
Backup enviado para o S3 com sucesso.
```

---

# 14. Permissão de execução

Depois que o script foi criado, foi necessário torná-lo executável:

```bash
chmod +x ~/backup-lab/backup.sh
```

A permissão foi verificada com:

```bash
ls -lh ~/backup-lab/backup.sh
```

Resultado:

```text
-rwxrwxr-x 1 rafael rafael 659 Sep  3 22:55 /home/rafael/backup-lab/backup.sh
```

A presença do `x` indica que o arquivo possui permissão de execução.

---

# 15. Primeiro teste do script

O script foi executado diretamente:

```bash
~/backup-lab/backup.sh
```

Resultado real:

```text
Iniciando backup...
Origem: /home/rafael/backup-lab/teste
Arquivo: /home/rafael/backup-lab/backup-2026-09-03-225723.tar.gz
Backup local criado com sucesso.
upload: ./backup-2026-09-03-225723.tar.gz to s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-03-225723.tar.gz
Backup enviado para o S3 com sucesso.
Backup concluído.
```

O primeiro backup criado pelo script foi:

```text
backup-2026-09-03-225723.tar.gz
```

---

# 16. Verificação do backup no S3

Foi utilizado:

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Resultado:

```text
2026-09-03 22:57:25        200 backup-2026-09-03-225723.tar.gz
```

O backup foi armazenado corretamente no S3.

---

# 17. Verificação do arquivo local

Também foi verificado o arquivo criado no Raspberry Pi:

```bash
ls -lh ~/backup-lab/backup-*.tar.gz
```

Resultado:

```text
-rw-rw-r-- 1 rafael rafael 200 Sep  3 22:57 /home/rafael/backup-lab/backup-2026-09-03-225723.tar.gz
```

O arquivo local possui 200 bytes, o mesmo tamanho apresentado no S3.

---

# 18. Verificação do conteúdo do backup

Para verificar o conteúdo do arquivo compactado sem extraí-lo, foi utilizado:

```bash
tar -tzf ~/backup-lab/backup-2026-09-03-225723.tar.gz
```

Resultado:

```text
./
./backup-02.txt
./backup-01.txt
./backup-03.txt
```

Isso confirmou que os três arquivos de teste estavam dentro do backup.

---

# 19. Teste completo de restauração

Para testar a recuperação do backup completo, foi criada uma pasta:

```bash
mkdir -p ~/backup-lab/restore-completo
```

O backup foi baixado do S3:

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-03-225723.tar.gz ~/backup-lab/restore-completo/
```

O download foi concluído com sucesso.

Depois, o arquivo foi extraído:

```bash
tar -xzf ~/backup-lab/restore-completo/backup-2026-09-03-225723.tar.gz -C ~/backup-lab/restore-completo/
```

Os arquivos restaurados foram verificados:

```bash
ls -lh ~/backup-lab/restore-completo/
```

Resultado:

```text
total 16K
-rw-rw-r-- 1 rafael rafael  19 Sep  3 22:45 backup-01.txt
-rw-rw-r-- 1 rafael rafael  19 Sep  3 22:45 backup-02.txt
-rw-rw-r-- 1 rafael rafael  19 Sep  3 22:45 backup-03.txt
-rw-rw-r-- 1 rafael rafael 200 Sep  3 22:57 backup-2026-09-03-225723.tar.gz
```

### Resultado

O processo de recuperação foi validado:

```text
S3
 ↓
Download do .tar.gz
 ↓
Extração
 ↓
Arquivos restaurados
```

Portanto, o projeto possui tanto o processo de **backup** quanto o processo de **download e restauração** funcionando.

---

# 20. Implementação da retenção

Foi definido que o bucket deve manter somente os **3 backups mais recentes**.

A variável utilizada é:

```bash
BACKUP_LIMIT=3
```

Primeiro o script lista os backups:

```bash
BACKUPS=$(aws s3 ls "$S3_DESTINATION" | sort | grep 'backup-.*\.tar\.gz$')
```

Depois conta quantos existem:

```bash
BACKUP_COUNT=$(echo "$BACKUPS" | grep -c 'backup-.*\.tar\.gz$')
```

Se houver mais de três:

```bash
if [ "$BACKUP_COUNT" -gt "$BACKUP_LIMIT" ]; then
```

o script calcula quantos precisam ser removidos:

```bash
DELETE_COUNT=$((BACKUP_COUNT - BACKUP_LIMIT))
```

Os backups mais antigos são identificados através da ordenação:

```bash
sort
```

e selecionados com:

```bash
head -n "$DELETE_COUNT"
```

A remoção é feita utilizando:

```bash
aws s3 rm
```

---

# 21. Teste da retenção

Foram criados backups adicionais para simular um cenário real de retenção.

Em determinado momento existiam quatro backups:

```text
backup-2026-09-03-225723.tar.gz
backup-2026-09-03-231443.tar.gz
backup-2026-09-03-231520.tar.gz
backup-2026-09-03-231550.tar.gz
```

Depois foi executado novamente o script.

O script criou um quinto backup:

```text
backup-2026-09-03-231843.tar.gz
```

E identificou:

```text
Quantidade de backups: 5
Removendo 2 backup(s) antigo(s)...
```

Foram removidos:

```text
backup-2026-09-03-225723.tar.gz
backup-2026-09-03-231443.tar.gz
```

Depois da execução, foram confirmados somente três backups no S3.

### Resultado

A política de retenção funcionou conforme planejado:

```text
5 backups
   ↓
remove 2 mais antigos
   ↓
3 backups restantes
```

---

# 22. Limpeza dos arquivos de teste no S3

Durante os testes iniciais havia uma pasta:

```text
backup/teste/
```

Essa pasta foi utilizada apenas para validar o upload manual dos arquivos.

Depois que os testes foram concluídos, ela foi removida:

```bash
aws s3 rm s3://raspberry-pi-s3-backup-lab-2026/backup/teste/ --recursive
```

O bucket passou a conter somente os backups compactados gerados pelo script.

---

# 23. Fluxo final do script

O funcionamento desenvolvido nesta etapa pode ser representado por:

```text
Raspberry Pi 5
      │
      ▼
Pasta de origem
      │
      ▼
backup.sh
      │
      ├── identifica data/hora
      │
      ├── cria .tar.gz
      │
      ├── verifica criação
      │
      ├── envia para S3
      │
      ├── verifica upload
      │
      └── aplica retenção
              │
              ▼
       mantém 3 backups
```

---

# 24. Comandos principais para treinamento

Para repetir o laboratório posteriormente, os principais comandos utilizados nesta etapa são:

### Criar arquivos de teste

```bash
mkdir -p ~/backup-lab/teste
```

```bash
echo "Backup de teste 01" > ~/backup-lab/teste/backup-01.txt
echo "Backup de teste 02" > ~/backup-lab/teste/backup-02.txt
echo "Backup de teste 03" > ~/backup-lab/teste/backup-03.txt
```

### Criar o script

```bash
touch ~/backup-lab/backup.sh
```

### Visualizar o script

```bash
cat ~/backup-lab/backup.sh
```

### Dar permissão de execução

```bash
chmod +x ~/backup-lab/backup.sh
```

### Executar

```bash
~/backup-lab/backup.sh
```

### Verificar backups no S3

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

### Verificar conteúdo do `.tar.gz`

```bash
tar -tzf ~/backup-lab/backup-*.tar.gz
```

### Baixar um backup

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/NOME-DO-BACKUP.tar.gz ~/backup-lab/restore-completo/
```

### Extrair um backup

```bash
tar -xzf ~/backup-lab/restore-completo/NOME-DO-BACKUP.tar.gz -C ~/backup-lab/restore-completo/
```

---

# 25. Resultado da etapa

Ao final desta etapa, o projeto possui um script Bash funcional capaz de:

- criar backups compactados;
- utilizar data e hora no nome dos arquivos;
- enviar os backups para o Amazon S3;
- verificar falhas durante a criação e o upload;
- recuperar backups armazenados no S3;
- restaurar os arquivos originais;
- manter somente os 3 backups mais recentes.

O processo de backup e restauração foi testado com sucesso no Raspberry Pi 5.

A próxima etapa será configurar a execução automática do script utilizando **Cron**, eliminando a necessidade de executar o backup manualmente.

---

[Anterior: Configuração do AWS CLI](04-configuracao-aws-cli.md) | [README](../README.md) | [Próximo: Automatização do Cron](06-automatizacao-cron.md)
