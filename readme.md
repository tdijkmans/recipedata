### Recipe data | Practicing data cleaning and basic analysis

#### What this repo contains

This repository contains a Jupyter Notebook file (ipynb format) and its content in markdown format. This is used to explore a list of 125k recipes (data not included).

#### Used technologies and concepts

- Jupyter notebook for python-based interactive data handling/analysis
- Pandas as a library to interact with dataframes
- regex for string-manipulation

#### Background

125k previously scraped recipes were available with ~20 relevant labels.
For an app focusing on recipes and ingredients, an attempt was made to identify most existing ingredients and possibly combinations from the label 'ingrediënten'. This list can e.g. serve for autocompletion features when users enter recipes, and reveal common ingredient combinations.
The source data appeared suboptimal: text contained many spelling errors and variations, additional text (e.g. quantities, sentences), leading to poor results. Reviewed recipes or food lists from other sources may deliver more usable results.
The data suggest that the number of ingredients used for most recipes is probably 1k-2k; this is a number feasible for (semi)manual collection/curation.

#### Practice goals

- to practice Jupyter & Python for data handling
- to practice regex
- to practice langage/text processing and analysis (removing stopwords, analyze 'bigrams' such as 'creme fraiche' or 'sambal oelek')
