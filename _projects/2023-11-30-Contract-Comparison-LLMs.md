---
title:  LexLingua - Bridging Legal Worlds with AI Contract Comparison
subtitle: Empowering Legal Analysis Through Cutting-Edge Language Models
date: 2023-11-30 00:00:00
description: Merging human expertise with advanced language models to empower legal analysis, delivering succinct summaries for fast decision-making
featured_image: law_books.jpg
accent_color: '#996515'
gallery_images:
  - law_books.jpg
---

<img src="/images/ai_lawyer.jpeg" alt="ai_lawyer" width="500">

All the source code for this project can be found [on my github](https://github.com/jmoro0408/Terms-Conditions-Comparison).


# Terms-Conditions-Comparison
## Background

This project utilises Large Language Models (LLMs) to summarise differences between two versions (2015 and 2023) of Apple's Terms and Conditions. 
The project is split into 3 parts:
1. Summarisation and comparison of the T&Cs.
2. Exploration of metrics applicable to quantify the performance of the summary.
3. Investigatin of alternative methods and models for the task.

## Motivation
Legal contracts are often extensively long document that require hours of tedious labour to parse and understand. Automatic summarisation of these documents can provide human experts with quick insights and reduce the time and effort required to understand them. 

Further, by providing a summary comparison of two contract documents the time and effort savings are doubled. 

The recent(ish) improvements in Large Language Models such as GPT4, Bard, and Claude have made this project possible. I also attempt summarisation with more traditional natural language processing approaches (see `Traditional Methods` below), however these performed extremely poorly in comparison to these new large models.  

## Summarisation 
I used two versions of Apple's Terms & Conditions as these are free and publically available. The first was from 2015, and the second from 2023. 

The 2015 version contained ~15,000 words (approximately 18,000 GPT-4 tokens), and the 2023 version approximately ~8000 words (10,000 GPT-4 tokens). This means that both versions exceeds the GPT-4 token limit of 8,192 (although could be stuffed into the GPT-4-32k model).

I utilised [langchain](https://python.langchain.com/docs/get_started/introduction) to work with the models in a programatic fashion, allowing me techniques to circumvent the token limit and create reproducible code. 

Langchain provides various methods for passing large documents to language models: 
1. Stuff
2. Map-Reduce
3. Map-Rerank
4. Refine

I tried both stuff and map-reduce. In future I would like to attempt using Map-Rerank and Refine and comparing the results.  

### Stuff

I attempted using the [stuff](https://python.langchain.com/docs/modules/chains/document/stuff) method, however this is better suited for smaller documents and didn't yield useful results (it essentially just said "these are Apple's T&Cs" - a little *too* summarised).

### Map reduce

I saw much better results with the [map-reduce](https://python.langchain.com/docs/modules/chains/document/map_reduce) method. Here, the LLM is supplied with a chunk of the document interatively and asked to summarise the chunk - `map`. The summaries are then combined and re-summarised in the `reduce` section. 

<img src="https://python.langchain.com/assets/images/map_reduce-aa3ba13ab16536d9f9e046276bd83dd2.jpg" alt="map-reduce" width="500">


Here's an example of the output for the 2015 Apple T&Cs: 

```
- iTunes is not responsible for lost or stolen gift certificates, iTunes Cards, and Codes, and any unavailable previously purchased content.
- Users can auto-download content on up to 10 devices, but no more than five can be iTunes-authorized computers.
- iTunes Match, a subscription service, allows users to access their songs and music videos from any associated device, up to 25,000 songs.
- The Genius feature collects users' media libraries information for personalized recommendations.
```

The full output comprised of 15 bullet points. 


### Map Reduce over Sections

Summarisation with map-reduce seemed to work well, however I always try to think how I would approach any ML problem from a human/manual perspective. In this regard, I would probably try to split up each contract into meaningful sections and summarise one by one.

Therefore I recreated this approach in code by manually delineating various sections in the text and asking the LLM to create summarise of these sections. 

This seemed to provide more useful outputs and capture the scope of the entire document more precisely. 

Here's an extract from the 2015 "Sections" version:

```
- Users must be at least 13 years old to create accounts and the stores are only available in the UK.
- Users are responsible for maintaining the confidentiality of their account information.
- iTunes is responsible for maintenance and support services for Apple Products only.
- Products are licensed, not sold, to users.
```

A lage improvement between the automatic map-reduce and manually sectioning the document is the sections version consisted of 38 bullets, rather than 15. Although providing more imformation in a summary may not be the clear goal, I think the 38 bullets managed to capture more of the useful information whilst maintaining readability.


### Vectors
This method was inspired by a [blog post](https://pashpashpash.substack.com/p/tackling-the-challenge-of-document). 
Here, the documents are split up and converted to word embeddings. A clustering algorithm then attempts to bucket pieces of text that are 'closer' to one another. After this, you can pull out the central point for each cluster and feed these into an LLM to produce your summary. 

The thinking here is that the clustering identifies the "key topics" within the document and re-assembles them onto a context-rich summary. 

This is essentially a automated way of creating document sections like above. 

I attempted this with mixed results and it looks like it shows a lot of promise however I need more time to play around with the various chunk sizes and number of clusters. 

A key benefit of this approach of map-reduce is that the text is only fed into the LLM once, providing cost savings, whereas map-reduce can produce several calls to the LLM, adding up in price. 


## Summary Metrics
As I repeated the summarisation using various models:
* GPT4 - OpenAI
* Davinci - OpenAI
* Bard - Google

I needed a way to objectively determine the performance of the summary. 

Understanding how "good" a summary is without reading the text that is being summarised is notoriously hard. 

Industry standard metrics for determinig the quality of text summaries include [ROUGE](https://en.wikipedia.org/wiki/ROUGE_(metric)), [BLEU](https://en.wikipedia.org/wiki/BLEU) or a combination of both.

However these metrics have downfalls as they both use syntax to calculate their scores, rather than semantic meaning. 
These metrics may have worked well for traditional summarisation methods, where summaries are essentially just extracts of the original text, but with Generative AI new text is created which may have the same contextual meaning however be completely different syntactivcally. 

For example the two sentences below have the same meaning, but score only 0.5 (out of 0 - 1) due to their differences in wording:
1. "The dog slept on the mat", 
2. "The canine snoozed on the rug"


A better metric may use word embeddings and cosine similarity to determine a score, as embeddings manage to capture contextual meaning over syntactic similarity. 

In order to undertake this I first load in the full T&Cs for both 2015 and 2023 and create word embeddings,storing these in a ChromaDB vector database. 

I then loop through each version of the summary created in the previous section and create embeddings for each. 
Finally I calculate cosine distance scores for each summary against the entire text, with a lower score indicating in a more similar text and a better summary. 

Final results are below:

| Model           | 2015         | 2023         |
| --------------- | ------------ | ------------ |
| sections (GPT4)       | 0.1315       | 0.1009       |
| map-reduce (bart)            | 0.3278       | 0.1500       |
| map-reduce (davinci)         | 0.1497       | 0.1691       |
| map-reducen (gpt4)             | 0.1340       | 0.1307       |
| vectors         | 0.2230       | 0.1967       |

As expected from human review, the model where sections are manually delineated produces the best summary. 

## Summary Comparison
With summaries of both the 2015 and 2023 contracts produced, I then utilised langchains document comparison [agent](https://python.langchain.com/docs/integrations/toolkits/document_comparison_toolkit) to compare the two summaries andf highlight any key differences:

``` 
Here are some key differences between the 2015-TandCS and 2023-TandCS:

1. Services Offered: The 2015-TandCS focus on the terms and conditions for purchasing and accessing digital content through the iTunes Store Service, while the 2023-TandCS mention various services offered by Apple, such as the App Store, Apple Music, Apple TV, Apple Books, Apple Podcasts, and Apple Fitness+.

2. Payment and Transactions: The 2015-TandCS include details about purchasing products, cancellations, refunds, and Family Sharing, while the 2023-TandCS state that Apple will charge the user's selected payment method for any paid transactions. It also covers payment, taxes, and refunds related to the services and content.

3. User Responsibilities: The 2023-TandCS mention that users are responsible for maintaining the confidentiality and security of their Apple ID. They must also comply with the Submission Guidelines when submitting materials to Apple's services.

4. Licensing and In-App Purchases: The 2015-TandCS emphasize the licensing of products to users, while the 2023-TandCS state that apps are licensed, not sold, to users by Apple or third-party developers. Apps may offer in-app purchases, with Apple or the app provider responsible for their maintenance and support.

5. Termination and Suspension: The 2023-TandCS mention that Apple can terminate or suspend services if users fail to comply with the agreement. Users may also face data charges incurred in connection with the use of the services.

6. Governing Laws: The 2015-TandCS do not specify the governing laws, while the 2023-TandCS state that the agreement is governed by the laws of the State of California, unless the user is not a U.S. citizen or resides outside the U.S. Users must also comply with all local, state, federal, and national laws that apply to their use of the services.

7. Notifications and Updates: The 2023-TandCS mention that Apple may notify users about the services via email, postal mail, or by posting on the services. Apple also reserves the right to modify the agreement and add new terms or conditions for the use of its services.

Please note that these are just some of the key differences based on the provided context, and there may be additional details and distinctions within the agreements.

```

I also tried prompt engineering by submitting the two summaries into a chat space and asking the model to provide differences "in the style of a contract lawyer". However the document comparison chain seemed to provide better results. 


## Alternative models and methods

I attempted to use some open source models to create summaries, in order to compare to the OpenAI models. Initially I tried to use a model that was a version of BERT fine tuned on legal text (https://huggingface.co/nlpaueb/legal-bert-base-uncased) however this didn't work with HuggingFaceHub, and I don't have enough compute power to run it locally. 

I also tried a BART model from facebook, fine tuned on CNN news (https://huggingface.co/facebook/bart-large-cnn). I got this working to produce a summary but it had poor performance on longer text (like the input contracts).

### Traditional Methods
Before large language models, there were a number of traditional methods capable of summarisation. I implemented a few of this with the [sumy](https://miso-belica.github.io/sumy/) package with mixed results..


1. Lexrank - Worked okay and seemed to capture some key points, however not as performant as GPT

2. LSA - Seems to pull out random text without much context

3. Luhn - Seems to pull out random text without any context

4. KL-Sum - Performed worse than Luhn 

Although I didn't spend much time tuning these methods, they don't show a lot of promise over the use of LLMs. 
