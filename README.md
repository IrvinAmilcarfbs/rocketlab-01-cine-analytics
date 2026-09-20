# 🎬 CineData Analytics — Pipeline de Dados End-to-End

Pipeline de engenharia de dados desenvolvido para o **Rocket Lab 2026.2 (Visagio)**, implementando um Data Lakehouse completo no Databricks seguindo a **Arquitetura Medalhão (Bronze → Silver → Gold)**.

O projeto estrutura uma base de filmes (TMDB/IMDb combinados), originalmente entregue suja e fragmentada em 5 arquivos CSV, transformando-a em Data Marts analíticos, uma modelagem dimensional (Star Schema) e uma base de contexto para um assistente de IA (RAG).

---

## 🏗️ Arquitetura

```
Landing (CSVs) → Bronze → Silver → Gold
                                    ├── Star Schema (BI)
                                    └── Contexto GenAI (RAG)
```

| Camada | Objetivo |
|---|---|
| **Bronze** | Ingestão crua dos 5 CSVs + cotação do dólar (API Banco Central), sem alterações, em Delta/Append |
| **Silver** | Limpeza, padronização, deduplicação e tipagem correta, em português |
| **Gold** | Star Schema (fato + dimensões + bridges) e tabela de contexto para IA |

---

## 📁 Estrutura do repositório

```
├── notebooks/
│   ├── Landing_to_Bronze.ipynb    # Ingestão + API do Banco Central
│   ├── Bronze_to_Silver.ipynb     # Limpeza e tratamento de dados
│   └── Silver_to_Gold.ipynb       # Star Schema + Contexto GenAI + Analytics
├── job/
│   └── job.yaml                   # Definição do Databricks Workflow
├── evidencias/
│   └── print_job_sucesso.png      # Execução do Job com dependências
└── README.md
```

---

## 🧱 Modelagem Gold (Star Schema)

**Fato:** `fact_movies_performance` — métricas financeiras e de engajamento, grão de um registro por filme.

**Dimensões:** `dim_movies`, `dim_genres`, `dim_people`, `dim_companies`, `dim_reviews`

**Bridge tables** (relações N:N): `bridge_movie_genre`, `bridge_movie_person`, `bridge_movie_company`

**Contexto GenAI:** `gold_genai_movies_context` — documento textual por filme, pronto para vetorização em um banco de dados vetorial (RAG).

---

## ⚙️ Orquestração

Job `CineData_PipeLine` com 3 tasks encadeadas via dependência explícita:

```
to_Bronze → to_Silver → to_Gold
```

Agendamento diário configurado via Databricks Workflows, simulando uma rotina de atualização em produção.

---

## 🔍 Desafio de Analytics

Resolvido dentro do notebook `Silver_to_Gold.ipynb`, respondendo a 6 perguntas de negócio sobre receita, popularidade, gêneros, ranking de receita, participação de atores e lucro por produtora.

---

## 🛠️ Principais desafios de qualidade de dados tratados

A base foi entregue **intencionalmente suja**, exigindo tratamento robusto em várias frentes:

- **Column shift**: valores de colunas vizinhas (sinopses, idiomas, paths de imagem) vazando para campos numéricos e categóricos
- **Formatos inconsistentes**: datas multi-formato, separadores decimais mistos (`.` e `,`), símbolos monetários variados (`$`, `USD`, abreviações como `34.0M`)
- **Duplicidade de registros**: tanto por chave (`id_filme`) quanto por variações cosméticas de título (capitalização/espaçamento)
- **Tratamento de nulos**: uso de `try_cast`/`coalesce` para evitar que operações e concatenações falhem silenciosamente diante de dados ausentes
- **Limitação conhecida**: a base combina TMDB/IMDb sem uma chave de deduplicação de entidade totalmente confiável — alguns filmes aparecem com múltiplas variações de título e `id` distintos (ex.: traduções regionais), o que pode inflar levemente contagens agregadas. Uma melhoria futura seria usar identificadores nativos (ex. `tconst` do IMDb) como chave de correspondência entre as fontes.

---

## 👤 Autor

Irvin — Engenharia de Dados, Rocket Lab 2026.2