---
layout: post
title: Expand ChatGPT knowledge with your own information
date: 2024-11-05 18:30:00 +0000
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

---

## 1. Why do we need to train an LLM (ChatGPT, Claude, Ollama, etc) with our own data?

To train LLMs, engineers have compressed the whole of humanity's knowledge using public documents on the internet:

![LLM training](https://emmanuelorozco.com/assets/blog/rag/llm-training.png)

However, they have 2 big problems:

1. Outdated information

Last snapshot used to train ChatGPT is from October 2023 (now it's November 2024) that means we have a few months lost of data while generating responses.

![last-snapshot](https://emmanuelorozco.com/assets/blog/rag/october-2023.png)

2. Lack of specialized knowledge

As mentioned, LLMs are trained with public available data, that means, if you have private information (information on how to run a process, specific domain information, etc) LLMs will not work as expected.

![Lack of knowledge description](https://emmanuelorozco.com/assets/blog/rag/emmanuel-orozco.png)

## 2. Expanding LLMs knowledge

So, if LLMs are out of date, what can we do about it? How can we update the information?

1. Fine tunning
   That means basically, re-train the whole model using new data (like reshaping a sculpture). We do this when we require the LLM to behave in a particular way (talk like trump, compose a new rap song).

![Fine Tunning description](https://emmanuelorozco.com/assets/blog/rag/fine-tune.png)

This is usually very expensive because it requires obtaining, cleaning, and processing all this new information. (requires computing and men's power).

2. RAG (**Retrieval augmented Generation**)
   When using RAG, we don't need to reshape the sculpture or re train or model, we simply need to **provide more information** (also called knowladge base embedding), while making a question to the LLM. Then the LLM will use this information to compute a more up-to-date-answer.

![RAG Diagram](https://emmanuelorozco.com/assets/blog/rag/fine-tune.png)

## Preparing Data for RAG

In a nutshell, to prepare our RAG we need to process our data:

1. Gather the documents with the extra information
2. Convert this documents to plain text (pdf to text, image to text, etc)
3. Convert this text into the embedding model (explained in the next section)
4. Store the embedding data into a vector database

![Data Process](https://emmanuelorozco.com/assets/blog/rag/data-process.jpg)

After doing this, we only need to provide the LLM with the right context based on the user's query.

![Data Process](https://emmanuelorozco.com/assets/blog/rag/rag-module.png)

## What is an embedding model and why do we need them?

We need to store our specialized information in a format that the LLM can understand. (LLMS can only understand text, not video or audio).

And we need to store this data efficiently.

Imagine you have 20,000 private PDFs of medical records about patients. ChatGPT does not know about this and you want to ask GPT about a trend in this data.

![Text embeddings](https://emmanuelorozco.com/assets/blog/rag/20k.jpg)

Without an emmbeding model, you would need to store PDFs in database and when a question comes in to the the LLMs, look **one-by-one** document until you find an answer.

![Text embeddings](https://emmanuelorozco.com/assets/blog/rag/currently.jpg)

This takes a lot of time to compute, instead, we can use an embedding model and a vectorized database.

In an emmbeding model, we classify information **depending how close is to each other**, using the DB records and the user question as reference.

![Text embeddings](https://emmanuelorozco.com/assets/blog/rag/text-emb.png)

We use an array of numbers to represent this proximity.

That way when we provide a word to this model (i.e diabetes), we don't need to look to all documents, the embed model will automatically return the documents that are close to this word (i.e diabetes will return diseases, insulin, sugar, etc)

And finally we need a vectoraised database to store this embbeded model. (Chroma and Pinecone DB are famous Vector DB engines).

## Practice time

Let's imagine wanting to help sales generate better email responses to their clients based on top-performing interactions with clients.

So, we need to:

1. Prepare/load the data with the email interactions [here](sales_data.csv)
2. Vectorised this data with an embedding model
3. Do a similarity search (for best practices) when a question comes in
4. Send a request to ChatGPT with the additional context
5. Get a response.

Here is the code step-by-step (is worth mentioning we are using LangChain, to create the responses)

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
loader = CSVLoader(file_path="sales_response.csv")
documents = loader.load()

# 2. Vectorise the sales response csv data
embeddings = OpenAIEmbeddings()
db = FAISS.from_documents(documents, embeddings)

# 3. Function for similarity search
def retrieve_info(query):
 similar_response = db.similarity_search(query, k=3)

 page_contents_array = [doc.page_content for doc in similar_response]

    # print(page_contents_array)

    return page_contents_array


#4. Send request to OpenAI with the correct documents as additional context
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo-16k-0613")

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
