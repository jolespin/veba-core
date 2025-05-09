# VEBA Core
Core utility functions and objects for VEBA

## Install
```
pip install veba-core
```

## Create SQLite database from VEBA essentials

From the CLI
```bash
output_database="./test/test-no-sequences.db" # sqlite:/// is preprended internally
veba_essentials_directory="./test/veba_output/essentials"
build-relational-database.py -i ${veba_essentials_directory} -o ${output_database}
```

From the API
```python
database_url =  "sqlite:///./test/test-no-sequences.db" 
veba_essentials_directory = "./test/veba_output/essentials"
store_sequences = False

db_controller = VEBAEssentialsDatabase(
    database_url=database_url,
    veba_essentials_directory=veba_essentials_directory,
    store_sequences=store_sequences
)
db_controller.populate_all(reset_first=True)

# --- Starting Full Database Population ---
# Loading and preparing source data...
# Parsing protein annotations structure...
# Parsing protein annotations: 100%|██████████| 139448/139448 [00:54<00:00, 2573.73 proteins/s]
# Skipping protein sequence loading (store_sequences=False).

# --- Stage 1: Samples, Pangenomes, Orthologs, HMM Lookups ---
# Populating samples...
# Adding samples: 100%|██████████| 4/4 [00:00<00:00, 151.80 samples/s]
# Samples populated (Added: 4, Skipped Existing: 0).
# Populating pangenomes...
# Found 32 unique pangenome IDs.
# Adding Pangenomes: 100%|██████████| 32/32 [00:00<00:00, 899.75 pangenomes/s]
# Pangenomes populated (Added: 32, Skipped Existing: 0).
# Flushed Samples and Pangenomes...
# Populating orthologs...
# Adding Orthologs: 100%|██████████| 122912/122912 [01:42<00:00, 1196.05 orthologs/s]
# Orthologs populated (Added: 122912 [Linked: 122912, Unlinked/Skipped: 0], Skipped Existing: 0).
# Populating HMM lookup tables...
# Ensuring HMM objects: 100%|██████████| 5/5 [00:00<00:00, 19.18 HMM DBs/s]
# HMM objects ensured (Added: 14882, Found Existing: 0).
# Attempting to commit Stage 1: Samples, Pangenomes, Orthologs, HMM Lookups...
# Commit successful for Stage 1: Samples, Pangenomes, Orthologs, HMM Lookups.
# --- Stage 1: Samples, Pangenomes, Orthologs, HMM Lookups Complete ---

# --- Stage 2: Genomes ---
# Populating genomes...
# Adding genomes: 100%|██████████| 43/43 [00:00<00:00, 1572.96 genomes/s]
# Genomes populated (Added: 43, Skipped Existing: 0, Link Errors: 0).
# Attempting to commit Stage 2: Genomes...
# Commit successful for Stage 2: Genomes.
# --- Stage 2: Genomes Complete ---

# --- Stage 2.5: Update Genome Paths ---
# Updating genome sequence file paths...

# Updating AA paths: 100%|██████████| 43/43 [00:00<00:00, 2097.13 files/s]
# Updating CDS paths: 100%|██████████| 43/43 [00:00<00:00, 2243.03 files/s]
# Genome paths updated (AA: 43, CDS: 43, Genomes Not Found: 0).
# Attempting to commit Stage 2.5: Update Genome Paths...
# Commit successful for Stage 2.5: Update Genome Paths.
# --- Stage 2.5: Update Genome Paths Complete ---

# --- Stage 3: Contigs ---
# Populating contigs...
# Adding contigs: 100%|██████████| 43/43 [00:56<00:00,  1.31s/ genomes]
# Contigs populated (Added: 29759, Skipped Existing: 0, Genome Not Found Errors: 0).
# Attempting to commit Stage 3: Contigs...
# Commit successful for Stage 3: Contigs.
# --- Stage 3: Contigs Complete ---

# --- Stage 4: Proteins & M2M Links ---
# Populating proteins and linking annotations...
# Cached 7473 objects for Pfam
# Cached 4 objects for NCBIfamAMR
# Cached 5399 objects for KOfam
# Cached 1960 objects for Enzyme
# Cached 46 objects for AntiFam
# Adding proteins: 100%|██████████| 139448/139448 [01:17<00:00, 1804.67 proteins/s]
# Proteins populated (Added: 139448, Skipped Existing: 0, Link Errors: 0).
# Attempting to commit Stage 4: Proteins & M2M Links...
# Commit successful for Stage 4: Proteins & M2M Links.
# --- Stage 4: Proteins & M2M Links Complete ---

# --- Database Population Finished Successfully ---
# --- Database Object Summary ---
# Available tables for summary: ['antifam', 'contig', 'enzyme', 'genome', 'kofam', 'ncbifam_amr', 'ortholog', 'pangenome', 'pfam', 'protein', 'protein_antifam_association', 'protein_enzyme_association', 'protein_kofam_association', 'protein_ncbifam_amr_association', 'protein_pfam_association', 'sample']
# - sample: 4
# - pangenome: 32
# - genome: 43
# - ortholog: 122912
# - contig: 29759
# - protein: 139448
# - pfam: 7473
# - kofam: 5399
# - ncbifam_amr: 4
# - antifam: 46
# - enzyme: 1960
# -----------------------------
# CPU times: user 4min 19s, sys: 11.3 s, total: 4min 30s
# Wall time: 10min 11s

```

## Converting SQLite to PostgreSQL
### Install `pgloader`
```
sudo apt install postgresql postgresql-contrib pgloader
```
### Start PostgreSQL server
```
sudo systemctl start postgresql
# sudo systemctl enable postgresql # To start on boot
sudo systemctl status postgresql
```

### Set up user
```
# Create a user
sudo -u postgres psql -c "CREATE USER veba WITH PASSWORD 'hello-postgresql';"

# Remove a user
# sudo -u postgres psql -c "DROP USER veba;"

# Check user
sudo -u postgres psql -c "\du"

                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 veba      |                                                            | {}

```

### Create PostgreSQL database
```
postgresql_database_name="veba-essentials-database"
sudo -u postgres psql -c "CREATE DATABASE \"${postgresql_database_name}\";"
```

### Convert to PostgreSQL
```
export PGUSER="veba"
export PGPASSWORD="hello-postgresql"
export PGHOST="localhost"
export PGPORT=5432

pgloader test-case-study.db postgresql://${PGUSER}:${PGPASSWORD}@${PGHOST}:${PGPORT}/${postgresql_database_name}
```

### Connect to PostgreSQL database
#### Connect

```
psql -U ${PGUSER} -d ${postgresql_database_name}
```

#### List all tables
```
\dt
                    List of relations
 Schema |              Name               | Type  | Owner 
--------+---------------------------------+-------+-------
 public | antifam                         | table | veba
 public | contig                          | table | veba
 public | enzyme                          | table | veba
 public | genome                          | table | veba
 public | kofam                           | table | veba
 public | ncbifam_amr                     | table | veba
 public | ortholog                        | table | veba
 public | pangenome                       | table | veba
 public | pfam                            | table | veba
 public | protein                         | table | veba
 public | protein_antifam_association     | table | veba
 public | protein_enzyme_association      | table | veba
 public | protein_kofam_association       | table | veba
 public | protein_ncbifam_amr_association | table | veba
 public | protein_pfam_association        | table | veba
 public | sample                          | table | veba
(16 rows)
```


#### Describe a table
```
\d protein
                Table "public.protein"
   Column    |  Type  | Collation | Nullable | Default 
-------------+--------+-----------+----------+---------
 name        | text   |           | not null | 
 cds         | text   |           |          | 
 aa          | text   |           |          | 
 md5_cds     | text   |           |          | 
 md5_aa      | text   |           |          | 
 length      | bigint |           |          | 
 product     | text   |           |          | 
 uniref      | text   |           |          | 
 mibig       | text   |           |          | 
 vfdb        | text   |           |          | 
 cazy        | text   |           |          | 
 contig_id   | text   |           |          | 
 ortholog_id | text   |           |          | 
Indexes:
    "idx_16437_sqlite_autoindex_protein_1" PRIMARY KEY, btree (name)
    "idx_16437_ix_protein_cazy" btree (cazy)
    "idx_16437_ix_protein_md5_aa" btree (md5_aa)
    "idx_16437_ix_protein_md5_cds" btree (md5_cds)
    "idx_16437_ix_protein_mibig" btree (mibig)
    "idx_16437_ix_protein_uniref" btree (uniref)
    "idx_16437_ix_protein_vfdb" btree (vfdb)
Foreign-key constraints:
    "protein_contig_id_fkey" FOREIGN KEY (contig_id) REFERENCES contig(name)
    "protein_ortholog_id_fkey" FOREIGN KEY (ortholog_id) REFERENCES ortholog(name)
Referenced by:
    TABLE "protein_antifam_association" CONSTRAINT "protein_antifam_association_protein_id_fkey" FOREIGN KEY (protein_id) REFERENCES protein(name)
    TABLE "protein_enzyme_association" CONSTRAINT "protein_enzyme_association_protein_id_fkey" FOREIGN KEY (protein_id) REFERENCES protein(name)
    TABLE "protein_kofam_association" CONSTRAINT "protein_kofam_association_protein_id_fkey" FOREIGN KEY (protein_id) REFERENCES protein(name)
    TABLE "protein_ncbifam_amr_association" CONSTRAINT "protein_ncbifam_amr_association_protein_id_fkey" FOREIGN KEY (protein_id) REFERENCES protein(name)
    TABLE "protein_pfam_association" CONSTRAINT "protein_pfam_association_protein_id_fkey" FOREIGN KEY (protein_id) REFERENCES protein(name)
```

## Querying SQL from cli

### Output all the genomes that contain a protein with the `PF11832.13` annotation:
```
$ sqlite3 test.db "SELECT DISTINCT g.name FROM genome g JOIN contig c ON g.name = c.genome_id JOIN protein p ON c.name = p.contig_id JOIN protein_pfam_association ppa ON p.name = ppa.protein_id JOIN pfam pf ON ppa.pfam_id = pf.name WHERE pf.name = 'PF11832.13';"
S1__BINETTE__P.1__bin_210
S2__BINETTE__P.1__bin_126
S3__BINETTE__P.1__bin_4
```
### Output the fasta for all the proteins that contain Pfam `PF11832.13`:

```
$ sqlite3 test.db "SELECT '>' || p.name || CHAR(10) || p.aa FROM protein p JOIN protein_pfam_association ppa ON p.name = ppa.protein_id JOIN pfam pf ON ppa.pfam_id = pf.name WHERE pf.name = 'PF11832.13';"
>S1__NODE_1718_length_2749_cov_4.329621_1
MKSNLIVFLSFFIFIGCTNSNKNTIKLIDYVPQNSVYIIKTNNLESLKSNIKNNLLISELKNYSTSKNFISKIKNLEFLNTSNEILFSISLDSKDSMQITAITKLKEDIFNTDSLPDLKIETIKSNKRTLTKTSINNEPLYSMVMDSLFLISSSKETLENAKPNYNTDLHKIYSTIDDSKLLSVLINSRVKNSFPKLFNILDLNFTNYSLLDIDIAQNEIIFNGITQAIDSTSSFINCFKNNVPQENLISKMCPTDIESFNSFTFKNFNEFYKQKSNYLKGSLELERTEFNSVIEFGNLSKADQNASVIRCIDPNTVNDAFVAESISESYRSVEIFSINNFDDIKNSFSPFFNNSSASYYFNIDDFFVLSSDIEFLKTIISNYQNNTSLYDYEPFKNIMKKLSDESSIFIYKNDFGLNQLFNNNFQENLDLKISNYKASAMQFVYDSDFAHINGITKIFKTKVSSNSVSEELNIKLDNDLISSPQIIINHTNNEKDIVVQDLKNNLYLI
>S2__NODE_2293_length_2384_cov_3.185487_1
MKSNLIVFLSFFIFIGCTNSNKNTIKLIDYVPQNSVYIIKTNNLESLKSNIKNNLLISELKNYSTSKNFISKIKNLEFLNTSNEILFSISLDSKDSMQITAITKLKEDIFNTDSLPDLKIETIKSNKRTLTKTSINNEPLYSMVMDSLFLISSSKETLENAKPNYNTDLHKIYSTIDDSKLLSVLINSRVKNSFPKLFNILDLNFTNYSLLDIDIAQNEIIFNGITQAIDSTSSFINCFKNNVPQENLISKMCPTDIESFNSFTFKNFNEFYKQKSNYLKGSLELERTEFNSVIEFGNLSKADQNASVIRCIDPNTVNDAFVAESISESYRSVEIFSFNNFDDIKNSFSPFFNNSSASYYFNIDDFFVLSSDIEFLKTIISNYQNNTSLYDYEPFKNIMKKLSDESSIFIYKNDFGLNQLFNNNFQENLDLKISNYKASAMQFVYDSDFAHINGITKIFKTKVSSNSVSEELNIKLDNDLISSPQIIINHTNNEKDIVVQDLKNNLYLISNKGKVFWKKQLDGKILGDIKQIDMFKNGRLQMVFNTSKHLYILDRNGK
>S3__NODE_103_length_48952_cov_10.001432_27
MSKKSASKKTSTRKKIWRGFKYLLLAGVLGAAAFVVYVLLQDTPDRDIYDFVPEKSVFVVEADDPIENWKSLSKTPMWKHLKKNELFADIEGDANFLDTLINDNERLFDLMAGKKVLICAQMTKADDYDFTYLVDLEKGSKVTFFIDIFKPIMSGVGYPMKESELGGKKTYTINDGYDDIYMAFLDNVLAISYSQKLLLGVIEQEKKPFYSKNANFQLVRDASYDAANRSSIGKVHLNFDQLDEYMSVFMDEVSGSILDISKSMRYASFDLKVEDEYTEMEGLVSVDSANPTLATVLLQQDRGEISAPRILPTNTSFLLTIDFDDFNDFYGSIGESMKEDADYKDFEKTKNTIGKLLGVNANDRKKDRAERKGKDKDYFDWLGQEIALAMVPKNESGSEQSYLAIFHTPDYANAVHDLSQISKKIRRRTPVKFKDYEYRGKDIQWLAMKGFFKLFLGKLFKQFDKPKFVVLEEHVVFSNDTSAIHRVIDASMAEETLYGETGYRRLTREFSDESNYFIYMNSERLYPHLPSLLDAESAGDLRKNREYVVCFPQVGLQLTSDDDDAFETKIYLEYEDPK*
```

## Querying SQL with natural language
Experimental usage for querying SQL database with LLMs

### Install dependencies
```
pip install dotenv pandasai openai
```

### Query SQL
```
import pandas as pd
from sqlalchemy import create_engine,text
from pandasai import SmartDataframe, SmartDatalake
from pandasai.llm import OpenAI # Or another supported LLM like Starcoder, OpenChat
from dotenv import dotenv_values

# Config
config = dotenv_values("~/.openai")
SQLITE_DB_PATH = "test/test-no-sequences.db"
OPENAI_API_KEY = config["api_key"] 

# Connect to SQLite and Load Data into Pandas DataFrames
engine = create_engine(f"sqlite:///{SQLITE_DB_PATH}")

tables = dict()
with engine.connect() as conn:
    result = conn.execute(text("SELECT name FROM sqlite_master WHERE type='table';"))
    available_tables = [row[0] for row in result]
    print(f"Successfully connected and queried.\nTables found: {available_tables}")

    # Get tables
    # ['pfam', 'kofam', 'ncbifam_amr', 'antifam', 'enzyme', 'sample', 'pangenome', 'ortholog', 'genome', 'contig', 'protein', 
    # 'protein_pfam_association', 'protein_kofam_association', 'protein_ncbifam_amr_association', 'protein_antifam_association', 'protein_enzyme_association']
    for name in available_tables:
        tables[name] = pd.read_sql_table(name, conn)


# Initialize PandasAI with an LLM
llm = OpenAI(api_token=OPENAI_API_KEY)

# Create SmartDataframe(s) ---
# You can pass multiple DataFrames to a single SmartDataframe if they are related,
# or create separate ones. PandasAI will try to infer relationships if column names overlap.

## Option A: Separate SmartDataframes (simpler to start)
# smart_df = SmartDataframe(tables["genome"], config={"llm": llm})

# Option B: Combining related DataFrames into a SmartDatalake concept (more powerful for joins)
smart_datalake = SmartDatalake(
    list(tables.values()),
    config={
        "llm": llm,
        "verbose": False, # See the generated Python code
        "save_charts": False,
        "enable_cache": True
    }
)

# Ask Questions (Natural Language Queries)
prompt = "Which proteins have photosystem functionality in this dataset?  Please provide a dataframe with the protein identifiers (excluding ortholog identifiers) and the associated annotation."
response = smart_datalake.chat(prompt)
response.head(10).set_index("protein_identifier")["annotation"]

protein_identifier
S1__NODE_3414_length_1504_cov_3.657005_1                 Putative photosystem II stability/assembly factor
S1__NODE_222_length_28518_cov_10.839792_8                Photosystem II Psb31 protein;Photosystem II Ps...
S1__NODE_111_length_38118_cov_10.717285_7                Manganese-stabilising protein / photosystem II...
S1__NODE_111_length_38118_cov_10.717285_9                              Extrinsic protein in photosystem II
S1__NODE_16_length_73671_cov_10.745599_11                         Photosystem II reaction center X protein
S1__NODE_78_length_45125_cov_10.381673_9                      Photosystem II reaction center Psb28 protein
S1__NODE_814_length_8251_cov_10.641654_1596:2075(-)      Photosystem II;Photosystem II Pbs27;photosyste...
S1__NODE_1172_length_4653_cov_10.408873_102:767(-)            Photosystem II reaction center Psb28 protein
S1__NODE_519_length_15093_cov_9.807687_13794:14298(+)    Photosystem II 12 kDa extrinsic protein;Photos...
S1__NODE_1059_length_5396_cov_11.872683_4313:4720(+)     Photosystem II 12 kDa extrinsic protein (PsbU)...
Name: annotation, dtype: object
```

## Querying PostgreSQL Data as a Graph with PuppyGraph
TBD