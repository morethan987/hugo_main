---
title: Qdrant Feature Guide
weight: -75
draft: true
description: Detailed documentation of Qdrant's vector database features for practical applications
slug: qdrant-feature-guide
language: en
tags:
  - Qdrant
series:
  - Technical Miscellany
series_order: 6
date: 2025-03-05
lastmod: 2025-03-05
authors:
  - Morethan
---
{{< katex >}}

{{< lead >}}   A detailed record of the core features of the Qdrant vector database, helping developers design better databases in practical applications 😋   {{< /lead >}}

## Preface  

Vector database-based RAG is the most fundamental form of RAG and is widely used in production. As the name suggests, vector databases primarily store vectors. Through vector operations, we can achieve more precise **relevance searches**, which serve as the **cornerstone** for many application scenarios.  

There are many vector database providers: Weaviate, Qdrant, Milvus, Chroma, and more. However, they are generally similar. For a detailed comparative analysis of mainstream vector databases, see: [Vector Database Comparison Report: A Detailed Comparison of 12 Mainstream Vector Databases](https://www.syndataworks.cn/blog/blog/2024-10-29-vectoc-db-eval/).  

This article focuses solely on Qdrant. All content is based on the [Qdrant Official Documentation](https://qdrant.tech/documentation/concepts/) (as of March 2025).  

This article focuses less on specific syntax and more on features and principles, helping developers quickly design product workflows without being constrained by cumbersome syntax, thereby improving imagination and efficiency.  

## Basic Data Types  

Qdrant defines some abstract data types to better handle vector data. Understanding these types is fundamental to flexible usage.  

### Point  

Points are the core data type in a vector database. All operations revolve around points.  

A very basic point contains only its vector, but typically, points are tagged with additional metadata to provide more information beyond the vector data. These tags are called [Payload]({{< relref "#payload" >}}) 👇.

**Thus: Point = Vector + Payload Tags**  

```json
// A simple point  
{  
    "id": 129,  
    "vector": [0.1, 0.2, 0.3, 0.4],  
    "payload": {"color": "red"},  
}  
```

Qdrant points can be configured with various types of vectors: **Dense Vectors, Sparse Vectors, Multi-Vectors, and Named Vectors**.  

| Vector Type | Description |
| ----------- | ----------- |
| Dense Vectors | Standard vectors; most embedding models generate this type. |
| Sparse Vectors | Variable-length vectors with few non-zero elements; typically used for exact token matching and collaborative filtering. |
| Multi-Vectors | Matrices composed of multiple dense vectors; dimensions must match across points, but the number of vectors can vary. Used to store different vector descriptions of the same target. |
| Named Vectors | A hybrid of the above types, allowing different vector types to coexist in a single point. These vectors are abstracted as named vectors. |

Basic CRUD operations are straightforward and won't be elaborated here.  

### Payload  

Additional metadata attached to vectors, described and stored using JSON syntax. Here's an example:  

```json
{  
    "name": "jacket",  
    "colors": ["red", "blue"],  
    "count": 10,  
    "price": 11.99,  
    "locations": [  
        {  
            "lon": 52.5200,  
            "lat": 13.4050  
        }  
    ],  
    "reviews": [  
        {  
            "user": "alice",  
            "score": 4  
        },  
        {  
            "user": "bob",  
            "score": 5  
        }  
    ]  
}  
```

Since this data is stored in the database, it serves a purpose: vector databases primarily rely on **semantic similarity matching**, and these tags allow you to add additional **logical filtering conditions** on top of that.  

For more on filtering, see: [Filtering]({{< relref "#filtering" >}})  

### Collection  

A collection is simply a group of points. At this level, you can define:  

1. **Similarity algorithms between points**  
2. **Vector dimensions**  
3. **Optimizer configurations**  
4. **HNSW algorithm configurations**: The key parameter to adjust is `ef`, which determines the number of neighboring nodes the algorithm visits. Higher values yield more accurate but slower queries.  
5. **WAL configurations**  
6. **Quantization configurations**  

## Common Operations  

These are the most fundamental and important operations when using a vector database.  

### Search  

In the context of vector databases, "search" primarily refers to **similarity search**. The theoretical premise is that objects with **higher similarity** in the real world are **closer** in vector space.  

The term "closer" implies a **similarity metric**. Qdrant supports several popular similarity algorithms:  

- Dot Product Similarity  
- Cosine Similarity  
- Euclidean Distance  
- Manhattan Distance  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
To improve performance, all vectors are normalized before storage. This means dot product similarity and cosine similarity are equivalent in Qdrant.  
{{< /alert >}}

For a better user experience, Qdrant provides a complete API for easy invocation: [Search API - Qdrant](https://qdrant.tech/documentation/concepts/search/#search-api).  

In summary, the functionalities you can invoke include:  

1. **Basic Operations**: Input a vector to perform similarity matching in the database. If your points store "named vectors," you need to specify which vector to use for matching.  
2. **Search Algorithm Control**: You can enable exact search, which performs similarity matching on every point, taking longer (more parameters are adjustable but rarely used).  
3. **Result Filtering**: Filter results before the actual search using *payload tags* to narrow the scope, or after the search using *similarity score thresholds*.  
4. **Result Count**: The `limit` parameter controls the number of results returned (the top `limit` most similar results).  
5. **Batch Search**: Input multiple vectors for search in one go.  
6. **Search Grouping**: Group search results by certain tags. The `group_size` parameter sets the number of results per group.  
7. **Search Planning**: Based on optional indexes, filter complexity, and the total number of points, a heuristic method selects an appropriate search approach (improving performance 🤔).  


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
If both `group_size` and `limit` are set, `limit` represents the number of groups.  
{{< /alert >}}

Additionally, sparse and dense vector searches in Qdrant have key differences:  

| Comparison | Sparse Vectors | Dense Vectors |
| ---------- | -------------- | ------------- |
| Similarity Algorithm | Defaults to dot product; no need to specify | You can specify supported algorithms |
| Search Method | Only exact search | Can use HNSW for approximate search |
| Search Results | Returns only vectors with shared non-zero elements | Returns the top `limit` vectors |

### Explore  

Explore operations are more flexible searches: they can query based on similarity as well as **dissimilarity**.  

#### Recommendation  

"Recommendation" allows you to provide both positive and negative vectors for search. Here's an official example:  

```typescript
import { QdrantClient } from "@qdrant/js-client-rest";  

const client = new QdrantClient({ host: "localhost", port: 6333 });  

client.query("{collection_name}", {  
    query: {  
        recommend: {  
            positive: [100, 231],  
            negative: [718, [0.2, 0.3, 0.4, 0.5]],  
            strategy: "average_vector"  
        }  
    },  
    filter: {  
        must: [  
            {  
                key: "city",  
                match: {  
                    value: "London",  
                },  
            },  
        ],  
    },  
    limit: 3  
});  
```


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
In the official example, 100 and 231 are vector IDs, each corresponding to a 4-dimensional vector.  
{{< /alert >}}

The `strategy` parameter controls the search algorithm. Here are the details:  

1. **Average Algorithm**: Averages positive and negative examples separately, then combines them into a final search vector.  

2. **Best Score Algorithm**: Each candidate point is matched against all positive and negative examples to compute scores. The highest scores are selected, and the final score is calculated as:  

```rust
let score = if best_positive_score > best_negative_score {  
    best_positive_score  
} else {  
    -(best_negative_score * best_negative_score)  
};  
```

3. **Negative-Only Algorithm**: Uses the best score algorithm 👆 without positive examples, yielding a reverse scoring algorithm to find the least relevant points.  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
Multi-vectors and other special vectors can also be processed with similar logic, though the syntax differs.  
{{< /alert >}}

#### Discovery  

"Discovery" operations partition the sample space. You provide pairs of **positive and negative vectors**, each dividing the space into positive and negative regions. The search returns points that **lie more in positive regions or less in negative regions**.  

Similar to [Recommendation]({{< relref "#recommendation" >}}), but here you pair positive and negative vectors together as input.  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
Due to hard partitioning, consider increasing the HNSW `ef` parameter to compensate for precision loss.  
{{< /alert >}}

Discovery operations enable Qdrant to handle two new search requirements:  

1. **Discovery Search**: Given a target vector and a set of positive-negative vector pairs as *contextual constraints*.  

2. **Region Partition Search**: A special case of discovery search 👆 where no target vector is provided. The database partitions the space directly and returns points lying most in positive regions.  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
In discovery search, contextual constraints are **enforced** with higher priority. In other words, discovery search first performs region partitioning, then standard similarity search.  
{{< /alert >}}

#### Distance Matrix  

This operation resembles batch search. Batch search inputs multiple vectors and searches for similar vectors for each. Distance matrix randomly selects a **subset of vectors**, then searches for similar vectors within this subset for each vector.  

For example, if `sample=100` vectors are selected to form a subset, and `limit=10` similar vectors are searched for each, the returned distance matrix will be a \(100 \times 10\) matrix, where each row represents the top 10 similar vectors for one point.  

This operation is typically used for data visualization or dimensionality reduction.  

### Filtering

Official English guide: [A Complete Guide to Filtering in Vector Search](https://qdrant.tech/articles/vector-search-filtering/). The guide explains how Qdrant internally performs filtering, which helps design efficient systems.  

The guide briefly lists some functionalities. For complete details, see: [Filtering](https://qdrant.tech/documentation/concepts/filtering/#filtering).  

#### Filter Conditions  

These refer to **individual filter conditions**, the basic units of filtering. Here are the types:  

| Type | Function |
| ---- | -------- |
| Match | The condition is a specific value; the attribute must exactly match it. |
| Match Any | The condition is a set of options; the attribute must match any of them. |
| Match Except | The condition is a set of options; the attribute must not match any of them. |
| Range | The condition is a range; the attribute must lie within it. |
| Values Count | The attribute is an array; filtering is based on the number of elements. |
| Is Empty | Filters based on whether the attribute exists. |

These are the basic filter types. Syntax varies for different payload types:  

- **JSON Payload**: A point's payload can be a JSON object, and any field can participate in filtering. See [Nested Key](https://qdrant.tech/documentation/concepts/filtering/#nested-key).  
- **Date Range**: Similar to numeric ranges, date ranges can also filter.  
- **Geofiltering**: Geographic locations can be filtered.  
- **Named Vectors**: Named vectors contain vectors of different dimensions. Filtering can check for the presence of specific vectors (e.g., filtering points with image embeddings).  

These basic conditions can be nested using logical keywords 👇 to form complex filters.  

#### Logical Keywords  

Similar to SQL's `AND`, `OR`, `NOT`, Qdrant uses `must`, `should`, `must_not` to express similar logic. These keywords build complex filters.  

- `must`: Returns true only if all listed conditions are met.  
- `should`: Returns true if any listed condition is met.  
- `must_not`: Returns true if none of the listed conditions are met.  

## Advanced Operations  

### Hybrid Queries  

### Optimization  

### Storage  

### Indexing  

### Snapshots  

## References  

- [Vector Database Comparison Report: A Detailed Comparison of 12 Mainstream Vector Databases | SynDataWorks](https://www.syndataworks.cn/blog/blog/2024-10-29-vectoc-db-eval/)  
- [Qdrant Official Documentation](https://qdrant.tech/documentation/concepts/)
