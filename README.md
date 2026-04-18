# Linkedinlab
# Projet : Analyse des Offres d’Emploi LinkedIn avec Snowflake

## Présentation

Ce projet a pour objectif d’analyser un jeu de données LinkedIn dans **Snowflake** afin d’étudier le marché de l’emploi à partir de plusieurs tables CSV et JSON disponibles dans un bucket S3 public.

Le travail a été organisé avec une architecture **Medallion** :

- **Bronze** : données brutes
- **Silver** : données nettoyées
- **Gold** : données prêtes pour l’analyse

Les tables exploitées dans cette version sont :

- `job_postings.csv`
- `benefits.csv`
- `job_skills.csv`
- `job_industries.json`

---

## Objectifs

Les analyses demandées sont :

1. Top 10 des titres de postes les plus publiés par industrie  
2. Top 10 des postes les mieux rémunérés par industrie  
3. Répartition des offres d’emploi par taille d’entreprise  
4. Répartition des offres d’emploi par secteur d’activité  
5. Répartition des offres d’emploi par type d’emploi  

---

## Technologies utilisées

- **Snowflake**
- **SQL**
- **Streamlit in Snowflake**
- **GitHub**

---

## Structure du projet

```text
linkedin_lab
├── bronze
│   ├── job_postings
│   ├── benefits
│   ├── job_skills
│   └── job_industries
├── silver
│   ├── job_postings
│   ├── benefits
│   ├── job_skills
│   └── job_industries
└── gold
    ├── job_postings
    ├── benefits
    ├── job_skills
    ├── job_industries
    └── job_postings_full
````
---

## 5. Étapes de réalisation

### 5.1 Création de la base et des schémas

```sql
CREATE DATABASE IF NOT EXISTS linkedin_lab;
USE DATABASE linkedin_lab;

CREATE SCHEMA IF NOT EXISTS bronze;
CREATE SCHEMA IF NOT EXISTS silver;
CREATE SCHEMA IF NOT EXISTS gold;
```
### 5.2 Sélection du warehouse

Avant toute opération de chargement ou de transformation, il est nécessaire de sélectionner un **warehouse** actif dans Snowflake.  
Le warehouse représente la ressource de calcul utilisée pour exécuter les requêtes SQL.

Sans cette instruction, certaines commandes comme `COPY INTO`, `CREATE TABLE AS SELECT` ou d’autres traitements peuvent échouer.

```sql
USE WAREHOUSE COMPUTE_WH;
```
### 5.3 Création du stage S3 et des formats de fichiers

Cette étape consiste à configurer l’accès aux fichiers sources stockés dans le bucket S3 public fourni pour le projet.  
Un **stage externe** est créé afin de référencer ce bucket dans Snowflake.

Ensuite, deux **formats de fichiers** sont définis :

- un format **CSV** pour les fichiers tabulaires,
- un format **JSON** pour les fichiers structurés en tableau JSON.

```sql
USE SCHEMA bronze;

CREATE OR REPLACE STAGE linkedin_stage
  URL = 's3://snowflake-lab-bucket/'
  COMMENT = 'Stage S3 public contenant les fichiers LinkedIn';

LIST @linkedin_stage;

CREATE OR REPLACE FILE FORMAT csv_format
  TYPE = 'CSV'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1
  NULL_IF = ('NULL', 'null', 'N/A', '')
  EMPTY_FIELD_AS_NULL = TRUE
  TRIM_SPACE = TRUE
  DATE_FORMAT = 'AUTO'
  TIMESTAMP_FORMAT = 'AUTO';

CREATE OR REPLACE FILE FORMAT json_format
  TYPE = 'JSON'
  STRIP_OUTER_ARRAY = TRUE
  STRIP_NULL_VALUES = FALSE;
```
## 6. Couche Bronze : chargement des données brutes

La couche **Bronze** contient les données brutes, c’est-à-dire les données importées directement depuis les fichiers du bucket S3, sans transformation avancée.

### 6.1 Table `bronze.job_postings`

Cette table contient les informations détaillées sur les offres d’emploi LinkedIn.

```sql
CREATE OR REPLACE TABLE bronze.job_postings (
  job_id                     VARCHAR,
  company_name               VARCHAR,
  title                      VARCHAR,
  description                TEXT,
  max_salary                 FLOAT,
  med_salary                 FLOAT,
  min_salary                 FLOAT,
  pay_period                 VARCHAR,
  formatted_work_type        VARCHAR,
  location                   VARCHAR,
  applies                    INT,
  original_listed_time       BIGINT,
  remote_allowed             BOOLEAN,
  views                      INT,
  job_posting_url            VARCHAR,
  application_url            VARCHAR,
  application_type           VARCHAR,
  expiry                     BIGINT,
  closed_time                BIGINT,
  formatted_experience_level VARCHAR,
  skills_desc                TEXT,
  listed_time                BIGINT,
  posting_domain             VARCHAR,
  sponsored                  BOOLEAN,
  work_type                  VARCHAR,
  currency                   VARCHAR,
  compensation_type          VARCHAR
);

COPY INTO bronze.job_postings
FROM @bronze.linkedin_stage/job_postings.csv
FILE_FORMAT = (FORMAT_NAME = bronze.csv_format)
ON_ERROR = 'CONTINUE';
```
### 6.2 Table `bronze.benefits`

Cette table contient les avantages proposés dans les offres d’emploi.

```sql
CREATE OR REPLACE TABLE bronze.benefits (
  job_id   VARCHAR,
  inferred BOOLEAN,
  type     VARCHAR
);

COPY INTO bronze.benefits
FROM @bronze.linkedin_stage/benefits.csv
FILE_FORMAT = (FORMAT_NAME = bronze.csv_format)
ON_ERROR = 'CONTINUE';
```

