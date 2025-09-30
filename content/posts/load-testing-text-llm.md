+++
title = 'Load Testing Text Llm'
date = 2025-09-30T15:17:57+02:00
+++

# Load Testing Open-Source LLMs on H100  

I am working as an LLMOps engineer since a few years, currently on a mission for the French government.  
Our team is building an API gateway to provide French administrations with access to open-source large language models (LLMs).  
I recently focused on **load testing newer models** to evaluate if they could become candidates for production.  

This post shares results and methodology for **text models**.  
(Next week I’ll publish the same analysis for **audio models**.)  

---

## Benchmark Setup  

All benchmarks were run with the following setup:  

- **Hardware:** 1× NVIDIA H100 (80 GB VRAM), 24 CPU cores, 230 GB RAM  
- **Framework:** [vLLM 0.10.2](https://github.com/vllm-project/vllm)  
- **Environment:** Kubernetes cluster (cloud deployment)  
- **Dataset:** prompts sampled randomly from [Alpaca dataset](https://github.com/tatsu-lab/stanford_alpaca)  
- **Prefix caching:** disabled (to avoid biased results)  
- **Protocol:**  
  - For each concurrency level, run 3 iterations  
  - Results averaged to reduce variability from prompt length and output size  

---

## Throughput vs. Latency  

📊 *[placeholder: graph TTFT comparison across models]*  
📊 *[placeholder: graph throughput comparison across models]*  

---

## A Note on “Concurrent Requests” vs. “Users”  

In practice, the number of concurrent requests is not equal to the number of simultaneous users.  
Users do not all send queries at the exact same time.  

Example:  
- 100 users  
- Each sends a request every ~40 s  
- Average request takes 20 s to process  

On average, each user is “active” (waiting for a response) 20/40 = 50% of the time.  
So 100 users ≈ **50 concurrent requests**.  

This estimation is rough and depends on the type of usage:  
- Human interacting with a chatbot → lower concurrency  
- Automated classification pipeline → higher concurrency  

---

## Key Takeaways  

- Benchmarks were run on **H100 (80 GB VRAM)** with **vLLM 0.10.2** in a Kubernetes environment  
- **TTFT and throughput vary significantly** depending on the model, even on identical hardware  
- Concurrency levels in benchmarks are **not directly equal** to number of end-users  
- Results provide a baseline for **evaluating candidate models for production usage**  

---

## Next Steps  

I will share benchmarks for **audio models** next week.  

In the meantime:  
- Have you measured similar metrics?  
- Do your results align, or differ, on other hardware / deployment setups?  

---

👉 *If you are working on LLM deployments and want to exchange notes on benchmarking methodology or results, feel free to reach out.*  
