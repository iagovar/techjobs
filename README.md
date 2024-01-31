# Job Market Scraper &  Analysis

As a junior developer searching for employment opportunities, I have created this repository to host a web scraper capable of extracting valuable job listing data from popular Spanish and international employment portals. Utilizing Puppeteer, I am able to gather essential details about available positions, allowing me to analyze the demand for different technologies and frameworks in my area (A Coruña) and remote locations.

As a junior developer looking for employment, you'll get tons of conflicting information about what should you focus on. With so much noise, I decided to gather an extract information from popular spanish and international job listings, so I can have more actionable information about positions in my area (A Coruña) and remote ones.

![FlowChart](./docs/job-listings-scraper.svg)

## Data sources:

- Infojobs
- Tecnoempleo
- LinkedIn
- Manfred
- Eures
- Indeed

## Stack:

- **Puppeteer** for most of the heavy lifting. It provides excelent tools to control a Chromium instance.
- **Flask** for serving a local instance of QA and Zero-shot classification from [HuggingFace](https://huggingface.co/models).
- **Mistral 7B Instruct** for some complex data extraction that can't be performed with simple models. As runing LLMs locally in a timely manner is very expensive right now, I'm going to outsource it to DeepInfra as it's the cheaper provider offering [Function Calling](https://deepinfra.com/docs/advanced/function_calling) with this model.
- **DuckDB** for storing and analytics.

### Why using this stack?

**Puppeteer** is in my opinion the best library for scraping, ahead of selenium and other typical ones available in python. I'm also familiar with async/await in JavaScript, so using Puppeteer is a no brainer.

[In a former project](https://github.com/iagovar/cometocoruna) I used SQLite instead of **DuckDB** because I was encountering many compatibility issues with Alpine Linux, DBeaver, etc. I didn't want to get stuck with it, and the project wasn't focused on analytics like this one. But here an OLAP DB Makes sense, and my SQL skills faded out since I had training on Postgre, so this is the perfect opportunity.

I also tried to use **BERT-family models** under JS using *transformers.js*, but found that converting models to *ONNX* worked on the surface, but when I wanted to analyze big chunks of texts I've got all kinds of bugs (different bugs depending on the model) while it worked perfectly fine using Python + *transformers* original library.

Since *ONNX* is it's own C++ beast and ML is a pretty deep rabbit hole, I decided to cut my losses and just serve this models over an API using **Flask** and be done with it. It's not the most elegant solution, but Flask and HuggingFace Transformers library make it so easy that it's difficult to say no.

Finally, if we want to extract info from the data we'll get, and want to avoid the whole deal of training or finetuning models, it's easier to just use *function-calling* over some LLM. This feature was introduced by OpenAI but now other providers offer the same with Open Source models, chaper. That's basically the reasons for choosing **Mistral 7B Instruct** over DeepInfra.


## Data Fields

So we need to define what fields do we need to store in order to define the database schema and what Puppeteer needs to look for.

Most fields depending on the data returned by Puppeteer should be able to go `null` until we run the data extraction pipeline. In my experience, even when on initial testing it looks like your CSS Selectors work, it is common to run into unexpected problems, layout changes, etc.

So if I don't want to sink too much time into testing the scrapers, I have to make the following trade-off: I'll store the whole HTML page in the DB (Increasing size significantly), trying to fill as many fields possible from Puppeteer and filling the gaps with data extraction later.

So all fields exept PK, date scraped, html and text content can be null.

We also will have two tables. One table called `scraped` where we dump all raw data (so pretty much everything will be text) and another called `extracted` where only the fields curated after our data extraction pipeline land.

This is because running analytical queries against dirty data is not a good Idea, and we want to separate this two concerns.

### Table Schema

+---------------------+------------+-------+--------+
| Column Name         | DataType   | Null? | Others |
+---------------------+------------+-------+--------+
| url                 | varchar    | no    | PK     |
+---------------------+------------+-------+--------+
| dateScrapedISO      | timestampz | no    |        |
+---------------------+------------+-------+--------+
| datePublishedISO    | timestampz | yes   |        |
+---------------------+------------+-------+--------+
| roleCompanyName     | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleTitle           | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleDescription     | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleLocation        | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleIsFullRemote    | boolean    | yes   |        |
+---------------------+------------+-------+--------+
| roleOpenPositions   | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleNumApplicants   | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleTechList        | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleSalaryRange     | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleMinExperience   | varchar    | yes   |        |
+---------------------+------------+-------+--------+
| roleIsJunior        | boolean    | yes   |        |
+---------------------+------------+-------+--------+
| roleBootCampAllowed | boolean    | yes   |        |
+---------------------+------------+-------+--------+
| roleCompanyName     | varchar    | yes   |        |
+---------------------+------------+-------+--------+
