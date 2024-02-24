+++
title = 'Building a Rag Part 3'
date = 2024-02-12T20:05:43+01:00
+++


In day 2, we setup metaflow to iterate faster, and we built a pipeline
 for scraping the markdown repo docs, clean and split it into chunks, then 
 embed those chunks and storing them in a new vector database, LanceDB.

Now, we can test our RAG app using the streamlit UI, and check if everything
 works fine.
 
 <-- insert screen of failure -->     
 
As you can see, it hallucinates on this subject. I debugged it like this :
- Check what was the context given for that query
- The context is absolutely not relevant
- Check the dataframe before embedding to see if there is anything regarding DPO
- There isn't, so it either wasn't scraped at all, or was filtered out during preprocessing steps
- My intuitions were : 
    - it was a too small chunk of text that filtered out
    - or, it comes from a special `.mdx` file ([Docusaurus](https://docusaurus.io/fr/docs/markdown-features))
    that is not handled by the crawler.   
- Both proved to be wrong 😂
    - Our thresholds are big enough that the chunk in HF docs fits : ```df = df[df.word_count > 15] ; df = df[df.char_count > 50]```
    - The crawler do handles `.mdx` files

The issue was some special filter in the open-source code I took that was parsing the mdx file :

```python
def get_contents_until_next_section(lines, start_idx=0):
    """
    Get the contents of a section of a docusaurus markdown file.
    """
    _i = start_idx + 1
    # N = total number of lines in the current file
    N = len(lines)
    contents = []
    # Stop when we reach the end of the file or a new top level section
    while _i < N and not lines[_i].startswith("#"):
        try:
            if lines[_i + 1].startswith("-"):
                break
        except IndexError:
            pass  # end of file
        contents.append(lines[_i])
        _i += 1
    return " ".join(contents).strip()
```

Here is the beginning of the raw `.mdx` content from the part about DPO:

```markdown
# DPO Trainer

TRL supports the DPO Trainer for training language models from preference data, as described in the paper [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/abs/2305.18290) by Rafailov et al., 2023. For a full example have a look at  [`examples/scripts/dpo.py`](https://github.com/huggingface/trl/blob/main/examples/scripts/dpo.py).


The first step as always is to train your SFT model, to ensure the data we train on is in-distribution for the DPO algorithm.

## Expected dataset format

The DPO trainer expects a very specific format for the dataset. Since the model will be trained to directly optimize the preference of which sentence is the most relevant, given two sentences. We provide an example from the [`Anthropic/hh-rlhf`](https://huggingface.co/datasets/Anthropic/hh-rlhf) dataset below:

<div style="text-align: center">
<img src="https://huggingface.co/datasets/trl-internal-testing/example-images/resolve/main/images/rlhf-antropic-example.png", width="50%">
</div>

Therefore the final dataset object should contain these 3 entries if you use the default `DPODataCollatorWithPadding` data collator. The entries should be named:

- `prompt`
- `chosen`
- `rejected`

for example:
``` 
 
Can you spot the issue ? 🤓
 
Yes, the `if lines[_i + 1].startswith("-"):` filter will exit the parsing loop at
 the first bullet point ! 
 Not sure why Outerbounds (devs who worked on the original code I used) wanted that, but I removed the filter, reran the
   pipeline, and it worked better ! 
   

<-- insert screen of success with new UI --> 


Next steps:
