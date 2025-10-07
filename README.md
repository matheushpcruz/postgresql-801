Assuntos da Reunião – 06/10 | PostgreSQL: Bate-Papo com o Mestre

Este repositório contém os arquivos utilizados durante a aula **Bate-Papo com o Mestre 4Linux** para o curso **PostgreSQL-801**.


**Certificação EDB – Versão do Curso 4Linux com a Certificação**

O conteúdo do curso da 4Linux é compatível com as provas de certificação da EDB, pois todo o material teórico e prático abrange o PostgreSQL open source e suas ferramentas.

Entretanto, no portal de treinamentos da EDB ([https://training.enterprisedb.com/pages/27/course-catalog](https://training.enterprisedb.com/pages/27/course-catalog)), existem dois tipos de provas de certificação: **Essential** e **Advanced**, tanto para o **EDB PostgreSQL** quanto para o **PostgreSQL Open Source**.

Todo o conteúdo que menciona **EDB PostgreSQL** está relacionado à versão **Enterprise da EDB** e às suas ferramentas específicas. Já o conteúdo voltado ao **PostgreSQL Open Source** refere-se à versão comunitária do banco de dados, que é a base do curso oferecido pela 4Linux.

Essa distinção é importante para garantir a escolha correta da prova de certificação correspondente ao curso realizado.

**Tipos de Backup – Dump e Restore**

No **PostgreSQL**, para realizar **backups lógicos**, utilizamos a ferramenta **pg_dump**. Com ela, é possível gerar dois tipos principais de backup:

1. **Backup em formato SQL** (`.sql`):
   Cria um arquivo único contendo todos os comandos SQL necessários para recriar a estrutura do banco e seus dados.

2. **Backup em formato de diretório** (`-Fd`):
   Divide o backup em vários arquivos, permitindo **execução paralela** e **maior desempenho** na restauração.

> Lembrando que esses comandos podem ser consultados nas páginas 352 a 363 da apostila.

---

**Exemplo de comando padrão do `pg_dump`:**

```bash
pg_dump -h localhost -U postgres -f /var/lib/postgresql/bkp -Fd -j1 -v teste
```

**Descrição dos parâmetros:**

* `-h localhost` → Define o host do servidor PostgreSQL.
* `-U postgres` → Define o usuário que executará o backup.
* `-f /var/lib/postgresql/bkp` → Caminho e nome do diretório onde o backup será salvo.
* `-Fd` → Define o formato **diretório** para o backup.
* `-j1` → Número de **processos paralelos** utilizados (neste caso, 1).
* `-v` → Modo **verboso**, exibindo detalhes do processo.
* `teste` → Nome do **banco de dados** a ser exportado.

---

**Arquivos gerados no formato de diretório:**

```bash
postgres@db1:~$ ls bkp/
3360.dat.gz  3361.dat.gz  3362.dat.gz  toc.dat
postgres@db1:~$
```

**Descrição dos arquivos:**

* `*.dat.gz` → Arquivos compactados contendo os **dados das tabelas** e **objetos** do banco.
* `toc.dat` → Arquivo de **tabela de conteúdo** (Table of Contents), que mapeia a estrutura do backup e orienta o `pg_restore` durante a restauração.

---

**Observação:**
O formato de diretório é recomendado para bancos de dados grandes ou ambientes onde se deseja restaurar apenas objetos específicos, pois permite **restauração seletiva** e **execução paralela** com o comando `pg_restore`.


**Backup em arquivo .sql**

O backup no formato **.sql** é feito da seguinte maneira:

```bash
pg_dump -f /var/lib/postgresql/bkp_series.sql -j1 -v teste
```

**Descrição dos parâmetros:**

* `-f /var/lib/postgresql/bkp_series.sql` → Define o caminho e o nome do arquivo onde o backup será salvo.
* `-j1` → Número de **processos paralelos** (nesse caso, 1; contudo, esse parâmetro só é válido para o formato de diretório).
* `-v` → Modo **verboso**, exibe detalhes do processo.
* `teste` → Nome do **banco de dados** a ser exportado.

**Observação:**
No formato **.sql**, o `pg_dump` gera um **script completo** com os comandos SQL necessários para recriar todo o banco — estruturas, dados, funções, permissões etc.



**Restauração de Backups no PostgreSQL**

Agora, vamos ver quais comandos utilizamos para restaurar cada tipo de backup.

---

**Backup em formato .sql**

O backup em formato **.sql** pode ser restaurado da seguinte maneira:

```bash
psql -h localhost -U postgres -d teste < bkp_series.sql
```

**Descrição dos parâmetros:**

* `psql` → Cliente de linha de comando do PostgreSQL, usado para executar scripts SQL.
* `-h localhost` → Define o host (endereço do servidor PostgreSQL).
* `-U postgres` → Usuário utilizado para se conectar ao banco.
* `-d teste` → Nome do banco de dados onde o script será restaurado.
* `< bkp_series.sql` → Redireciona o conteúdo do arquivo `.sql` para o comando `psql`, executando os comandos SQL contidos nele.

> Esse tipo de backup é restaurado executando diretamente os comandos SQL gerados pelo `pg_dump`.

---

**Backup em formato de diretório (-Fd)**

Para o backup gerado no formato de diretório, utilizamos o **pg_restore**, conforme abaixo:

```bash
pg_restore -h localhost -U postgres -d teste -j1 -Fd /var/lib/postgresql/bkp/
```

**Descrição dos parâmetros:**

* `pg_restore` → Utilitário responsável por restaurar backups criados com `pg_dump` nos formatos **diretório**, **custom** ou **tar**.
* `-h localhost` → Define o host (endereço do servidor PostgreSQL).
* `-U postgres` → Usuário utilizado para se conectar ao banco.
* `-d teste` → Nome do banco de dados de destino.
* `-j1` → Número de **processos paralelos** utilizados na restauração (1 neste exemplo).
* `-Fd` → Indica que o formato do backup é de **diretório**.
* `/var/lib/postgresql/bkp/` → Caminho onde estão os arquivos do backup.

> O formato de diretório é mais flexível, permitindo restaurar objetos específicos (como tabelas ou esquemas) e utilizar múltiplos processos para acelerar a restauração.


**Restauração Parcial de Backup (Tabelas Específicas)**

Também é possível restaurar **parcialmente** um backup completo, recuperando apenas uma ou algumas tabelas específicas.

Para isso, podemos gerar uma **lista dos objetos** contidos no backup e escolher quais deles serão restaurados.

---

**Gerando a lista de objetos do backup**

Utilize o comando abaixo com base em um backup completo no formato de diretório:

```bash
pg_restore -l /var/lib/postgresql/2025-10-06/ > db.list
```

Esse comando lê o conteúdo do backup e gera um arquivo chamado **`db.list`**, listando todos os objetos (tabelas, índices, dados, funções, etc.) armazenados no backup.

---

**Exemplo do conteúdo de `db.list`:**

```bash
postgres@db1:~$ cat db.list
;
; Archive created at 2025-10-06 23:44:02 UTC
;     dbname: teste
;     TOC Entries: 10
;     Compression: gzip
;     Dump Version: 1.16-0
;     Format: DIRECTORY
;     Dumped from database version: 17.6 (Debian 17.6-2.pgdg12+1)
;     Dumped by pg_dump version: 17.6 (Debian 17.6-2.pgdg12+1)
;
; Selected TOC Entries:
;
217; 1259 16395 TABLE public series postgres
218; 1259 16398 TABLE public series2 postgres
219; 1259 16401 TABLE public series3 postgres
3360; 0 16395 TABLE DATA public series postgres
3361; 0 16398 TABLE DATA public series2 postgres
3362; 0 16401 TABLE DATA public series3 postgres
postgres@db1:~$
```

---

**Editando o arquivo `db.list`**

Podemos editar o arquivo e comentar (`;`) as linhas que **não desejamos restaurar**.

* Se colocarmos `;` na frente de uma linha com **`TABLE DATA`**, o **conteúdo (dados)** dessa tabela **não será restaurado**, apenas a estrutura.
* Se colocarmos `;` na frente de uma linha com **`TABLE public`**, a **estrutura da tabela** **não será criada**, e consequentemente seus dados também não serão importados.

---

**Exemplo prático**

Se quisermos restaurar **apenas a tabela `series`**, deixamos o arquivo assim:

```bash
217; 1259 16395 TABLE public series postgres
3360; 0 16395 TABLE DATA public series postgres
;218; 1259 16398 TABLE public series2 postgres
;219; 1259 16401 TABLE public series3 postgres
;3361; 0 16398 TABLE DATA public series2 postgres
;3362; 0 16401 TABLE DATA public series3 postgres
```

---

**Restaurando com base na lista editada**

Após editar o arquivo, executamos:

```bash
pg_restore -L db.list -d teste -U postgres -v /var/lib/postgresql/2025-10-06/
```

**Descrição dos parâmetros:**

* `-L db.list` → Especifica o arquivo com a lista de objetos que serão restaurados.
* `-d teste` → Banco de dados de destino.
* `-U postgres` → Usuário utilizado para conexão.
* `-v` → Modo verboso (exibe detalhes da restauração).
* `/var/lib/postgresql/2025-10-06/` → Caminho do backup em formato de diretório.

---

 **Resumo:**

* `pg_restore -l` → Gera lista de objetos do backup.
* Editar `db.list` → Define o que será restaurado.
* `;` antes de `TABLE` → ignora a **estrutura** da tabela.
* `;` antes de `TABLE DATA` → ignora os **dados** da tabela.
* `pg_restore -L` → Restaura somente os objetos selecionados.


**Backup Físico com Barman**

O **backup físico** realiza a cópia de **todos os arquivos do PostgreSQL**, incluindo **WALs** (Write-Ahead Logs) e **configurações do banco**. Para isso, utilizamos a ferramenta **Barman**: [https://pgbarman.org/](https://pgbarman.org/).

Na aula, foram usadas **duas VMs básicas**:

* **DB1 (172.27.11.10)** → Servidor primário do PostgreSQL
* **DB2 (172.27.11.20)** → Servidor de backup com Barman, Debian instalado

---

**Instalação do Barman**

No **DB2**, execute:

```bash
apt-get install barman
```

---

**Criação de chaves SSH**

Para que o Barman possa se conectar via SSH sem senha, criamos chaves para os usuários **postgres** e **barman** em ambas as VMs.

**No DB1 e DB2:**

```bash
# Usuário postgres
su - postgres
ssh-keygen -t rsa -b 4096 -N ""

# Usuário barman
su - barman
ssh-keygen -t rsa -b 4096 -N ""
```

Adicione a chave pública do usuário remoto no arquivo `authorized_keys` correspondente:

```bash
# Chave do postgres adicionada ao barman
cat /var/lib/postgresql/.ssh/id_rsa.pub (DB1) 

nano /var/lib/barman/.ssh/authorized_keys (DB2)

# Chave do barman adicionada ao postgres
cat /var/lib/barman/.ssh/id_rsa.pub  (DB2)

nano /var/lib/postgresql/.ssh/authorized_keys (DB1)
```

---

**Criação de usuário no PostgreSQL**

No **DB1**, criamos um usuário com permissões para o Barman:

```sql
CREATE USER barman WITH ENCRYPTED PASSWORD 'barman123' SUPERUSER;
```

> Esse usuário precisa de privilégios de **superuser** para permitir que o Barman faça backups completos e acesse os arquivos do sistema.

---

**Configuração do Barman no DB2**

Arquivo: `/etc/barman.d/backup_postgresql.conf`

```ini
[backup_postgres]

description = 'Backup Curso PostgreSQL'
compression = gzip
archiver = on
ssh_command = ssh postgres@172.27.11.10
conninfo = host=172.27.11.10 user=barman dbname=postgres password=barman123 port=5432
reuse_backup = link
backup_method = rsync
backup_options = concurrent_backup
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 DAYS
path_prefix = "/usr/bin"
```

**Explicação dos parâmetros:**

* [backup_postgres] → Nome dado ao backup, usado para referenciar este servidor de backup no Barman.
* `description` → Descrição do backup.
* `compression` → Tipo de compressão dos backups (`gzip`).
* `archiver` → Ativa o arquivamento dos WALs.
* `ssh_command` → Comando SSH usado para acessar o servidor primário.
* `conninfo` → String de conexão com o PostgreSQL.
* `reuse_backup = link` → Permite reutilizar arquivos de backup anteriores usando links simbólicos, economizando espaço.
* `backup_method = rsync` → Método de backup (copia arquivos com `rsync`).
* `backup_options = concurrent_backup` → Permite backup enquanto o servidor está ativo.
* `retention_policy_mode = auto` → Política automática de retenção de backups.
* `retention_policy = RECOVERY WINDOW OF 7 DAYS` → Mantém backups para recuperação nos últimos 7 dias.
* `path_prefix` → Prefixo para localizar binários do PostgreSQL.

---

**Configuração do PostgreSQL para archive**

No **DB1**, edite `/etc/postgresql/17/main/postgresql.conf`:

```ini
archive_mode = on
archive_command = 'rsync -a %p barman@172.27.11.20:/var/lib/barman/backup_postgres/incoming/%f'
```

* `archive_mode = on` → Ativa o arquivamento dos WALs.
* `archive_command` → Comando que envia cada WAL gerado para o servidor Barman usando `rsync`.
* `%p` → Caminho do arquivo WAL atual.
* `%f` → Nome do arquivo WAL que será copiado.

Após a alteração, recarregue as configurações:

```sql
SELECT pg_reload_conf();
```

---

**Verificação da configuração no DB2**

Entre como usuário **barman**:

```bash
su - barman
barman check backup_postgres
```

> Esse comando verifica se todas as configurações estão corretas e se a conexão com o servidor primário está funcional.

---

**Executando o backup**

No **DB2**, execute:

```bash
barman backup backup_postgres
```

> Esse comando inicia o backup físico completo do PostgreSQL primário para o servidor Barman.

---

**Observações**

* A parte de **archive** e configuração dos WALs está detalhada na **apostila, página 344**.


**Restauração de Backup com Barman**

Para **restaurar um backup** gerado pelo Barman, primeiro consulte os backups disponíveis:

```bash
barman list-backups backup_postgres
```

**Exemplo de saída:**

```bash
barman@db2:~$ barman list-backups backup_postgres
backup_postgres 20251007T004258 - R - Tue Oct  7 00:43:02 2025 - Size: 133.7 MiB - WAL Size: 1013.3 KiB
backup_postgres 20251007T004041 - R - Tue Oct  7 00:40:45 2025 - Size: 133.7 MiB - WAL Size: 32.3 KiB
backup_postgres 20251007T003333 - R - Tue Oct  7 00:33:39 2025 - Size: 133.7 MiB - WAL Size: 32.4 KiB
barman@db2:~$
```

* Cada linha representa um backup realizado.
* O **timestamp** indica o momento exato em que o backup foi concluído.
* `R` indica que o backup está **ready** (pronto para uso).
* `Size` mostra o tamanho do backup e `WAL Size` indica o tamanho dos logs arquivados.

---

**Restaurando um backup específico**

Escolha o backup que deseja restaurar e execute:

```bash
barman recover backup_postgres 20251007T003333 /tmp/restore
```

**Explicação:**

* `barman recover` → Comando para restaurar o backup.
* `backup_postgres` → Nome do servidor de backup configurado no Barman.
* `20251007T003333` → Identificador do backup que será restaurado (timestamp).
* `/tmp/restore` → Diretório onde os arquivos do backup serão recuperados.

Após a execução, o diretório `/tmp/restore` conterá todos os arquivos do banco restaurados.

> Para colocar o backup em operação, basta copiar esses dados para um **novo PGDATA** e iniciar o PostgreSQL apontando para esse diretório.

---

**Replicação**

A explicação e os comandos detalhados sobre **replicação** podem ser consultados na **página 371** da apostila.
