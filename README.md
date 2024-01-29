# Job Market Analysis Scraper

This repository contains a web scraper built using Puppeteer to extract job listings data from various Spanish and international employment portals. 

The goal of this project is to perform an analysis of the geographical distribution of jobs in the field of development, focusing on roles such as front end, back end, and specific technologies or frameworks.

Additionally, we aim to determine which frameworks are most in-demand in A Coruña and remote positions. The extracted data will be analyzed with the help of Natural Language Processing models provided by Hugging Face's Transformers library (server through a simple Flask API), specifically QA & Zero-Shot Classification models. These models will assist in categorizing job postings based on factors like technology stack, experience level (junior vs senior), and location (remote or A Coruña).

For more complex information that cannot be obtained through simple CSS selectors or basic NLP techniques, we utilize DeepInfra's Function Calling API over Mistral 7B Instruct. This powerful tool allows us to extract deeper insights about each job posting, including detailed requirements and qualifications.
