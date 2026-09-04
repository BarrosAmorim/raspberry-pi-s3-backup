# 07 — Restauração de Backups

## Objetivo

Nesta etapa foi realizado um teste de restauração de um backup armazenado no Amazon S3.

O objetivo foi validar se um arquivo de backup poderia ser:

1. localizado no bucket S3;
2. baixado para o Raspberry Pi;
3. extraído localmente;
4. comparado com os arquivos originais.

Essa etapa é importante porque um backup só pode ser considerado útil se também for possível restaurar os dados armazenados.

---

## 1. Identificação do backup

Primeiro foi verificado quais backups estavam disponíveis no bucket S3.

### Comando utilizado

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

Entre os arquivos disponíveis estava o backup mais recente:

```text
backup-2026-09-04-115202.tar.gz
```

Esse foi o arquivo escolhido para realizar o teste de restauração.

---

## 2. Criação da pasta de restauração

Para evitar alterar os arquivos originais utilizados no teste, foi criada uma pasta separada para a restauração.

### Comando utilizado

```bash
mkdir -p ~/backup-lab/restore/teste
```

A estrutura utilizada ficou:

```text
~/backup-lab/
│
├── teste/
│   ├── backup-01.txt
│   ├── backup-02.txt
│   └── backup-03.txt
│
└── restore/
    └── teste/
```

A pasta `teste` contém os arquivos originais.

A pasta `restore/teste` foi utilizada para armazenar os arquivos restaurados.

---

## 3. Download do backup

Depois de criar a pasta de restauração, o arquivo foi baixado do Amazon S3 para o Raspberry Pi.

### Comando utilizado

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-04-115202.tar.gz ~/backup-lab/restore/teste/
```

O comando `aws s3 cp` realiza uma cópia entre o Amazon S3 e o sistema local.

Neste caso:

```text
Amazon S3
    │
    │ download
    ▼
Raspberry Pi 5
    │
    ▼
~/backup-lab/restore/teste/
```

---

## 4. Verificação do arquivo baixado

Depois do download, foi acessada a pasta de restauração.

### Comando utilizado

```bash
cd ~/backup-lab/restore/teste
```

Em seguida:

```bash
ls
```

O conteúdo observado foi:

```text
backup-01.txt
backup-02.txt
backup-03.txt
backup-2026-09-04-115202.tar.gz
```

Os arquivos `backup-01.txt`, `backup-02.txt` e `backup-03.txt` estavam presentes na pasta de restauração.

O arquivo:

```text
backup-2026-09-04-115202.tar.gz
```

é o arquivo compactado baixado do S3.

---

## 5. Extração do backup

Com o arquivo `.tar.gz` disponível localmente, foi realizada a extração do conteúdo.

### Comando utilizado

```bash
tar -xzf ~/backup-lab/restore/teste/backup-2026-09-04-115202.tar.gz -C ~/backup-lab/restore/teste/
```

### O que significa o comando?

O comando utilizado foi:

```bash
tar -xzf
```

Cada opção possui uma função:

```text
-x
```

Extrai os arquivos.

```text
-z
```

Indica que o arquivo está compactado com gzip.

```text
-f
```

Indica o arquivo que será utilizado.

```text
-C
```

Define o diretório onde os arquivos serão extraídos.

Portanto:

```bash
tar -xzf backup.tar.gz -C pasta/
```

significa:

> Extrair o conteúdo do arquivo `backup.tar.gz` dentro da pasta especificada.

---

## 6. Verificação dos arquivos restaurados

Depois da extração, os arquivos foram listados novamente.

### Comando utilizado

```bash
ls
```

O resultado foi:

```text
backup-01.txt
backup-02.txt
backup-03.txt
backup-2026-09-04-115202.tar.gz
```

Os três arquivos que estavam originalmente na pasta de teste estavam presentes:

```text
backup-01.txt
backup-02.txt
backup-03.txt
```

Isso confirmou que o conteúdo armazenado dentro do arquivo `.tar.gz` foi extraído corretamente.

---

## 7. Comparação com os arquivos originais

Para verificar se os arquivos restaurados eram iguais aos arquivos originais, foi utilizado o comando `diff`.

### Comando utilizado

```bash
diff -r ~/backup-lab/teste ~/backup-lab/restore/teste --exclude='*.tar.gz'
```

O parâmetro:

```text
-r
```

faz a comparação de forma recursiva.

O parâmetro:

```text
--exclude='*.tar.gz'
```

faz com que os arquivos `.tar.gz` sejam ignorados durante a comparação.

### Resultado

O comando não apresentou nenhuma saída.

Isso significa que não foram encontradas diferenças entre os arquivos comparados.

Portanto, os arquivos restaurados foram considerados iguais aos arquivos existentes na pasta original durante esse teste.

---

## 8. Fluxo completo da restauração

O processo realizado foi:

```text
Amazon S3
    │
    │ aws s3 cp
    ▼
backup-2026-09-04-115202.tar.gz
    │
    │ tar -xzf
    ▼
Arquivos restaurados
    │
    ├── backup-01.txt
    ├── backup-02.txt
    └── backup-03.txt
    │
    │ diff
    ▼
Comparação com os arquivos originais
    │
    ▼
Nenhuma diferença encontrada
```

---

## 9. Relação entre backup e restauração

O processo completo desenvolvido até esta etapa pode ser representado da seguinte forma:

```text
                  BACKUP

Arquivos no Raspberry Pi
          │
          ▼
      backup.sh
          │
          ▼
      tar.gz
          │
          ▼
      AWS CLI
          │
          ▼
    Amazon S3
          │
          │
          │
          ▼
       RESTORE

    Amazon S3
          │
          ▼
      AWS CLI
          │
          ▼
      tar.gz
          │
          ▼
   Extração com tar
          │
          ▼
Arquivos restaurados
```

O mesmo arquivo que foi criado pelo processo de backup pode posteriormente ser recuperado do S3 e utilizado para restaurar os dados.

---

## 10. Comandos utilizados

Os principais comandos utilizados nesta etapa foram:

### Listar os backups no S3

```bash
aws s3 ls s3://raspberry-pi-s3-backup-lab-2026/backup/
```

### Criar a pasta de restauração

```bash
mkdir -p ~/backup-lab/restore/teste
```

### Baixar o backup

```bash
aws s3 cp s3://raspberry-pi-s3-backup-lab-2026/backup/backup-2026-09-04-115202.tar.gz ~/backup-lab/restore/teste/
```

### Acessar a pasta de restauração

```bash
cd ~/backup-lab/restore/teste
```

### Listar os arquivos

```bash
ls
```

### Extrair o backup

```bash
tar -xzf ~/backup-lab/restore/teste/backup-2026-09-04-115202.tar.gz -C ~/backup-lab/restore/teste/
```

### Comparar os arquivos

```bash
diff -r ~/backup-lab/teste ~/backup-lab/restore/teste --exclude='*.tar.gz'
```

---

## 11. Conceitos praticados

Nesta etapa foram praticados conceitos importantes de infraestrutura e DevOps:

- Amazon S3;
- AWS CLI;
- download de objetos;
- arquivos `.tar.gz`;
- compactação e descompactação;
- restauração de arquivos;
- organização de diretórios Linux;
- comparação de arquivos;
- validação do processo de backup;
- recuperação de dados.

---

## 12. Resultado da validação

| Teste                                   | Resultado |
| --------------------------------------- | --------- |
| Localizar backup no S3                  | OK        |
| Baixar backup do S3                     | OK        |
| Arquivo `.tar.gz` disponível localmente | OK        |
| Extrair backup                          | OK        |
| Arquivos restaurados                    | OK        |
| Comparar com arquivos originais         | OK        |
| Diferenças encontradas                  | Nenhuma   |

---

## 13. Conclusão

O processo de restauração foi realizado com sucesso.

O backup:

```text
backup-2026-09-04-115202.tar.gz
```

foi baixado do Amazon S3 para o Raspberry Pi 5 e posteriormente utilizado para restaurar os arquivos.

Os arquivos:

```text
backup-01.txt
backup-02.txt
backup-03.txt
```

foram encontrados na pasta de restauração.

A comparação utilizando o comando `diff` não apresentou nenhuma saída, indicando que não foram encontradas diferenças entre os arquivos originais e os arquivos restaurados durante o teste.

Isso confirmou, dentro do cenário de teste utilizado, que o processo desenvolvido permite não apenas enviar os arquivos para o Amazon S3, mas também recuperar os dados posteriormente.

Essa validação é importante porque demonstra o funcionamento do ciclo completo:

```text
Backup
  ↓
Compactação
  ↓
Upload para S3
  ↓
Download
  ↓
Extração
  ↓
Restauração
  ↓
Validação
```

O processo de restauração está validado e a próxima etapa será documentar os problemas encontrados durante o desenvolvimento e as respectivas soluções.

---

[Anterior: Automatização com Cron](06-automatizacao-cron.md) | [README](../README.md) | [Próximo: Troubleshooting](08-troubleshooting.md)
