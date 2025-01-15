---
layout: post
title: Expand ChatGPT knowledge with your own information
date: 2024-11-04 18:30:00 +0000
tags:
  - python
  - ai
  - llm
  - rag
---

## Table of Contents

1. [Why do we need to train an LLM with own data?](#why)
2. [Expanding LLMs current knowledge](#overview)
   - [Fine Tunning](#client-side)
   - [RAG](#server-side)
3. [What is RAG?](#frontend)
4. [Preparing Data for RAG](#backend)
5. [Embbeding models](#backend)
6. [Code Example]()
7. [Conclusion]()

---

## 1. Why must we train an LLM (ChatGPT, Claude, Ollama, etc) with our private data?

To train LLMs, engineers have compressed the whole of humanity's knowledge using public documents on the internet:

![LLM training](https://emmanuelorozco.com/assets/blog/rag/llm-training.png)

However, they have 2 big problems:

**1. Outdated information**

The last snapshot used to train ChatGPT is from October 2023 (now it's November 2024), which means we lost a few months of data while generating responses.

![last-snapshot](https://emmanuelorozco.com/assets/blog/rag/october-2023.png)

**2. Lack of specialized knowledge**

As mentioned, LLMs are trained with publicly available data. That means if you have private information (information on how to run a process, specific domain information, etc.), LLMs will not work as expected.

![Lack of knowledge description](https://emmanuelorozco.com/assets/blog/rag/emmanuel-orozco.png)

## 2. Expanding LLMs knowledge

So, if LLMs are outdated, what can we do about it? How can we update the information?

### 1. Fine Tunning

That means basically, re-train the whole model using new data (like reshaping a sculpture). We do this when we require the LLM to behave in a particular way (talk like trump, compose a new rap song).

![Fine Tunning description](https://emmanuelorozco.com/assets/blog/rag/fine-tune.png)

This is usually very expensive because it requires obtaining, cleaning, and processing all this new information. (requires computing and men's power).

### 2. RAG

When using RAG (**Retrieval augmented Generation**), we don't need to reshape the sculpture or re-train or model, we simply need to **provide more information** (also called context), to the LLM while making a question. Then the LLM will use this information to compute a more up-to-date answer.

![RAG Diagram](https://emmanuelorozco.com/assets/blog/rag/rag.png)

## Preparing Data for RAG

In a nutshell, to prepare our RAG we need to process our data in the following shape:

1. Gather the documents with the extra information
2. Convert this documents to plain text (pdf to text, image to text, etc)
3. Convert this text into the embedding model (explained in the next section)
4. Store the embedding data into a vector database

![Data Process](https://emmanuelorozco.com/assets/blog/rag/data-process.jpg)

After doing this, when making a question, we need to create an RAG module that will search (in the knowledge base), for the correct aditional information and add it as context to the prompt. (retriever).

![Data Process](https://emmanuelorozco.com/assets/blog/rag/rag-module.png)

## What is an embedding model and why do we need them?

We need to store our specialized information in a format that the LLM can understand. (LLMs can only understand text, not video or audio).

And we need to store this data efficiently.

Imagine you have 20,000 private PDFs of information about animal behavior (You work in a Zoo). ChatGPT does not know about this information and you want to ask GPT about a trend in this data.

Without an emmbeding model, you would need to store PDFs in database and when a question comes in to the the LLMs, look **one-by-one** document until you find an answer.

![Text embeddings](https://emmanuelorozco.com/assets/blog/rag/currently.jpg)

This takes a lot of time to compute, instead, we can use an embedding model and a vectorized database.

In an embedding model, we classify information **depending on how close is to each other**, using the DB records and the user question as reference.

![Text embeddings](https://emmanuelorozco.com/assets/blog/rag/text-emb.png)

In the example above, notice how the animals, ships or celestial bodies are together.

We use an array of numbers to represent this proximity.

![Numbers](https://emmanuelorozco.com/assets/blog/rag/numbers.png)

That way when we provide a word to this model (i.e. elephant), we don't need to look at all documents, the embed model will automatically return the documents that contain this word (i.e. elephant will return animal, forest, big, tree).

And finally we need a vectoraised database to store this embbeded model. (Chroma and Pinecone DB are famous Vector DB engines).

## Practice time

We would like to:

> Help sales reps generate better email responses to their clients based on top-performing interactions with clients.

This means, that when a question comes in, we need to search what similar answers have been given before by top-sales reps in the past and generate a similar answer.

So, we need to:

1. Prepare/load the data with the email interactions [download it here](https://emmanuelorozco.com/assets/blog/rag/customer_data.csv)
2. Create an embedding model (proximity vectors) with email interaction data.

And when a question comes in:

1. Do a similarity search
2. Send a request to ChatGPT with the additional context (user question + extra documents retrieved)
3. Get a response.

Here is the code step-by-step (is worth mentioning we are using [LangChain](https://python.langchain.com/docs/introduction/), (a framework to create AI apps).

```python
from langchain.document_loaders.csv_loader import CSVLoader
from langchain.vectorstores import FAISS
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.prompts import PromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.chains import LLMChain
from dotenv import load_dotenv

load_dotenv()

# 1. Load the database with the interactions
loader = CSVLoader(file_path="customer_data.csv")
documents = loader.load()

# 2. Vectorise the sales response csv data
embeddings = OpenAIEmbeddings()
db = FAISS.from_documents(documents, embeddings)

# 3. Function for similarity search
def retrieve_info(query):
 similar_response = db.similarity_search(query, k=3)
 page_contents_array = [doc.page_content for doc in similar_response]
    return page_contents_array


#4. Send request to OpenAI with the correct documents as additional context
llm = ChatOpenAI(temperature=0, model="gpt-4o")

template = """
You are an expert business development representative. I will provide you with a message from a prospect, and your task is to generate the most effective response.

This response should be informed by our previous successful interactions. Here is the message from the client:
{message}.

Below are examples of our best practices for responding to clients in similar situations:
{best_practice}
"""

prompt = PromptTemplate(
 input_variables=["message", "best_practice"],
 template=template
)

chain = LLMChain(llm=llm, prompt=prompt)


# 5. Run user query with the right context and get response
def generate_response(message):
 best_practice = retrieve_info(message)
 response = chain.run(message=message, best_practice=best_practice)
    return response

generate_response("What are the opening hours?")

```

## Conclusion

Sometimes we need to train LLMs to give better responses based on our private knowledge, using a technique like RAG will allow us to do it fast and cheap while keeping the quality of the responses.

Thank you! and until the next one!
