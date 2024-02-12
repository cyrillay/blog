+++
title = 'Journey into building a custom RAG (Day 2)'
date = 2024-02-09T01:23:06+01:00
+++


Now, I want to check if the context is well retrieved.
I did an experiment where I took one part of huggingface TRL library that is supposed to be included in my embeddings,
  regarding DPO:

![dpo docs](/dpo-docs.png)


I asked my model questions about DPO, but it wouldn't get them right, it seems like he didn't get it in its context.
 
Thanks to my metaflow pipeline I could access the dataframe containing the data before embedding 
and indexing (`Flow.run.df`), and check what's going on : it's not in the dataframe, so it was either 
filtered out, or not scraped at all.

At this moment, I figured out I am probably going to perform multiple experiments : adjusting the filters, parsing techniques, etc,
so I figured I will need a good tool to track the experiments, quickly retrieve the artifacts, etc.
Time to deploy metaflow service, DB and UI locally, but with docker so that we can easily push it to the cloud 
if I need it at some point.

To run the metaflow service (which includes one docker for metadata and one for UI backend) : 
```bash
git clone https://github.com/Netflix/metaflow-service.git && cd metaflow-service
docker-compose -f docker-compose.development.yml up
``` 
It runs on the 8083 port by default.

Now run the metaflow UI front-end, using the endpoint of your just deployed service : 

```
git clone https://github.com/Netflix/metaflow-ui/ && cd metaflow-ui
docker build --tag metaflow-ui:latest .
docker run -p 3000:3000 -e METAFLOW_SERVICE=http://localhost:8083/ metaflow-ui:latest
```

![Metaflow UI](/metaflow-ui.png)

It works 🥳 


### Switch to lanceDB

I wanted to persist my embeddings so that they are not recomputed each time (it takes CPU/GPU time if done locally, money 
if done with openAI's embedding)
For now it was persisted on the disk by default, but I want to switch to a proper vector DB : 
- more efficient for querying
- closer to a production setup

I heard from my friend in ML that lanceDB was currently a great choice (open-source, very efficient (written in Rust), 
easy to use), but if you need to do it yourself, I suggest comparing the available solutions yourself.
Here's a [helpful pdf](https://tge-data-web.nyc3.digitaloceanspaces.com/docs/Vector%20Databases%20-%20A%20Technical%20Primer.pdf) to help you in this task.

Here is how to do it : 

### Create the Vector DB 
```python

vector_store = LanceDBVectorStore(uri="./data_lanceDB") # Switch to a 
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_vector_store(
    vector_store, storage_context=storage_context
)
```

### Embed your data and insert it in the vector store
```python

# Get the latest execution of our markdown processing pipeline with metaflow
for _run in Flow('DataTableProcessor'):
    if _run.data.save_processed_df:
        run = _run
        break
df = run.data.processed_df

vector_store = LanceDBVectorStore(uri="./data_lanceDB")
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_documents(
    [Document(t) for t in df.contents], storage_context=storage_context,
    )
```

Under the hood, the last instruction actually embeds our data using the previously defined `service_context` (See
[part 1](https://blog.lays.pro/posts/building-a-rag-part-1/#make-iteration-cycles-cheap) of this blog series), if
 defined globally (`set_global_service_context(service_context)`), or you can also pass it 
 as a `service_context=service_context` parameter of the `VectorStoreIndex.from_documents().`

### Retrieve your existing embeddings
```python

def load_index():
    vector_store = LanceDBVectorStore(uri="./data_lanceDB")
    storage_context = StorageContext.from_defaults(vector_store=vector_store)
    index = VectorStoreIndex.from_vector_store(
        vector_store, storage_context=storage_context
    )
    return index

```

### Next Steps

- Check how the default prompt logic works with LlamaIndex
- Verify that the model can use both of it's existing knowledge,
 and the onf from our embeddings
- Switch OpenAI's gpt to a local open source LLM for low cost dev iteration (even if the price of gpt-3.5 is super low,
 I'm also interested in doing all of this with a local model)

See you for day 3 !