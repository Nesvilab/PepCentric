# PepCentric backend server

## Requirements
1. Linux
2. Java 11+
3. GCC: in order to get the flags for `open()` in `jaydio`


## Disk related command reference
- `sudo fdisk -l`
- `sudo file -s /dev/nvme1n1`
- `sudo mkfs -t xfs /dev/nvme1n1`
- `sudo mkdir /slowstorage`
- `sudo mount /dev/nvme1n1 /slowstorage`


## Install and configure Tomcat 9 on Ubuntu
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

__Note:__ Must put `mysql-connector-java-\*.jar` to `/var/lib/tomcat9/lib/`` and `/usr/share/tomcat9/lib/`` because we do not pack it in the war to prevent an issue in Tomcat.

### Useful tips
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

### Useful commands
```shell
mysqldump -u pepcentric -p --no-tablespaces --databases fragmentsearcher > dump.sql # backup a database
mysql -u pepcentric -p fragmentsearcher < dump.sql # restore a database
sudo service mysql start
# If there is a "Failed to start mysql.service: Unit mysql.service is masked" error, run sudo systemctl unmask mysql.service
```


## Directory structure
### `~/pepcentric` directory example.
`pepcentric_scratch` directory is empty at the beginning. It contains users' requests and logs.
```shell
.
├── mysql-connector-java-8.0.33.jar
├── nonspecific_decoy_hist.bin
├── pepcentric_scratch
├── run_pepcentric.sh
└── enzymatic_decoy_hist.bin
```

`*.bin` are decoy histograms. Please refer to [Steps to generate `decoy_hist.bin`](#steps-to-generate-decoy_histbin) for how to generate it.

`run_pepcentric.sh` example.
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
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/lf/ /slowstorage/pepcentric/index/lf/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/tmt.pepcentricparams > searchserver_lf.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/lf2/ /slowstorage/pepcentric/index/lf2/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/tmt.pepcentricparams > searchserver_lf2.log 2>&1 &
nohup java -Xmx20g -cp $pepcentric_jar pepcentric.searcher.SearchServer $port /slowstorage/pepcentric/index/hla/ /slowstorage/pepcentric/index/hla/scans_100_0.01_0.1/deployed.dat $search_dir $params_dir/tmt.pepcentricparams > searchserver_hla.log 2>&1 &
```

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
pepcentric.repo.overlay.prefix =  
pepcentric.repo.overlay.overlay_name =  
```

### `/slowstorage` directory example.
`blockMetadata.txt` record the structure of block disk. `*.pepXML` files will be converted to `*.pepBIN` files in the index building step. `index` directory is empty at the beginning. `params` directory contains modifications for each index. `targetindex.txt` record the index name.
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
        ├── lf.pepcentricparams
        ├── lf2.pepcentricparams
        └── tmt.pepcentricparams
```

`blockMetadata.txt` example.
```shell
device /dev/disk/by-id/nvme-Amazon_EC2_NVMe_Instance_Storage
alloc tmt 0 200G
alloc lf 200G 800G
alloc lf2 800G 1200G
```
Make sure of having the permission to read and write the NVMe drive.

`tmt.pepcentricparams` example.
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
Example: java -Xmx64g -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.nullmodel.GenerateDecoyHistBin pepcentric_scratch/ decoy-% enzymatic lf bioplex tmt
         java -Xmx64g -cp peptidecentricsearch.jar:mysql-connector-java-8.0.33.jar pepcentric.nullmodel.GenerateDecoyHistBin pepcentric_scratch/ decoy-% nonspecific hla
``` 
The output will be in `[pepcentic scratch dir]`.

