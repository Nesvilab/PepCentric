# PepCentric backend server

## Requirements
1. Linux
2. Java 11+
3. GCC: in order to get the flags for `open()` in `jaydio`
4. Docker and the Docker Compose plugin, for the recommended frontend deployment


## Disk related command reference
- `sudo fdisk -l`
- `sudo file -s /dev/nvme1n1`
- `sudo mkfs -t xfs /dev/nvme1n1`
- `sudo mkdir /slowstorage`
- `sudo mount /dev/nvme1n1 /slowstorage`


## Tomcat

### Using Docker Compose (recommended)
Installing and configuring Tomcat manually is not needed if `PepCentricFrontend` is deployed with Docker Compose. Docker Compose is the recommended deployment method.

#### Build the WAR file
The `Dockerfile` copies the WAR in under its exact name, so build it in the `PepCentric-frontend`
working copy and put it beside the `Dockerfile`:

```shell
./gradlew war   # writes build/libs/PepCentricFrontend-<version>.war
```

On Apple silicon, change `JAVA_HOME` in the `Dockerfile` from `java-11-openjdk-amd64` to
`java-11-openjdk-arm64`.

#### Configure the frontend
`PepCentricFrontend` reads `SERVER_URL`, `DB_URL`, `DB_USERNAME` and `DB_PASSWORD` from runtime
environment variables. The MySQL database is not created by the compose file; set it up separately as
described under [SQL database](#sql-database).

Recommended Docker Compose configuration:

```yaml
services:
  pepcentricfrontend-app:
    image: pepcentricfrontend:latest
    container_name: pepcentricfrontend-app
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "80:8080"
    environment:
      SERVER_URL: ${SERVER_URL:?Set SERVER_URL to the externally managed backend server URL}
      DB_URL: ${DB_URL:?Set DB_URL to the externally managed MySQL JDBC URL}
      DB_USERNAME: ${DB_USERNAME:?Set DB_USERNAME for the externally managed MySQL database}
      DB_PASSWORD: ${DB_PASSWORD:?Set DB_PASSWORD for the externally managed MySQL database}
```

Supply the values in a `.env` file beside `docker-compose.yml`, excluded from version control:

```shell
SERVER_URL=http://<backend-host>:50001
DB_URL=jdbc:mysql://<mysql-host>/fragmentsearcher?serverTimezone=US/Eastern
DB_USERNAME=pepcentric
DB_PASSWORD=<password>
```

Start the frontend server from the directory containing `docker-compose.yml`:

```shell
docker compose up -d
```

Stop and remove the frontend container:

```shell
docker compose down
```

Stop and remove the frontend container and image:

```shell
docker compose down --rmi all
```

### Set up on host
Use this section only when deploying the WAR directly to a system Tomcat installation.

#### Install and configure Tomcat 9 on Ubuntu
```shell
sudo apt update
sudo apt install tomcat9 tomcat9-admin
ss -ltn
sudo systemctl enable tomcat9
sudo ufw allow from any to any port 8080 proto tcp

# edit to create users
sudo vim /etc/tomcat9/tomcat-users.xml

# change max upload file size limit
sudo vim /usr/share/tomcat9-admin/manager/WEB-INF/web.xml

# add JAVA_HOME to the setenv.sh file
sudo vim /usr/share/tomcat9/bin/setenv.sh

sudo systemctl restart tomcat9
```

__Note:__ Must put `mysql-connector-java-*.jar` in `/var/lib/tomcat9/lib/` and `/usr/share/tomcat9/lib/` because we do not pack it in the war to prevent an issue in Tomcat.

#### Configure `PepCentricFrontend` WAR
`PepCentricFrontend` reads deployment-specific URLs and passwords from runtime environment variables. For a manual system Tomcat install on Ubuntu, put the exports in `/usr/share/tomcat9/bin/setenv.sh`:

```shell
#!/bin/sh

export SERVER_URL="http://<backend-host>:50001"
export DB_URL="jdbc:mysql://<mysql-host>/fragmentsearcher?serverTimezone=US/Eastern"
export DB_USERNAME="pepcentric"
export DB_PASSWORD="<password>"
```

Then restart Tomcat:

```shell
sudo systemctl restart tomcat9
```

#### Useful tips
Stop and start tomcat:
```shell
service tomcat stop
service tomcat start
service tomcat restart
```

Paths:
```shell
/etc/tomcat9/
/usr/share/tomcat9/
/var/lib/tomcat9/
/var/log/tomcat9/
```


## SQL database
- table `jobque`
- `jobque` columns: `id`, `version`, `time`, `email`, `jobname`, `ip`, `query`, `reqm`, `result`, `selection`, `status`, `uuid`, `varm`, `enzyme`, `peplenmin` and `indexes`

### Install and configure MySQL on Ubuntu
```shell
sudo apt update 
sudo apt install mysql-server 
 
# Configuring MySQL 
sudo mysql_secure_installation 

# Creating a Dedicated MySQL User and Granting Privileges 
sudo mysql 
mysql> CREATE DATABASE fragmentsearcher; 
mysql> CREATE USER 'pepcentric'@'%' IDENTIFIED BY '<password>'; 
mysql> GRANT ALL PRIVILEGES ON *.* to 'pepcentric'@'%' WITH GRANT OPTION; 
mysql> FLUSH PRIVILEGES;
mysql> exit 
```

### Allow remote access to MySQL:
[https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql](https://www.digitalocean.com/community/tutorials/how-to-allow-remote-access-to-mysql)

### Create SQL table
```mysql
mysql> USE fragmentsearcher; 
mysql> CREATE TABLE jobque (id bigint NOT NULL AUTO_INCREMENT, version bigint, time varchar(255), email varchar(255), jobname varchar(255), ip varchar(255), query longtext, reqm varchar(255), result varchar(255), selection varchar(255), status varchar(255), uuid varchar(255), varm varchar(255), enzyme varchar(255), peplenmin int, indexes varchar(255), PRIMARY KEY (id));
mysql> CREATE INDEX id ON jobque (id);
```

The API server drops and recreates two further tables, `indexes` and `datasets`, every time it
starts, so the database user needs `DROP` and `CREATE` privileges as well.

### Useful commands
```shell
mysqldump -u pepcentric -p --no-tablespaces --databases fragmentsearcher > dump.sql # backup a database
mysql -u pepcentric -p fragmentsearcher < dump.sql # restore a database
sudo service mysql start
# If there is a "Failed to start mysql.service: Unit mysql.service is masked" error, run sudo systemctl unmask mysql.service
```


## Directory structure
### `~/pepcentric` directory example
```shell
.
├── mysql-connector-java-8.0.33.jar
├── nonspecific_decoy_hist.bin
├── pepcentric_scratch
├── run_pepcentric.sh
└── enzymatic_decoy_hist.bin
```

- `pepcentric_scratch` directory is empty at the beginning. It contains users' requests and logs.
- `*.bin` are decoy histograms. Please refer to [Steps to generate `decoy_hist.bin`](#steps-to-generate-decoy_histbin) for how to generate it.

*Note: One file is generated per digestion type, covering every index that digestion type can reach, and it must be named `<digestion type>_decoy_hist.bin`. When the content of any covered index is changed, the file should be regenerated.*

#### `run_pepcentric.sh` example
```shell
#!/bin/bash

set -xe

pepcentric_jar=/home/ubuntu/peptidecentricsearch.jar
mysql_jar=mysql-connector-java-8.0.33.jar
index_root_dir=/slowstorage/pepcentric/index/
dataset_dir=/slowstorage/pepcentric/dataset/
search_dir=/home/ubuntu/pepcentric/pepcentric_scratch/
params_dir=/slowstorage/pepcentric/params/
decoy_peptides_bin_1=/home/ubuntu/pepcentric/enzymatic_decoy_hist.bin
decoy_peptides_bin_2=/home/ubuntu/pepcentric/nonspecific_decoy_hist.bin
port=50001

nohup java -Xmx10g -cp $pepcentric_jar:$mysql_jar pepcentric.api.APIServer --port $port --datasets-root $dataset_dir --indexes-root $index_root_dir --working-scratch $search_dir --decoy-peptides-bin $decoy_peptides_bin_1 --decoy-peptides-bin $decoy_peptides_bin_2 > apiserver.log 2>&1 &
nohup java -Xmx10g -cp $pepcentric_jar pepcentric.assembler.AssemblyServer $search_dir $index_root_dir $params_dir > assemblyserver.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar:$mysql_jar pepcentric.digestion.DigestionServer $params_dir $search_dir > digestionserver.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/tmt/ /slowstorage/pepcentric/index/tmt/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/tmt.pepcentricparams > searchserver_tmt.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/lf/ /slowstorage/pepcentric/index/lf/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/lf.pepcentricparams > searchserver_lf.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/lf2/ /slowstorage/pepcentric/index/lf2/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/lf2.pepcentricparams > searchserver_lf2.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/hla/ /slowstorage/pepcentric/index/hla/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/hla.pepcentricparams > searchserver_hla.log 2>&1 &
```

Each search server takes the parameter file of the index it serves.

### `~/.pepcentric.config` example
```shell
deisotoped = 1

pepcentric.web.db.host =  
pepcentric.web.db.port =  
pepcentric.web.db.user =  
pepcentric.web.db.password =  
pepcentric.web.db.dbname =  

pepcentric.repo.db.host =  
pepcentric.repo.db.port =  
pepcentric.repo.db.user =  
pepcentric.repo.db.password =  
pepcentric.repo.db.dbname =  

pepcentric.repo.pathtranslator.poolfilename =  

pepcentric.fasta = /slowstorage/pepcentric/proteome.fas  
pepcentric.fasta.decoyprefix = rev_  

pepcentric.params = /slowstorage/pepcentric/params/  

pepcentric.commonptm = 79.96630(aa=ST);42.01060(aa=[^TSKYHR);15.99490(aa=TESQLIWHVRMYFPNKD);0.98400(aa=NQC)  
```

The last three keys drive [`peptide_type`](api.md#the-peptide_type-field) and
[`caution`](api.md#the-caution-field):

- `pepcentric.fasta` is the reference proteome peptides are compared against for a SAAV. Headers
  starting with `pepcentric.fasta.decoyprefix` are skipped, so a combined target-decoy file works.
- `pepcentric.params` is the **directory** holding the `*.pepcentricparams` files, not one of them.
  Give it the same value `run_pepcentric.sh` uses for `params_dir`.
- `pepcentric.commonptm` lists the modifications `caution` treats as common, separated by `;`, each a
  mass optionally followed by `(aa=<residues>)`.

An empty `pepcentric.fasta` leaves every `peptide_type` 0 or 1, and an empty `pepcentric.params`
leaves it 0 or 2. Both are read at start-up, so changing either requires a restart.

### `/slowstorage` directory example
```shell
.
├── pepcentric
    ├── blockMetadata.txt
    ├── dataset
    │   ├── PXD001468
    │       ├── targetindex.txt
    │       ├── Closed
    │       │   ├── b1906_293T_proteinID_01A_QE3_122212.pepXML
    │       │   └── b1906_293T_proteinID_01A_QE3_122212.auxscores
    │       ├── Open
    │       │   ├── b1906_293T_proteinID_01A_QE3_122212.pepXML
    │       │   └── b1906_293T_proteinID_01A_QE3_122212.auxscores
    │       └── mzBIN
    │           └── b1906_293T_proteinID_01A_QE3_122212.mzBIN
    ├── index
    └── params
        ├── hla.pepcentricparams
        ├── lf.pepcentricparams
        ├── lf2.pepcentricparams
        └── tmt.pepcentricparams
```

- `blockMetadata.txt` record the structure of block disk.
- `*.pepXML` files will be converted to `*.pepBIN` files in the index building step.
- `index` directory is empty at the beginning. 
- `params` directory contains PepCentric parameter files for each index. 
- The `*.pepcentricparams` file name should be the same as the index name because it is used to pair the parameters with the index.
- `targetindex.txt` record the index name for each dataset.

#### `blockMetadata.txt` example
```shell
device /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage
alloc tmt 0 200G
alloc lf 200G 800G
alloc lf2 800G 1200G
alloc hla 1200G 1400G
```
Make sure of having the permission to read and write the NVMe drive. Every index needs both a block
allocation here and a `*.pepcentricparams` file of its own name.


#### `targetindex.txt` example
```shell
tmt16
url@ https://proteomecentral.proteomexchange.org/cgi/GetDataset?ID=PXD034789
publication@ Deep Proteomics Network and Machine Learning Analysis of Human Cerebrospinal Fluid in Japanese Encephalitis Virus Infection
sample source@ patient
organ@ cerebrospinal fluid
disease@ Japanese encephalitis virus infection
```

- The first line is the name of the index
- The second line is the URL of the dataset (optional)
- the third line is the name of the publication (optional)
- The forth line is the name of the sample source (optional)
- The fifth line is the name of the organ (optional)
- The sixth line is the name of the disease (optional) 


#### `tmt.pepcentricparams` example
```shell
precursor_mass_lower = -20			# Lower bound of the precursor mass window.
precursor_mass_upper = 20			# Upper bound of the precursor mass window.
precursor_mass_units = 1			# Precursor mass tolerance units (0 for Da, 1 for ppm).
fragment_mass_tolerance = 20			# Fragment mass tolerance (window is +/- this value).
fragment_mass_units = 1			# Fragment mass tolerance units (0 for Da, 1 for ppm).
deisotope = 1			# Perform deisotoping or not

min_fragments_modelling = 2			# Minimum number of matched peaks in PSM for inclusion in statistical modeling.
min_matched_fragments = 5			# Minimum number of matched peaks for PSM to be reported.

score_tolerance = -0.01

digest_min_length = 7
digest_max_length = 50
digest_mass_range = 500.0 5000.0

search_enzyme_name = trypsin
search_enzyme_cut = KR        
search_enzyme_nocut = -       
allowed_missed_cleavage = 2      

num_enzyme_termini = 2
clip_nTerm_M = 1

variable_mod_01 = 15.9949 M 3
variable_mod_02 = 42.0106 [^ 1
variable_mod_03 = 229.162930 n^ 1

add_Cterm_peptide = 0.000000
add_Nterm_peptide = 0.000000
add_Cterm_protein = 0.000000
add_Nterm_protein = 0.000000

add_G_glycine = 0.000000
add_A_alanine = 0.000000
add_S_serine = 0.000000
add_P_proline = 0.000000
add_V_valine = 0.000000
add_T_threonine = 0.000000
add_C_cysteine = 57.021464
add_L_leucine = 0.000000
add_I_isoleucine = 0.000000
add_N_asparagine = 0.000000
add_D_aspartic_acid = 0.000000
add_Q_glutamine = 0.000000
add_K_lysine = 229.162932
add_E_glutamic_acid = 0.000000
add_M_methionine = 0.000000
add_H_histidine = 0.000000
add_F_phenylalanine = 0.000000
add_R_arginine = 0.000000
add_Y_tyrosine = 0.000000
add_W_tryptophan = 0.000000
add_B_user_amino_acid = 0.000000
add_J_user_amino_acid = 0.000000
add_O_user_amino_acid = 0.000000
add_U_user_amino_acid = 0.000000
add_X_user_amino_acid = 0.000000
add_Z_user_amino_acid = 0.000000
```


## Index building pipeline
```shell
Usage: java -cp peptidecentricsearch.jar pepcentric.indexers.IndexBuildingPipeline [data dir] [index dir]
Example: java -cp peptidecentricsearch.jar pepcentric.indexers.IndexBuildingPipeline /slowstorage/pepcentric/dataset /slowstorage/pepcentric/index
```

This step will
1. Unify mzBIN file extensions.
2. Convert pepXML files to pepBIN files.
3. Rescore PSMs in pepBIN files.
4. Build scan indices.
5. Extract scores.
6. Extract scans.

Files will be written to `/slowstorage/pepcentric/index` directory.


## Deploy indices
```shell
Usage: java -cp peptidecentricsearch.jar pepcentric.indexers.IndexWriter [blockMetadata.txt] [extracted data set 1]...
Example: java -cp peptidecentricsearch.jar pepcentric.indexers.IndexWriter blockMetadata.txt /slowstorage/pepcentric/index/dataset1/scans_100_0.01_0.1/
```

This step will write indices to the fast disk.


## Start PepCentric server
Using `run_pepcentric.sh`.

Stop every backend process using
```shell
pkill -9 -ef peptidecentricsearch
```

Each enzyme value users may submit needs a `<enzyme>_decoy_hist.bin` of its own passed through
`--decoy-peptides-bin`, otherwise its jobs fail when their results are requested.


## Steps to generate `decoy_hist.bin`
1. Search decoy proteins using
```shell
Usage: java -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.testing.ProteinJobSubmitter [job name prefix] [proteins per shard] [proteins cap] [fasta file] [enzyme]
Example: java -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.testing.ProteinJobSubmitter enzymatic-decoy 10000 20000 decoy-only.fas enzymatic
         java -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.testing.ProteinJobSubmitter nonspecific-decoy 10000 20000 decoy-only.fas nonspecific
```

2. Generate `*_decoy_hist.tsv` and `*_decoy_hist.bin` using
```shell
Usage: java -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.nullmodel.GenerateDecoyHistBin [pepcentic scratch dir] [job name pattern using % for wildcard] [decoy hist bin prefix] [index name 1] [index name 2]...
  [decoy hist bin prefix] must match the digestion parameter, such as enzymatic
Example: java -Xmx64g -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.nullmodel.GenerateDecoyHistBin pepcentric_scratch/ enzymatic-decoy-% enzymatic lf bioplex tmt
         java -Xmx64g -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.nullmodel.GenerateDecoyHistBin pepcentric_scratch/ nonspecific-decoy-% nonspecific hla
``` 
The output will be in `[pepcentic scratch dir]`.

The job name pattern has to match the prefix given in step 1, which is extended with `-<shard number>`.
