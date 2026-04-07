# Information Retrieval
Information retrieval (IR) is the science and process of obtaining relevant information from large collections of unstructured or semi-structured data, such as documents, images, and videos. 

Unlike traditional database searching (data retrieval), which relies on structured data and exact matches, IR focuses on satisfying a ***user's specific information need by ranking results based on relevance**. 

## Core Components and Processes
### Indexing
The problem:

Information must be store before retrieval; therefore we have to understand efficient data storage that allows for effective and efficient retrieval.

For example to answer a user query on "Wangari Maathai and the nobel prize", in a traditional database, we would need to have a structured table with specific fields (e.g., name, birthdate, achievements) and the query would need to match those fields exactly. We have to run a database query or a use `grep` command to find the exact match which will go through the entire dataset to find the exact match giving `O(n)` time complexity as this is a linear search.

For a web search this time complexity is not acceptable, as we have billions of documents to search through. 

> Indexing is the process of structuring and organizing data, typically by mapping terms or keywords to their respective document locations for fast, accurate, and efficient searching.


Solutions:
To search through large volumes of unstructured data efficiently, IR systems use indexing techniques.

1. A **term-document incidence matrix**

This a binary data structure used in information retrieval and NLP to map the presence (1) or absence (0) of unique terms (rows) across a collection of documents (columns). It facilitates **Boolean retrieval** for efficient, exact-match searches via bitwise operations (AND, OR, NOT).

Before construction of the matrix, text is processed by through techniquees including tokenization, normalization, stemming, and stop-word removal.

##### Example of a Term-Document Incidence Matrix:
| Term           | Document 1 | Document 2 | Document 3 | Document 4
|----------------|------------|------------|------------|------------|
| "Wangari"      | 1          | 0          | 1          | 0          |
| "Maathai"      | 1          | 1          | 0          | 0          |
| "Nobel"        | 1          | 0          | 0          | 1          |

The **Term Vector** for Document 1 would be [1, 1, 1], indicating that all three terms ("Wangari", "Maathai", "Nobel") are present in Document 1.


It is more efficient than a linear search, but still not ideal for large datasets due to its size and sparsity.

Limitations:
- **Size**: For large document collections, the term-document incidence matrix can become extremely large and sparse, leading to inefficient storage and retrieval.
- **Exact Match**: It only supports exact matches, which may not be sufficient for many queries that require more flexible matching (e.g., synonyms, partial matches).
- **No Frequency Data**: It does not capture how many times a term appears, only if it appears.

2. An **Inverted Index**

Offers faster retrieval by mapping each term to the documents that contain it. The index data structure is built offline (i.e., not during query time) and we get a quick lookup of `O(1)` at query-time.

An IR system can search through unstructured text and return relevant documents even if they don't contain the exact phrase "Wangari Maathai".

#### Inverted Index Construction
1. **Tokenization**: The text is broken down into individual terms or tokens.
2. **Normalization**: Terms are standardized using these techniques:
    - Case folding — converting to uniform case (e.g. "USA" → "usa").Without case folding, the same word in different cases would be stored as separate index entries, causing mismatches between queries and documents.
    - Accent/diacritic removal — "résumé" → "resume"
    - Punctuation normalisation — "co-operate" vs "cooperate"
    - Unicode normalisation — handling equivalent character representations
    - Number normalisation — "10" vs "ten"
3. **Stemming/Lemmatization**: Words are reduced to their base or root form (e.g., "dancing" becomes "dance").
4. **Stop-word Removal**: Common words that do not carry significant meaning (e.g., "the", "is") are removed to reduce noise in the index.
5. **Indexing**: Each term is mapped to a list of document IDs where it appears, along with optional metadata such as term frequency (TF) and document frequency (DF).

##### Example of an Inverted Index:
| Term           | Document IDs       |
|----------------|--------------------|
| "wangari"      | [1, 3, 5]          |
| "maathai"      | [1, 2, 5]          |
| "nobel"        | [1, 4]             |
In this example, the term "wangari" appears in documents 1, 3, and 5, while "maathai" appears in documents 1, 2, and 5. This allows the IR system to quickly identify relevant documents based on the presence of these terms, returning 1,2,3,5 as relevant documents for the query "wangari maathai".

The term in the inverted index is known as the **dictionary** and the list of document IDs is known as the **posting list**.

The dictionary is in memory and the posting list is stored on disk. 

The dictionary is implemented as a hash table or a trie or a B-tree, allowing for quick lookups of terms, while the posting list is implemented as a linked list or an array, providing the necessary information to retrieve relevant documents efficiently.

The postings can also include additional metadata such as term frequency (TF), document frequency (DF), position information, and which field the term appears in(e.g the title, the body), which can be used to calculate relevance scores during the matching and ranking phase.

Such an index is also known as a zone index, as it can store separate posting lists for different fields (e.g., title, body) to allow for field-specific weighting during ranking.

##### Zone index
A zone is a named region of a document — for example, the title, abstract, author field, or body text. A **zone index** extends the standard inverted index by recording not just which documents contain a term, but which zone within each document the term appears in.

This allows retrieval systems to weight matches differently depending on where they occur. A term match in the title is generally a stronger relevance signal than one buried in the body text. Zone indexes are the foundation of weighted zone scoring, where each zone is assigned a weight and the document score is a weighted sum of zone match indicators.

Example: A query term found in the title zone scores higher than the same term found only in the body, reflecting the higher semantic importance of titles.

##### Tiered indexing
Tiered indexing is a technique for improving query processing efficiency at large scale. Documents in the postings list are partitioned into tiers based on their quality or importance score — for example, PageRank, authority score, or a static relevance estimate computed at index time.

At query time, the system first searches only the top tier (the highest-quality documents). If enough results are found, it stops there. If not, it falls through to lower tiers. This avoids scanning the full postings list for every query, dramatically reducing retrieval latency.

Trade-off: Tiered indexing improves speed but risks missing relevant documents that sit in lower tiers. The tier cutoff must be tuned carefully.

##### Example of an Inverted Index with more metadata:

Document 1: "The quick brown fox jumped over the lazy dog."

Document 2: "Quick brown foxes leap over lazy dogs in summer." 

After text processing (tokenization, lowercasing, stemming, and removing stop words like "the" and "over"), the terms for indexing would be: 

Doc 1 terms: quick, brown, fox, jump, lazi, dog

Doc 2 terms: quick, brown, fox, leap, lazi, dog, summer 

The resulting inverted index with metadata would look like this:
| Term | Document ID | Metadata (Frequency, Positions, Offsets) |
|------|-------------|-------------------------------------------|
| brown | Doc 1 | {"freq": 1, "pos": [2], "start_offset": 8, "end_offset": 13} |
| | Doc 2 | {"freq": 1, "pos": [1], "start_offset": 6, "end_offset": 11} |
| dog | Doc 1 | {"freq": 1, "pos": [8], "start_offset": 40, "end_offset": 43} |
| | Doc 2 | {"freq": 1, "pos": [6], "start_offset": 30, "end_offset": 33} |
| fox | Doc 1 | {"freq": 1, "pos": [3], "start_offset": 14, "end_offset": 17} |
| | Doc 2 | {"freq": 1, "pos": [2], "start_offset": 12, "end_offset": 16} |
| jump | Doc 1 | {"freq": 1, "pos": [4], "start_offset": 18, "end_offset": 24} |
| lazi | Doc 1 | {"freq": 1, "pos": [7], "start_offset": 35, "end_offset": 39} |
| | Doc 2 | {"freq": 1, "pos": [5], "start_offset": 25, "end_offset": 29} |
| leap | Doc 2 | {"freq": 1, "pos": [3], "start_offset": 17, "end_offset": 21} |
| quick | Doc 1 | {"freq": 1, "pos": [1], "start_offset": 4, "end_offset": 8} |
| | Doc 2 | {"freq": 1, "pos": [0], "start_offset": 0, "end_offset": 5} |
| summer | Doc 2 | {"freq": 1, "pos": [8], "start_offset": 37, "end_offset": 43} |

The additional metadata stored for each term in a document's postings list includes: 

- Term Frequency (freq): The number of times a term appears in a specific document. This is vital for relevance scoring algorithms like TF-IDF or BM25 to determine how important a term is to a document.

- Positions (pos): The specific word positions within the document where the term occurs. This information is necessary for executing phrase queries (e.g., searching for "quick brown fox" as an exact phrase) and proximity searches.

- Character Offsets (start_offset, end_offset): The start and end character offsets of the original word in the source document. This allows for accurate searching snippets in the original text, even after the text has been tokenized and normalized.

### Matching and Ranking - Relevance Scoring
The system uses algorithms to compare the query with the index and calculate a relevance score for each document.

#### TF-IDF (Term Frequency-Inverse Document Frequency) 

TF-IDF is used in information retrieval to evaluate the importance of a term in a document relative to a collection of documents.
- **Term Frequency (TF)**: Measures how frequently a term appears in a document. It is calculated as the number of times a term appears in a document divided by the total number of terms in that document.

$$
tf = Number of times term t appears in a document / Total number of terms in the document

$$
##### Zipf's Law
Zipf's Law is an empirical observation about term frequency distributions in natural language that states that the frequency of a term is inversely proportional to its rank in the frequency table.
> The most common word appears roughly twice as often as the second most common, three times as often as the third, and so on.

Formally: frequency ∝ 1 / rank

$$
f(r) = C / r^s
$$

- where f(r) is the frequency of the term at rank `r`, `C` is a constant, and `s` is typically close to 1.

example: If "the" is the most common word in a corpus and appears 10,000 times, then the second most common word (e.g., "of") would appear approximately 5,000 times, the third most common word (e.g., "and") would appear around 3,333 times, and so on.

Zipf's Law explains why a small number of very high-frequency terms (stopwords like 'the', 'is', 'and') dominate document collections, while the vast majority of vocabulary terms are rare. This distribution motivates stopword removal and IDF weighting becausecommon terms carry little discriminative power.

The long tail of rare terms is where most of the semantic meaning lives. IDF amplifies these rare but discriminative terms in ranking.

- **Inverse Document Frequency (IDF)**: Measures how important a term is across the entire collection of documents. It is calculated as the logarithm of the total number of documents divided by the number of documents containing the term. Using logarithm helps to dampen the effect of very common terms that appear in many documents and gives more weight to rarer terms that are more likely to be relevant to the query and avoids division by zero by adding 1 to the denominator.

$$
idf = log(Total number of documents / Number of documents containing term t)

$$
The TF-IDF score for a term in a document is calculated by multiplying the term frequency (TF) by the inverse document frequency (IDF):

$$
tf-idf = tf * idf
$$

#### BM25 (Best Matching 25)
BM25 is a ranking function used in information retrieval to estimate the relevance of documents to a given search query. It is an extension of the TF-IDF model and incorporates term frequency, inverse document frequency, and document length normalization to provide more accurate relevance scores.

The BM25 score for a document with respect to a query is calculated using the formula:
$$
`BM25(D, Q) = ∑ (IDF(t) * ((TF(t, D) * (k1 + 1)) / (TF(t, D) + k1 * (1 - b + b * (|D| / avgDL)))))`

$$
Where:
- `D` is the document
- `Q` is the query
- `t` is a term in the query
- `k1` and `b` are free parameters (typically set to 1.2 and 0.75 respectively)
- `|D|` is the length of the document in terms`
- `avgDL` is the average document length in terms

`k1` controls the term frequency saturation, while `b` controls the degree of length normalization. BM25 provides a more nuanced relevance scoring mechanism compared to TF-IDF, making it a popular choice for modern information retrieval systems.

Example BM25 Calculation:

Assume we have a query "information retrieval" and a document with the following term frequencies:
| Term                | TF in Document | IDF  |
|---------------------|----------------|------|
| "information"       | 3              | 1.5  |
| "retrieval"         | 2              | 1.8  |

Assuming the document length is 100 terms, the average document length is 50 terms, and using `k1 = 1.2` and `b = 0.75`, we can calculate the BM25 score for this document with respect to the query.

```python
# Example BM25 Calculation
k1 = 1.2
b = 0.75
|D| = 100
avgDL = 50

# Calculate BM25 score for each term
bm25_score = 0
for term, tf, idf in [("information", 3, 1.5), ("retrieval", 2, 1.8)]:
    numerator = tf * (k1 + 1)
    denominator = tf + k1 * (1 - b + b * (|D| / avgDL))
    bm25_score += idf * (numerator / denominator)

print(f"BM25 Score: {bm25_score}")
```

## Evaluation
How	good or poor is the IR system?	

For	a trustworthy evaluation we need a benchmark dataset, a query set,	and	an answer_key and a meaningful	metric such that given	query q,returned ranked	list,and answer	key,computes a number that indicates the quality of	the	returned ranked list.

### Binary Relevance Metrics

Systems are assessed using precision (accuracy of results) and recall (completeness of results)

The returned results can be divided into four categories:
- **True Positives (TP)**: Relevant documents that are correctly retrieved.
- **False Positives (FP)**: Irrelevant documents that are incorrectly retrieved.
- **True Negatives (TN)**: Irrelevant documents that are correctly not retrieved.
- **False Negatives (FN)**: Relevant documents that are incorrectly not retrieved.

|                 | Retrieved (Yes) | Retrieved (No) |
|-----------------|-----------------|----------------|
| Relevant (Yes)  | True Positives (TP) | False Negatives (FN) |
| Relevant (No)   | False Positives (FP) | True Negatives (TN) |

#### Precision
Precision is the ratio of relevant documents retrieved(True Positives) to the **total number of documents retrieved**. 

It measures the accuracy of the search results. A high precision indicates that most of the retrieved documents are relevant to the query.

Precision = True Positives / (True Positives + False Positives)

#### Recall
Recall is the ratio of relevant documents retrieved(True Positives) to the total number of **relevant documents** in the collection. 

It measures the completeness of the search results. A high recall indicates that most of the relevant documents have been retrieved, even if some irrelevant documents are included.

Recall = True Positives / (True Positives + False Negatives)

#### Precision@k
Precision@k measures the precision of the top-k retrieved documents. Precision@k is useful for evaluating the quality of the top results, which is often more important in practice since users typically focus on the first few results.

It is calculated as the number of relevant documents in the top-k results divided by k where k is the number of top results considered(The rank chosen).

Precision@k = Number of relevant documents in top-k / k

Example: If a search query returns 10 documents, and 4 of them are relevant, then Precision@10 would be 4/10 = 0.4 (or 40%) and precision@5 would be 3/5 = 0.6 (or 60%) if 3 of the top 5 documents are relevant.


The choice of k depends on the context of the evaluation and the typical user behavior. For example, in web search, k might be set to 10 or 20 to reflect the number of results users typically examine.


#### Mean Average Precision (MAP)
MAP calculates the average precision for a set of queries and then takes the mean of those average precisions.

It measures how well a system ranks relevant items (e.g., documents, products) by averaging precision scores across multiple queries. Unlike simple precision, which calculates the fraction of relevant items in a single result set, MAP accounts for the order of results and their relevance across all queries.

###### Example of MAP Calculation 

To compute MAP, first calculate Average Precision (AP) for each query. AP averages precision scores at every position where a relevant item appears in the ranked list. For example, suppose a search query returns five items, and relevant items are at positions 1, 3, and 5. Precision at each relevant position is 1/1 (100%), 2/3 (~66.7%), and 3/5 (60%). AP for this query is (1 + 0.667 + 0.6) / 3 ≈ 0.756. If a second query has relevant items at positions 2 and 4, with precisions 1/2 (50%) and 2/4 (50%), its AP is (0.5 + 0.5) / 2 = 0.5. MAP is the mean of these two AP scores: (0.756 + 0.5) / 2 ≈ 0.628. This reflects the system’s overall ability to rank relevant items higher across different queries.

MAP is commonly used to compare ranking algorithms in tasks like document retrieval, product recommendations, or image search. For instance, if two search algorithms return the same number of relevant documents, MAP helps determine which algorithm places them higher in the list. 

> It’s favored over metrics like precision@k because it captures ranking quality across the entire result set, not just a fixed cutoff. MAP assumes binary relevance (items are either relevant or not), which may not suit scenarios with graded relevance such as partially relevant items.

#### F1 Score

The F1 score balances both precision and recall. It is the harmonic mean of precision and recall, providing a single score that reflects both the system's accuracy and its ability to retrieve relevant documents.

The F1 score is useful because it considers both false positives (irrelevant documents retrieved) and false negatives (relevant documents not retrieved), making it a good overall measure of an IR system’s performance. A high F1 score indicates that the system is both accurate and comprehensive.

F1 score = 2 * (Precision * Recall) / (Precision + Recall)

#### Mean Reciprocal Rank (MRR)
MRR is a metric used to evaluate the effectivenessof an information retrieval system based on the rank of the first relevant document in the search results. It is calculated as the average of the reciprocal ranks of the first relevant document for a set of queries.

$$
MRR = (1/Q) * ∑ (1/rank_i)
$$

Where Q is the total number of queries and rank_i is the rank position of the first relevant document for the i-th query. 

A higher MRR indicates that relevant documents are appearing earlier in the search results, which is desirable for user satisfaction. 

For example, if the first relevant document for a query is at rank 1, the reciprocal rank is 1/1 = 1. If it is at rank 2, the reciprocal rank is 1/2 = 0.5, and so on. 

MRR is particularly useful for evaluating systems where users are likely to click on the first relevant result they see.
### Graded Relevance Metrics
These metrics account for varying degrees of relevance rather than treating documents as simply relevant or irrelevant. They are more nuanced and better reflect user satisfaction in real-world scenarios where some results may be more useful than others.

##### Cumulative Gain (CG)
CG is a metric that sums the relevance scores of retrieved documents, giving higher scores to more relevant documents. Documents are given a rating based on their relevance to the query (e.g., 0 for irrelevant, 1 for relevant, 2 for highly relevant). Then the relevance scores of the retrieved documents are summed to calculate the CG.

It is calculated as:
$$
CG = ∑ (relevance_i)
$$

Where `relevance_i` is the relevance score of the i-th retrieved document. 

CG does not account for the position of the documents in the ranked list which is a limitation because users are more likely to interact with higher-ranked results. This is where Discounted Cumulative Gain (DCG) comes in.

#### Discounted Cumulative Gain (DCG)
DCG addresses the limitation of CG by assigning higher weights to relevant documents that appear earlier in the ranked list. 

It is calculated as:
$$
DCG = ∑ (relevance_i / log_2(i + 1))
$$

Where `relevance_i` is the relevance score of the i-th retrieved document, and the logarithmic function discounts the contribution of documents based on their rank.

#### nDCG (Normalized Discounted Cumulative Gain)
nDCG normalizes DCG by the ideal DCG (IDCG), which is the DCG of the ideal ranking where all relevant documents are ranked at the top. This allows for comparison across different queries and systems.

It is calculated as:
$$
nDCG = DCG / IDCG
$$
IDCG is calculated by sorting the retrieved documents by their relevance scores in descending order and then applying the DCG formula to this ideal ranking.

$$
IDCG = ∑ (relevance_i / log_2(i + 1))
$$

Where `IDCG` is the DCG of the ideal ranking. nDCG values range from 0 to 1, with higher values indicating better performance.  

nDCG is widely used in information retrieval evaluations because it accounts for both the relevance and the position of retrieved documents, providing a more comprehensive measure of ranking quality.

## Relevance Feedback

A technique for iteratively refining retrieval results using signals about document relevance which helps bridge the gap between a user's initial query and their true information need.

Users rarely express their information need perfectly in a single query. Relevance feedback addresses this by using judgements about retrieved documents — whether explicit, inferred, or assumed — to reformulate the query and retrieve better results in a second pass.

> Relevance feedback solves the **query formulation** problem where users struggle to articulate what they want upfront. Relevance feedback lets the system iteratively hone in on the user's true information need.

Once you have some retrieved documents, you have more information about the topic than the original query alone conveyed. Feeding that information back into the system improves both precision and recall thus improving overall accuracy of the search results.


### Types of Relevance Feedback

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; margin: 1.5rem 0;">

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #7a6df0; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Explicit</span>

**Explicit feedback**

The user directly marks retrieved documents as relevant or irrelevant. High-quality signal but rarely happens in practice — most users won't bother.

</div>

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #9e5a9e; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Implicit</span>

**Implicit feedback**

The system infers relevance from user behaviour — clicks, dwell time, scroll depth. No user effort required but noisier: a click doesn't always mean a document was useful.

</div>

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #5ae3e7; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Pseudo-Relevance</span>

**Pseudo-relevance feedback**

Assumes the top-k retrieved documents are relevant without any user input, extracts their key terms, and expands the query automatically. Fully automated but vulnerable to query drift.

</div>

</div>

### The Rocchio Algorithm

The Rocchio algorithm is the classical method for implementing relevance feedback in a vector space model. It updates the original query vector by moving it closer to relevant documents and further from non-relevant ones.

$$
q' = \alpha q_0 + \beta \left( \frac{1}{|D_r|} \sum_{d \in D_r} d \right) - \gamma \left( \frac{1}{|D_{nr}|} \sum_{d \in D_{nr}} d \right)
$$

Where:  
- $q_0$ — original query vector  
- $D_r$ — set of relevant documents  
- $D_{nr}$ — set of non-relevant documents  
- $\alpha, \beta, \gamma$ — tunable weights controlling the influence of each component  

In practice, $\gamma$ is often set to 0 or a small value — penalising non-relevant documents too aggressively can push the query into sparse regions of the vector space. A common setting is $\alpha = 1, \beta = 0.75, \gamma = 0.25$.

### How It Works Step by Step

1. **Initial retrieval**: Run the original query and return the top-k documents.
2. **Feedback collection**: User marks documents as relevant or non-relevant (explicit), or the system infers this from behaviour (implicit), or assumes the top results are relevant (pseudo).
3. **Query reformulation**: Apply the Rocchio formula (or equivalent) to compute a new query vector incorporating the feedback signal.
4. **Second retrieval**: Run the expanded query to retrieve an improved result set.

### Trade-offs

#### Benefits
- Improves both precision and recall
- Adapts to the user's true information need
- PRF requires no labelled data or user effort
- Works on top of any retrieval model

#### Limitations
- Explicit feedback rarely collected in practice
- Implicit signals are noisy proxies for relevance
- PRF vulnerable to query drift
- Adds latency from a second retrieval pass

### Relevance Feedback vs Query Expansion

> Query expansion is a mechanism — adding terms to a query. Relevance feedback is a process — using document-level judgements to drive that expansion. PRF is the overlap: it is a form of relevance feedback that uses query expansion as its implementation. Not all query expansion involves relevance feedback (e.g. thesaurus-based expansion), and not all relevance feedback uses query expansion (e.g. re-ranking based on click signals).


## Query Expansion

A technique for improving recall by enriching queries with semantically related terms — bridging the vocabulary gap between what users type and what documents contain. 

It addresses the **vocabulary mismatch** problem which occurs when different words or phrases are used to describe the same concept, leading to failures in systems like search engines, databases, or chatbots. For example, a user searching for “smartphone” might not find results labeled “mobile phone,” even though they mean the same thing.

Query expansion aims to improve recall by retrieving more relevant documents without sacrificing precision. The challenge is expanding meaningfully without introducing noise from adding irrelevant terms that could reduce precision.


#### Approaches to query expansion

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 12px; margin: 1.5rem 0;">

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #5a7d9e; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Classical</span>

**Thesaurus-based expansion**

Uses a manually or automatically constructed synonym dictionary (e.g. WordNet) to add semantically equivalent terms. Simple and deterministic, but limited by thesaurus coverage and lacks context-sensitivity.

</div>

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #6b5b95; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Classical</span>

**Pseudo-relevance feedback (PRF)**

Assumes the top-k retrieved documents are relevant, extracts their most discriminative terms, and adds these to the query for a second retrieval pass. Fully automatic but can suffer from query drift.

</div>

<div style="border-radius: 10px; padding: 14px; background: #2d2d2d; color: white; font-size: 13px;">

<span style="display: inline-block; background: #4a7c59; color: white; padding: 3px 8px; border-radius: 18px; font-size: 11px; margin-bottom: 8px;">Modern</span>

**LLM-based expansion**

Uses a large language model to predict the user's intent and generate hypothetical relevant documents or enriched query terms. Semantically rich and context-aware, but adds latency and computational cost.

</div>

</div>

---

#### Pseudo-relevance feedback

PRF is the most widely studied classical approach. The process works in two passes:

- **Pass 1:** Run the original query and retrieve the top-k documents.
- **Pass 2:** Extract high-scoring terms from those documents using a weighting function, add them to the query, and re-run retrieval.

**The query drift problem:**
If the top-k results happen to be irrelevant, PRF amplifies the error because the expanded query drifts further from the user's intent with each iteration. 
Limiting expansion to 1–2 passes and capping the number of added terms helps mitigate this.

---

#### LLM-based expansion: HyDE

A modern technique is Hypothetical Document Embeddings (HyDE). Instead of expanding the query with keywords, an LLM generates a hypothetical document that would answer the query. This document is then encoded into a dense vector and used to retrieve real documents via approximate nearest neighbour search.

This sidesteps the vocabulary mismatch problem entirely, using the LLM's broad world knowledge to bridge the semantic gap between short queries and long documents.

---

####  Multi-Query Expansion
Uses an LLM to generate multiple variations of the user's original query, covering different potential perspectives or interpretations.

Example: For the query "best restaurants in New York," the LLM might generate:
- "top-rated dining establishments in NYC"
- "popular eateries in New York City"
- "highly recommended restaurants in NYC"

---


#### Trade-offs

##### Benefits
- Improved recall for semantically rich queries
- Handles vocabulary mismatch and synonymy
- Can be applied on top of any retrieval model
- PRF requires no labelled data

##### Limitations
- Risk of query drift with poor top-k results
- Can reduce precision if expansion is too aggressive
- LLM-based methods add latency
- Thesaurus methods are domain-limited

---

#### When to use query expansion

Query expansion is most valuable in domains where vocabulary varies significantly across documents and users — such as medical or legal search. It is less necessary for navigational queries (where the user knows exactly what they want) and can hurt precision in short-query settings where added terms introduce ambiguity.

In modern hybrid retrieval systems, dense retrieval already handles much of the semantic matching that expansion was designed to address — but PRF and LLM-based expansion remain useful as lightweight, complementary layers on top of any retrieval pipeline.


## Resources
- [Introduction	to Information Retrieval & Web Search](https://www.cs.jhu.edu/~kevinduh/notes/jsalt19-kduh-IntroIR.pdf)

- [Introduction to Information Retrieval](https://web.stanford.edu/class/cs276/handouts/lecture1-intro-handout-1-per.pdf)

- [Basic Information Retrieval — Boolean Retrieval Model](https://bajpaihimanshu.medium.com/basic-information-retrieval-problem-boolean-retrieval-model-ed05f3d74ddc)

- [Detailed IR](https://svrec.ac.in/docs/CAI/lecture/IV-I-IIRS-LECTURE%20NOTES.pdf)

- [Measuring Search: Metrics Matter](https://jamesrubinstein.medium.com/measuring-search-metrics-matter-de124c2f6f8c)
