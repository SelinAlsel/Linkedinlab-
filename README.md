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
