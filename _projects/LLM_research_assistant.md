---
layout: distill
title: LLM Research Assistant
description: How to build a Research assistant with LLM
importance: 3
related_publications: false
bibliography: LLM_research_assistant.bib
---

In the last few years, LLMs were real game changers in a lot of domains.
However, LLMs are just models trained to predict the next single token, and this is not enough to build complex application.

The need of chaining the LLMs with traditional software stack to build larger applications gave rise to <b>Langchain</b>, one of the most fast-growing Open-Source projects of all time.

The simple idea of Langchain is to allow the interaction between models and other software (APIs,  databases, …) through a common interface.

One of the most powerful technologies (simply integrable with Langchain) is <b>Retrieval Augmented Generation</b> <d-cite key="lewis2020retrieval"></d-cite>.

RAG was proposed by Meta in NeurIPS 2020.
They combined LLM (parametric memories) with information retrieval (non-parametric memories) to increase and update the knowledge without additional training.
More specifically, they combine these components in a probabilistic model:
<ul>
    <li>The retriever provides latent documents conditioned on the input.</li>
    <li>The seq2seq model conditions on these latent documents together with the input to generate the output.</li>
</ul>
A typical RAG application has two main components:
<ul>
    <li><b>Indexing</b>: a pipeline for ingesting data from a source and indexing it. This usually happens offline.</li>
    <li><b>Retrieval and generation</b>: the actual RAG chain, which takes the user query at run time and retrieves the relevant data from the index, then passes that to the model.</li>
</ul>

The indexing is composed by several substeps in Langchain:
<ul>
    <li><b>Load</b>: First we need to load our data. This is done with <b>DocumentLoaders</b>.</li>
    <li><b>Split</b>: <b>Text splitters</b> break large <b>Documents</b> into smaller chunks. This is useful both for indexing data and for passing it into a model, since large chunks are harder to search over and won’t fit in a model’s finite context window.</li>
    <li><b>Store</b>: We need somewhere to store and index our splits, so that they can later be searched over. This is often done using a <b>VectorStore</b> and <b>Embeddings</b> model.</li>
</ul>

The problem faced in this project is to have a large bibliography with a huge number of papers, and they are the starting point to write a paper considering those topics.
It is hard for an LLM to help because the knowledge required is really specific, otherwise the writing style will not be suitable for a scientific paper.
LLMs suffers also of hallucination problems, so it is hard to understand if they are generating wrong sentences.
The idea is to consider to integrate RAG over these papers to have a chatbot able to help.

I started with the bibliography of my Master Thesis (40 papers) about WSOD in RSI.
I used to the LLM to answer a question about PCIR, one of the most relevant networks in the literature.
There is not additional experiment/question, since it was made just for assessment.

I used the Langchain framework, and most  of the tools in the indexing pipeline are the default ones:
<ul>
    <li>RecursiveTextSplitter as splitter</li>
    <li>Chroma as vectorstore</li>
    <li>OpenAIEmbeddings as embeddings</li>
    <li>I choosed Grobid as DocumentLoader</li>
</ul>

Grobid is a machine learning library for extracting, parsing, and re-structuring raw documents.
It is designed to be used with academic papers and it uses a cascade of sequence labelling models, that can be selected among ML architecture such as CRF and other DL models.
The models do not work on text but on Layout Tokens. 
Grobid could be used (and integrated into Langchain) to parse papers in PDF format into Documents.

The basic experiment (without RAG) is performed using this prompt:

```html
System: The user will provide one or more questions 
related to a scientific argument. 
You are a paper writing assistant that 
writes a paragraph that answers to the questions.

Notice that the paragraph will be 
part of a scientific paper, 
so the writing style must be technical and formal.


Human: What are the main components and the
key innovations of Progressive Contextual Instance Refinement (PCIR)?

```

The answer is inaccurate and partially hallucinating.

```html
Progressive Contextual Instance Refinement (PCIR) is a 
novel approach in the field of computer vision that 
aims to improve the accuracy of object detection and 
instance segmentation tasks. 

The main components of PCIR include a backbone network, 
a contextual module, and an instance refinement module. 

The backbone network is responsible for extracting 
high-level features from the input image. 

The contextual module utilizes contextual information 
to enhance the representation of objects and their surrounding context. 

This module incorporates a contextual attention mechanism 
that selectively focuses on relevant regions. 

The instance refinement module refines the initial object 
proposals by iteratively updating the bounding box coordinates 
and segmentation masks. 

It employs a progressive refinement strategy that 
gradually improves the accuracy of the detected instances. 

The key innovation of PCIR lies in its ability to 
effectively leverage contextual information and refine 
object instances in a progressive manner, 
leading to improved performance in object detection and 
instance segmentation tasks.
```

Applying RAG, the prompt became the following (with the context filled at run time):
 
```html
System: The user will provide one or more questions 
related to a scientific argument. 
You are a paper writing assistant that 
writes a paragraph that answers to the questions.

Notice that the paragraph will be 
part of a scientific paper, 
so the writing style must be technical and formal.

Use the following pieces of retrieved context 
to write a more complete paragraph:
Context: {context}

Human: What are the main components and the
key innovations of Progressive Contextual Instance Refinement (PCIR)?
```

In this case the result is much more precise and compatible with the writing style of a scientific paper.

```html
The main components of Progressive Contextual Instance Refinement 
(PCIR) are the DCIR module and the PPSP module. 

The DCIR module is responsible for mining all 
potential instances existing in the images, 
while the PPSP module removes noisy negative samples 
and maintains the diversity of the samples. 

These two modules work together to progressively 
refine the contextual instances. 

The key innovation of PCIR lies in its ability 
to effectively combine these two modules to 
achieve superior performance in weakly supervised object detection. 

Experiments conducted on the NWPU VHR-10.v2 and 
DIOR data sets have clearly demonstrated the 
effectiveness of PCIR compared to state-of-the-art 
methods under a weakly supervised paradigm.
```