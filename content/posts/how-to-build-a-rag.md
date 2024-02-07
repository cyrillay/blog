+++
title = 'Develop a Retrieval Augmented Generation (RAG) LLM Application'
date = 2024-02-06T23:09:33+01:00
+++

This article explains how to customize a large language model for your specific needs.
 
There are many approaches to adapt a Large Language Model (LLM) for your data, with the goal of enhancing the responses relevance.

1. Prompt engineering : you spend time working on a good prompt, eventually a templated prompt where you
can include your data. This is the fastest way, but will also quickly become limited, as the context size 
(maximum input you can give to a model) is usually in the range of 4 - 16K tokens (3 - 11K words), and it is shared between question
and response. So you won't be able to include all of your data in the prompt (or you will do so at the expense of the response length)   

2. Fine-tuning : you can take existing models and fine-tune them with your own data. You could use openAI's 
fine-tuning API (can become expensive if you don't get it right the first time), or fine-tune an open-source model, 
like mistral, mixtral, llama2. This approach yields great results, but is still expensive in terms of engineering
resources and computing costs, as one needs to go through model training iteration loops, harder than iterating over a prompt.
More information on when to fine-tune [here](https://platform.openai.com/docs/guides/fine-tuning/when-to-use-fine-tuning). 

3. Training from scratch : this is so prohibitively expensive that only companies like OpenAI, Meta, Google can afford
the 1000s of GPUs required to perform what we call the **pretraining** (understand basic language and concepts by learning to predict the next token) 

4. Retrieval Augmented generation (RAG) : it is a good trade off with medium engineering complexity and costs,
 and high impact on the accuracy and relevance of the augmented model. In a way, it's a bit like prompt engineering on steroids.
 Let's deep dive into it and see how to make one.

RAG is a technique for enhancing the accuracy, reliability and relevance of generative AI models by giving 
it ultra relevant facts at query time, previously fetched from internal or external sources and vectorized as embeddings.

Here is how such a system can be built :
 
![RAG architecture](/rag-archi.png)

Let's dive into building a RAG-based LLM application using publicly available data. 

### What we'll cover:

- 💻 Setting up and processing workloads locally.
- 🚀 Scaling your solution to the cloud effortlessly.
- 💡 Enhancing performance with reranking, fine-tuning, and prompt engineering.

### Steps

#### 1. Get the data

Start by gathering data from public sources. For this project, we aim for a structure like this:

```markdown
| url | section | text |
| --- | ------- | ---- |
| https://example.com | Introduction | "Here's how to start with LLM..." |
| https://example.com | Getting Data | "Data is crucial for training LLMs..." |
| https://example.com | Training | "To train your model, begin with..." |
```

Future articles will delve into using internal/proprietary databases.

#### 2. Preprocess the data

Normalize data by chunking larger sections and combining smaller ones. Here’s a simple Python snippet to chunk text data:

```python
from textwrap import wrap

def chunk_text(text, chunk_size=512):
    return wrap(text, chunk_size)

# Example usage
chunked_section = chunk_text(long_text_section)
```

#### 3. Transform text data to embeddings

Embeddings turn text into a numerical format that's understandable by LLMs. We'll use the Instructor👨‍🏫 model for its performance:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('hkunlp/instructor-large')

def embed_text(text):
    return model.encode(text)

# Example usage
embedded_text = embed_text("Your text here")
```

> **Note:** While we start with embeddings for efficiency, exploring LLMs for the top k chunks and reranking with tools like Cohere Rerank could further refine relevance.

#### 4. Store the data in a vectorDB

We'll use pgvector for storage. Here’s how to set it up with PostgreSQL:

```sql
CREATE EXTENSION pgvector;
CREATE TABLE documents (id SERIAL PRIMARY KEY, text_vector vector(512));
```

And inserting data:

```python
import psycopg2

# Connect to your PostgreSQL database
conn = psycopg2.connect("dbname=your_db user=your_user")
cur = conn.cursor()

# Insert the embedded text
cur.execute("INSERT INTO documents (text_vector) VALUES (vector(%s))", (embedded_text,))
conn.commit()
```

#### 5. Perform inference

For inference, we retrieve relevant documents, optionally rerank them, and compile a contextual prompt:

```python
def query_vector_db(query_embedding, top_k=5):
    # Query the top_k most similar vectors
    cur.execute("SELECT id, text_vector <-> vector(%s) AS distance FROM documents ORDER BY distance ASC LIMIT %s", (query_embedding, top_k))
    return cur.fetchall()

# Example: embedding a query and retrieving related documents
query_embedding = embed_text("Your query here")
related_documents = query_vector_db(query_embedding)
```

### Conclusion

Building a RAG-based LLM from scratch teaches you the nuts and bolts of modern NLP applications, from data preprocessing to deploying scalable solutions. This guide is just the beginning; stay tuned for more in-depth exploration of fine-tuning and prompt engineering techniques.

