{\rtf1\ansi\ansicpg1252\cocoartf2865
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx566\tx1133\tx1700\tx2267\tx2834\tx3401\tx3968\tx4535\tx5102\tx5669\tx6236\tx6803\pardirnatural\partightenfactor0

\f0\fs24 \cf0 # MovieLens-32M + Tag Genome 2021 \'97 Data Audit\
\
## Executive summary (non-technical)\
\
- We analyzed the latest MovieLens catalog and the 2021 Tag Genome.\
- We have **\{overlap_ratio_pct\}%** semantic coverage from Tag Genome across the catalog, with a tag vocabulary of **\{n_tags\} tags**.\
- **\{pct_ge5_tags_08\}%** of all movies have **\uc0\u8805 5 high-confidence tags** (relevance \u8805  0.8), and **\{pct_ge5_tags_06\}%** pass at a looser threshold (\u8805  0.6).  \
  \uc0\u8594  This is enough to give **good cold-start recommendations** and **clear \'93why this\'94 explanations** on day one.\
- Ratings volume is **\{n_ratings:,\}** across **\{n_users:,\} users** and **\{n_movies_in_ratings:,\} movies** (time span: **\{time_range_pretty\}**), which we\'92ll use to add **collaborative filtering** and a **hybrid model** that improves as the user interacts.\
- **Joinability is strong**: **\{n_tagged_and_tmdb:,\}** of the tag-covered movies also have TMDb IDs, so we can fetch synopses/posters to **tag new releases** and **explain recommendations**.\
\
> Bottom line: We can ship a **content-based recommender with explanations now**, then blend in collaborative signals and small-LLM tagging to cover gaps and new items.\
\
---\
\
## Key figures\
\
| Metric | Value |\
|---|---:|\
| Movies in catalog (`movies.csv`) | \{n_movies:,\} |\
| Movies with Tag Genome scores | \{n_tagged_movies:,\} |\
| Coverage (Genome \'f7 Movies) | \{overlap_ratio_pct\}% |\
| Tag vocabulary (unique tags) | \{n_tags:,\} |\
| % movies with \uc0\u8805 5 tags @ 0.8 | \{pct_ge5_tags_08\}% |\
| % movies with \uc0\u8805 5 tags @ 0.6 | \{pct_ge5_tags_06\}% |\
| Movies with TMDb ID | \{n_with_tmdb:,\} |\
| Tag-covered \uc0\u8745  TMDb | \{n_tagged_and_tmdb:,\} |\
| Users (ratings) | \{n_users:,\} |\
| Movies (ratings) | \{n_movies_in_ratings:,\} |\
| Ratings total | \{n_ratings:,\} |\
| % users with \uc0\u8804 3 ratings | \{pct_users_le3_ratings\}% |\
| % movies with \uc0\u8804 5 ratings | \{pct_movies_le5_ratings\}% |\
| Ratings time range | \{time_range_pretty\} |\
\
---\
\
## What this means for the product\
\
- **Great first session:** if a user asks for \'93sci-fi thriller, not gory,\'94 we map that to tags and show **Top-5 immediately**, with **human-readable reasons** (\'93near-future, conspiracy, cerebral\'94).\
- **Improves with feedback:** quick \uc0\u55357 \u56397 /\u55357 \u56398  or 1\'965 stars updates their **taste vector** and refreshes the list in seconds.\
- **Covers the long tail:** where official tags are missing, we\'92ll use a **small fine-tuned LLM** on synopses to assign tags and keep explanations grounded.\
\
---\
\
## Technical details\
\
### Datasets\
- **MovieLens-32M**: `movies.csv`, `ratings.csv`, `links.csv`  \
- **Tag Genome 2021**: `scores/glmer.csv` (chosen), `raw/tags.json`  *(column names normalized to `movieId, tagId, relevance`)*\
\
### Method\
1. **Coverage & density**  \
   - `n_movies`, `n_tagged_movies`, `overlap_ratio`  \
   - Per-movie strong-tag counts at \uc0\u964  \u8712  \{0.8, 0.6\}; compute **% with \u8805 5 tags** at each \u964 \
2. **Joinability for text**  \
   - Intersect Genome-covered movies with those having `tmdbId`\
3. **Ratings snapshot**  \
   - `n_users`, `n_movies_in_ratings`, `n_ratings`, `% users \uc0\u8804 3`, `% movies \u8804 5`, and time span\
\
### Sanity checks\
- Relevance in **[0,1]**: \{relevance_ok\}  \
- Duplicates `(movieId, tagId)`: \{duplicates_status\}  \
- Movies in Genome but not in `movies.csv`: \{genome_not_in_movies\}  \
- Movies in `movies.csv` with zero Genome rows: \{movies_without_genome\}\
\
---\
\
## Next steps (v1 \uc0\u8594  v2)\
\
1) **Artifacts (v1, Genome)**: build `movie\'d7tag` CSR (`tag_matrix.npz`), `movie_index.csv`, `tag_vocab.csv`  \
2) **Content-only Top-N** with \'93why\'94 (top shared tags) + diversity (MMR)  \
3) **Eval harness** (temporal split): Recall@K/NDCG@K + new-user/new-item slices  \
4) **Collaborative baseline** (ALS/LightGCN) \uc0\u8594  **hybrid** with \u945 -schedule  \
5) **Tag expansion (v2)**: supervised text model / small LLM to back-fill tags for the rest of the catalog\
\
}