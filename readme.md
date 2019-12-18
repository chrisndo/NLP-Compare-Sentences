# ![Deloitte](img/deloitte.png)
# <center>ITS Cyber Security</center>


This notebook is used to show the things I've learned during my analysis of comparing the US Cloud Security Standards with the Global Cloud Security Standards.  The original problem involved comparing the two different standards to determine if there were any discrepancies between the two.  The US Cloud Security Standards are still being finalized, and were not written by the same group that wrote the Global Cloud Security Standards, so naturally there are going to be a lot of differences, also taking into account the fact that the US Cloud Security Standards may have additional requirements due to regulatory reasons that must be followed within the US.

By: Chris Do


The Global Cloud Security Standards were created and formatted into a table within an Excel spreadsheet listed like the image below.

### Global Cloud Security Standards
![Global Cloud Security Standards](img/global.png)

<br/><br/>

The US Cloud Security Standards are still being finalized, but it was created in a standard paragraph format, like shown below.

### US Cloud Security Standards
![US Cloud Security Standards](img/us.png)


### Problem #1:
As you can see, these documents are not formatted similarly, and although they do not serve the exact same purpose, they do have similar content, which is a list of standards that should be followed in order to ensure that the Cloud Security of Deloitte is up to the regulations and policies.

### Solution #1
In order to best compare the two different types of documents, one being a table in Excel, and one being a word document containing sentences not in a table, I decided to parse the Word document to create an Excel spreadsheet similar to the format of the Global Cloud Security Standards.  

In order to do this, I parsed the entire word document and for every "Heading 2, 3, or 4", I added a new row, and then for every "Normal" text, I added in a new column.  This would create a table with four columns, being **[Heading 2, Heading 3, Heading 4, Normal]**

The script for this conversion is shown below.



```python
import docx
import pandas as pd
from openpyxl import Workbook, load_workbook
from datetime import datetime

filename = 'css.docx'
destPath = 'Results/results-'+datetime.now().strftime("%Y%m%d-%H%M%S%p")+'.xlsx'

# ---------- Create a file to save results ----------------------#
# Create Workbook & Sheet
destFile = Workbook()
destFile.save(destPath)

data = []
h2=h3=h4=''

doc = docx.Document(filename)
for para in doc.paragraphs:
    if para.style.name == 'Heading 2':
        h2 = para.text
        h3=h4=''
    elif para.style.name == 'Heading 3':
        h3 = para.text
        h4=''
    elif para.style.name == 'Heading 4':
        h4 = para.text
    elif h2 != '' and para.text != '\n' and (para.style.name == 'Normal' or para.style.name == 'List Paragraph'):
        data.append([h2, h3, h4, para.text])
    else:
        continue

df = pd.DataFrame(data, columns=['Domain', 'Sub Domain', 'Control Type', 'Control'])
# Load
book = load_workbook(destPath)
writer = pd.ExcelWriter(destPath, engine='openpyxl')
writer.book = book
# Write
df.to_excel(writer, sheet_name='data', index=None)
book.remove(book['Sheet'])
# Save
writer.save()
writer.close()
```

After running the script, an excel file is generated, which is in the format of the image below.  As you can see, this will make it a lot easier to compare the Global Standards with the US Standards.  The image below is only a small portion of the US Cloud Security Standards that corresponds with the screenshot of the original Word Doc shown above.

![US Cloud Security Standards](img/usconverted.png)

### Problem #2:  
Now that we have the separate files into a format that will make it easier to compare them, the next problem is that the US and Global standards were not written by the same team, and as mentioned earlier, may have discrepancies or differences due to different regulations in the US. 

In order to do a quick match, you can compare the **Control** column from the US table with the **Requirement description** from the Global table, but you will quickly see that it won't be easy to find the matching standards.  If you do a standard VLOOKUP to look for matches, you will find that any space, period, typo, comma, etc will change the string and they will not produce a match.  

In the examples below, you can see that both A and B are conveying the same  control, but they will never show as "equal" in a comparison programmatically. 


```python
A = 'This is a control about cloud security'
B = 'This is a control about cloud security.'
A==B
```




    False




```python
A = 'This is a control about cloud security'
B = 'This is a control about Cloud Security'
A==B
```




    False




```python
A = 'This is a control about cloud security'
B = 'This is a control  about cloud security'
A==B
```




    False



You will also find that some of the standards are written similarly, but are nowhere near identical.  There are some examples below.

```
A='Monitor remote connections, detect aberrant behavior and notify appropriate personnel.'
B='Aberrant behavior on remote connections must be detected. Appropriate personnel must be notified as appropriate.'
C='Temporary access to cloud environment resources shall be restricted, monitored, and revoked within 24 hours of expiration.'
D='Temporary and or conditional access to cloud environment resources must be restricted based on least privileged, monitored, and must be revoked within organizational defined period of expiration'
```
It is clear after reading the controls that A and B are conveying the same control, and C and D are conveying the same control, but if I were to do a simple equals comparison programmatically, they would never be found as a match.  





This is where **Natural Language Processing** comes into play.

Natural language processing (NLP) is a subfield of linguistics, computer science, information engineering, and artificial intelligence concerned with the interactions between computers and human (natural) languages, in particular how to program computers to process and analyze large amounts of natural language data. (https://en.wikipedia.org/wiki/Natural_language_processing)

NLP is a very popular area amongst data scientists, and it's how systems like Google Search, Alexa and Siri, targeted Ads, etc are able to communicate and understand what we as humans are trying to say.  However, the process of understanding and manipulating human language is complex, so there are many different methods available by very smart people for different ways to perform NLP.  A very, very quick high-level overview of the fundamentals of NLP is discussed below.  (All of the summaries are from https://towardsdatascience.com/your-guide-to-natural-language-processing-nlp-48ea2511f6e1)

#### Bag of Words
This is a commonly used model that counts words in a document or string and creates a matrix that contains the frequencies of each word in the document.  The downside to this is it includes common "stop words" like `them` or `an` which add noise to the analysis.  The way to solve solve this is with a scoring approach called **Term Frequency -- Inverse Document Frequency (TF-IDF)** which improves the weights by scoring words higher if they are frequently used, but begin to get scored lowered if those terms are frequent in other documents in the algorithm.  Essentially, this method rewards unique terms when considering the all of the documents.

#### Tokenization
This is the process of separating text/documents into sentences or words, called tokens.  This also will throw away punctuation and allow each token to be used for analysis when looking for matches.

#### Stop Words
This is the process of removing common words, like `and`, `the`, and `to`.  This will help to eliminate some of the noise that is produced when performing analysis on text.

#### Stemming
This is the process of shortening words to to become just the root of the word.  Prefixes and suffixes are removed from words, like `astrobiology` is shortened to `biology`, and `working` becomes `work`.  These affixes are almost always deritives of the root word, and if we use only root words in our analysis, it makes it easier to do our analysis with less noise.

#### Lemmatization
This is a process very similar to stemming, in which the goal is to reduce a word to its base form.  Lemmatization goes a step further, and attempts to group together different forms of the same word.  Verbs in past tense are changed to present tense, e.g. `went` is changed to `go`, and synonyms are unified, e.g. `best` is changed to `good`, which standardizes words.  Lemmatization can also take into consideration the context of the word, such as the word `bat`, which can be a species, or a tool used to hit a baseball.  This is done by using a part of speech paramater with the word to define a role for that word in the sentence and help remove disambiguation.

Lemmatization may be more powerful and effective than stemming, but it is also much more resource intensive, which is why stemming is the typical approach when speed/efficiency is prioritized over accuracy.


### Solution #2: ###
Using NLP and Python, I will use various libraries and models to perform analysis on the two different spreadsheets.  I will discuss each library and the type of analysis it does, as well as showing the code used and the output.  

The code below is just importing libraries and defining functions that will be used later.  It also defines some variables, such as the input spreadsheet, the columns that will be used for analysis, and creates an Excel sheet to output the results to.


```python
import pandas as pd
import numpy as np
from openpyxl import Workbook, load_workbook
from datetime import datetime
from difflib import SequenceMatcher
from fuzzywuzzy import fuzz, process
import jellyfish as jf
import nltk, string
from nltk.corpus import stopwords
from nltk.corpus import wordnet as wn
from nltk.stem import WordNetLemmatizer
from sklearn.feature_extraction.text import TfidfVectorizer
import ssl
import sys, argparse
import warnings
import time
import spacy
import wmd


#################### Functions & global vars ####################
remove_punctuation_map = dict((ord(char), None) for char in string.punctuation)

#SequenceMatcher
def similar(a, b):
    return SequenceMatcher(None, a, b).ratio()

#begin NLTK functions
def stem_tokens(tokens): #for NLTK Cosine Analysis
    return [stemmer.stem(item) for item in tokens]

def lemma_tokens(tokens): #for use with lemma normalization (not being used, but may be better)
    return [wnl.lemmatize(item) for item in tokens]

def stem_and_synonym(tokens):
    stemmed=[]
    orig_items = []
    for item in tokens:
        orig_items.append(item)
        stemmed.extend(get_synonyms(item))
        stemmed.append(stemmer.stem(item))
    return stemmed

def get_synonyms(word):
    synonyms = []
    for syns in wn.synsets(word):
        for item in syns.lemma_names():
            if item.lower() not in synonyms:
                synonyms.append(item.lower())
    return synonyms

#remove punctuation, lowercase, stem
def normalize(text): #for NLTK Cosine Analysis
    return stem_tokens(nltk.word_tokenize(text.lower().translate(remove_punctuation_map)))

def normalize_lemma(text): #for use with lemma normalization (not being used, but may be better)
    return lemma_tokens(nltk.word_tokenize(text.lower().translate(remove_punctuation_map)))

def normalize_syn(text): #for use with lemma normalization (not being used, but may be better)
    return stem_and_synonym(nltk.word_tokenize(text.lower().translate(remove_punctuation_map)))

def cosine_sim(text1, text2): #for NLTK Cosine Analysis
    tfidf = vectorizer.fit_transform([text1, text2]) #use vectorizer instead of vectorizer2 for nltk stopword list
    return ((tfidf * tfidf.T).A)[0,1]

def cosine_sim_syn(text1, text2): #for NLTK Cosine Analysis
    tfidf = vectorizer_syn.fit_transform([text1, text2]) #use vectorizer instead of vectorizer2 for nltk stopword list
    return ((tfidf * tfidf.T).A)[0,1]

file1 = 'input.xlsx'
file2 = 'input.xlsx'
sheet1 = 'US'
sheet2 = 'Global'
column1 = 'Control'
column2 = 'Requirement description'

#################### Initialize workbook ####################
#file = 'input.xlsx'
destPath = 'Results/results-'+datetime.now().strftime("%Y%m%d-%H%M%S%p")+'.xlsx'
# Create Workbook & Sheet
destFile = Workbook()
destFile.save(destPath)
book = load_workbook(destPath)
writer = pd.ExcelWriter(destPath, engine='openpyxl')
writer.book = book
book.remove(book['Sheet'])
excel_file1 = pd.ExcelFile(file1)
excel_file2 = pd.ExcelFile(file2)
# Convert to Dataframe
df_f1 = pd.read_excel(excel_file1, sheet_name=sheet1, index_col=None, na_values=['NA'])
df_f2 = pd.read_excel(excel_file2, sheet_name=sheet2, index_col=None, na_values=['NA'])
df_combined = pd.DataFrame(df_f1[column1])
```

### SequenceMatcher Library
The first library we will use for our analysis is the SequenceMatcher module.  The Sequence Matcher module comes from the difflib library in Python.  This module compares pairs of sequences, in our cause, it will be strings of text.  The result will be a score/ratio of similarity.  The code below loops through every US Control, and compares it to every Global Control.  It uses the SequenceMatcher comparison function to assign a ratio to each US Control and Global Control, and it will take the best score for every row and assign a "match".

https://kite.com/python/docs/difflib.SequenceMatcher


```python

#################### SequenceMatcher ####################
compare_time = time.time()
highest = 0
similarity = 0
match = ''

for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        similarity = similar(search1, search2)
        if (highest < similarity):
            highest = similarity
            match = search2
    df_f1.at[i,'Match Percentage'] = highest
    df_f1.at[i,'Closest Match'] = match
    highest = 0
    match=''

df_combined['SequenceMatcher Percentage'] = df_f1['Match Percentage']
df_combined['SequenceMatcher Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='Sequence Matcher', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
print('finished Sequence Matcher Analysis - %s seconds' % (time.time() - compare_time))

```

    finished Sequence Matcher Analysis - 43.39950680732727 seconds



```python
df_combined
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>SequenceMatcher Percentage</th>
      <th>SequenceMatcher Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A formal cloud security governance program whi...</td>
      <td>0.365049</td>
      <td>Security baselines configurations for all syst...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The IT Risk Committee must meet every other mo...</td>
      <td>0.320786</td>
      <td>Intrusion Detection and Intrusion Prevention s...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cloud security must be integrated into Deloitt...</td>
      <td>0.407295</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A cloud security operating model shall be deve...</td>
      <td>0.387931</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cloud security tooling and automation shall be...</td>
      <td>0.430108</td>
      <td>Concurrent user-sessions must be restricted/li...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Perimeter security controls to be implemented</td>
      <td>0.480000</td>
      <td>Role Based Access Controls (RBAC) must be impl...</td>
    </tr>
    <tr>
      <th>361</th>
      <td>Environmental controls, such as generators, Un...</td>
      <td>0.367816</td>
      <td>Unauthorized and unintentional transfer of inf...</td>
    </tr>
    <tr>
      <th>362</th>
      <td>Supporting utilities to have controls in place...</td>
      <td>0.375796</td>
      <td>Firewalls are in place to prevent unauthorized...</td>
    </tr>
    <tr>
      <th>363</th>
      <td>Equipment maintenance to be performed to ensur...</td>
      <td>0.418972</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>364</th>
      <td>In cases where a cloud service contains regula...</td>
      <td>0.388298</td>
      <td>Password complexity and expiration policies mu...</td>
    </tr>
  </tbody>
</table>
<p>365 rows × 3 columns</p>
</div>



As you can see, there is a score (Column 2) for every US control (Column 1), and a corresponding "match" (Column 3) that is found from the Global Controls.  


```python
df_combined[(df_combined['SequenceMatcher Percentage'] > 0.5)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>SequenceMatcher Percentage</th>
      <th>SequenceMatcher Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>74</th>
      <td>On an annual basis, a review shall be performe...</td>
      <td>0.512821</td>
      <td>An annual review of all users with privileged ...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>0.702532</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>IAM solutions for cloud environments must be c...</td>
      <td>0.574545</td>
      <td>User credentials must be stored using cryptogr...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>An inventory of key cloud information assets s...</td>
      <td>0.647687</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Assets associated with cloud and cloud process...</td>
      <td>0.522613</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>Cloud-hosted application owners must identify ...</td>
      <td>0.527363</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Owners of cloud-based applications must ensure...</td>
      <td>0.778210</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.719626</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>151</th>
      <td>A data lifecycle for cloud-based data shall be...</td>
      <td>0.515152</td>
      <td>Events that include untrusted data shall not b...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.881159</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.945312</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.603352</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.967742</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>170</th>
      <td>For component engagements, data must reside in...</td>
      <td>0.605769</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>171</th>
      <td>Engagement metadata (e.g. engagement name, tea...</td>
      <td>0.503268</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>175</th>
      <td>All engagement or client data stored within cl...</td>
      <td>0.555556</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.977064</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.886700</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>221</th>
      <td>Intrusion Prevention Systems (IPS) are in plac...</td>
      <td>0.544910</td>
      <td>Intrusion Detection and Intrusion Prevention s...</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Support automatic intrusion detection, alertin...</td>
      <td>0.827160</td>
      <td>Automatic intrusion detection, alerting and re...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>0.829630</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Monitor remote connections, detect aberrant be...</td>
      <td>0.505051</td>
      <td>Aberrant behavior on remote connections must b...</td>
    </tr>
    <tr>
      <th>237</th>
      <td>Native IP filtering must be enabled for PaaS s...</td>
      <td>0.526316</td>
      <td>Auditing must be enabled for all applications/...</td>
    </tr>
    <tr>
      <th>239</th>
      <td>Production cloud-hosted applications and envir...</td>
      <td>0.506250</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Application code and endpoints for cloud-hoste...</td>
      <td>0.519824</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>245</th>
      <td>Applications have a Deloitte custom domain nam...</td>
      <td>0.625592</td>
      <td>All Deloitte hosted applications MUST have a D...</td>
    </tr>
    <tr>
      <th>248</th>
      <td>Source code for cloud-hosted applications shal...</td>
      <td>0.541985</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.901961</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>256</th>
      <td>A formal change request process shall be estab...</td>
      <td>0.544379</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>268</th>
      <td>All privileged activities on cloud instances /...</td>
      <td>0.506667</td>
      <td>All administrative activities must be logged f...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.815068</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>A Business Impact Assessment (BIA) must be per...</td>
      <td>0.542453</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>331</th>
      <td>A cloud service provider must agree that the D...</td>
      <td>0.524444</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Risks related to cloud service providers must ...</td>
      <td>0.655340</td>
      <td>Risks related to third parties must be identif...</td>
    </tr>
  </tbody>
</table>
</div>



After doing some manual comparisons, I was able to determine that a good score for an accurate match was typically any result above `0.5`, which is shown above.  As you can see, most of the controls above do correlate/match with each other.


```python
len(df_combined[(df_combined['SequenceMatcher Percentage'] > 0.5)])
```




    35



Just showing that out of 365 US Standards, only about 35 are a potential match.

### FuzzyWuzzy Library
This library is used to compare string similarity using the Levenshtein Distance.

Levenshtein distance is a string metric for measuring the difference between two sequences. Informally, the Levenshtein distance between two words is the minimum number of single-character edits (insertions, deletions or substitutions) required to change one word into the other. It is named after the Soviet mathematician Vladimir Levenshtein, who considered this distance in 1965.  The code below performs uses the same process as above, but using the FuzzyWuzzy functions to score each result.

https://github.com/seatgeek/fuzzywuzzy <br>
https://en.wikipedia.org/wiki/Levenshtein_distance


```python
#################### fuzzywuzzy ####################
compare_time = time.time()
highest = 0
similarity = 0
match = ''

for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        similarity = fuzz.token_set_ratio(search1, search2)
        if (highest < similarity):
            highest = similarity
            match = search2
    df_f1.at[i,'Match Percentage'] = highest
    df_f1.at[i,'Closest Match'] = match
    highest = 0
    match=''

df_combined['FuzzyWuzzy Percentage'] = df_f1['Match Percentage']
df_combined['FuzzyWuzzy Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='FuzzyWuzzy', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
print('finished FuzzyWuzzy Analysis - %s seconds' % (time.time() - compare_time))
```

    finished FuzzyWuzzy Analysis - 5.0954015254974365 seconds



```python
fuzzywuzzy = pd.DataFrame(df_combined[['Control', 'FuzzyWuzzy Percentage', 'FuzzyWuzzy Match']])
fuzzywuzzy
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>FuzzyWuzzy Percentage</th>
      <th>FuzzyWuzzy Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A formal cloud security governance program whi...</td>
      <td>55.0</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The IT Risk Committee must meet every other mo...</td>
      <td>57.0</td>
      <td>Events from all the cloud systems/services/ten...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cloud security must be integrated into Deloitt...</td>
      <td>57.0</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A cloud security operating model shall be deve...</td>
      <td>58.0</td>
      <td>An accurate and up to date inventory of all as...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cloud security tooling and automation shall be...</td>
      <td>52.0</td>
      <td>Standardized application security vulnerabilit...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Perimeter security controls to be implemented</td>
      <td>68.0</td>
      <td>Role Based Access Controls (RBAC) must be impl...</td>
    </tr>
    <tr>
      <th>361</th>
      <td>Environmental controls, such as generators, Un...</td>
      <td>52.0</td>
      <td>User sessions must be automatically terminated...</td>
    </tr>
    <tr>
      <th>362</th>
      <td>Supporting utilities to have controls in place...</td>
      <td>52.0</td>
      <td>IAM solution must be capable of locking or dea...</td>
    </tr>
    <tr>
      <th>363</th>
      <td>Equipment maintenance to be performed to ensur...</td>
      <td>57.0</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>364</th>
      <td>In cases where a cloud service contains regula...</td>
      <td>55.0</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
  </tbody>
</table>
<p>365 rows × 3 columns</p>
</div>




```python
fuzzywuzzy[(fuzzywuzzy['FuzzyWuzzy Percentage'] > 70)]

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>FuzzyWuzzy Percentage</th>
      <th>FuzzyWuzzy Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>67</th>
      <td>Access to high risk unauthorized cloud service...</td>
      <td>73.0</td>
      <td>User accounts must be automatically disabled a...</td>
    </tr>
    <tr>
      <th>74</th>
      <td>On an annual basis, a review shall be performe...</td>
      <td>80.0</td>
      <td>An annual review of all users with privileged ...</td>
    </tr>
    <tr>
      <th>82</th>
      <td>Deloitte U.S. Firm shall:</td>
      <td>72.0</td>
      <td>Password complexity and expiration policies mu...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>73.0</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>User system access is removed upon termination...</td>
      <td>78.0</td>
      <td>Upon personnel termination, their account must...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>93.0</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>88.0</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>106</th>
      <td>This access shall be reviewed by the respectiv...</td>
      <td>96.0</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Deloitte U.S. Firm Users</td>
      <td>72.0</td>
      <td>Password complexity and expiration policies mu...</td>
    </tr>
    <tr>
      <th>114</th>
      <td>External Users</td>
      <td>73.0</td>
      <td>Protect stored credentials: All authentication...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>IAM solutions for cloud environments must be c...</td>
      <td>96.0</td>
      <td>User credentials must be stored using cryptogr...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>An inventory of key cloud information assets s...</td>
      <td>92.0</td>
      <td>Assets must be tracked throughout the asset li...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Assets associated with cloud and cloud process...</td>
      <td>85.0</td>
      <td>Assets must be tracked throughout the asset li...</td>
    </tr>
    <tr>
      <th>138</th>
      <td>The records must contain the following attribu...</td>
      <td>76.0</td>
      <td>Events from all the cloud systems/services/ten...</td>
    </tr>
    <tr>
      <th>139</th>
      <td>Owner</td>
      <td>100.0</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Owners of cloud-based applications must ensure...</td>
      <td>94.0</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>77.0</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>151</th>
      <td>A data lifecycle for cloud-based data shall be...</td>
      <td>75.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Who can create data</td>
      <td>100.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>153</th>
      <td>Who can access data within a cloud-based appli...</td>
      <td>87.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>154</th>
      <td>Who can share or export data outside of the ap...</td>
      <td>100.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>155</th>
      <td>If and how long data can be archived</td>
      <td>100.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Policy and workflow for deleting data used in ...</td>
      <td>84.0</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>97.0</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>99.0</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>100.0</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>100.0</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>100.0</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>175</th>
      <td>All engagement or client data stored within cl...</td>
      <td>72.0</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>100.0</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>78.0</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>97.0</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>For certain classifications of data, Deloitte ...</td>
      <td>73.0</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>221</th>
      <td>Intrusion Prevention Systems (IPS) are in plac...</td>
      <td>78.0</td>
      <td>Intrusion Detection and Intrusion Prevention s...</td>
    </tr>
    <tr>
      <th>222</th>
      <td>Network filtering shall be implemented using A...</td>
      <td>77.0</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>225</th>
      <td>IPS solutions for cloud environments must:</td>
      <td>74.0</td>
      <td>Single Sign-on Capabilities via Deloitte appro...</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Support automatic intrusion detection, alertin...</td>
      <td>94.0</td>
      <td>Automatic intrusion detection, alerting and re...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>93.0</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Monitor remote connections, detect aberrant be...</td>
      <td>87.0</td>
      <td>Aberrant behavior on remote connections must b...</td>
    </tr>
    <tr>
      <th>236</th>
      <td>Cloud-hosted applications have a production en...</td>
      <td>83.0</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Application code and endpoints for cloud-hoste...</td>
      <td>95.0</td>
      <td>Application code and endpoints for cloud-hoste...</td>
    </tr>
    <tr>
      <th>244</th>
      <td>Application code is scanned and validated prio...</td>
      <td>72.0</td>
      <td>Penetration Testing must be done all applicati...</td>
    </tr>
    <tr>
      <th>245</th>
      <td>Applications have a Deloitte custom domain nam...</td>
      <td>90.0</td>
      <td>All Deloitte hosted applications MUST have a D...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>92.0</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>94.0</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>A Business Impact Assessment (BIA) must be per...</td>
      <td>72.0</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Risks related to cloud service providers must ...</td>
      <td>86.0</td>
      <td>Risks related to third parties must be identif...</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(fuzzywuzzy[(fuzzywuzzy['FuzzyWuzzy Percentage'] > 70)])
```




    47



After doing some manual comparisons, I was able to determine that a good score for an accurate match was typically any result above 70, which is shown above. As you can see, most of the controls above do correlate/match with each other.  There are abot 47 matches out of 365 possible inputs.

### JellyFish Library
This library is used to compare string similarity using multiple different algorithms, such as Levenshtein, Damerau-Levenshtein, Jaro, and Jaro-Winkler distance.

`Damerau-Levenshtein` is a modification of Levenshtein distance, Damerau-Levenshtein distance counts transpositions (such as ifhs for fish) as a single edit. 

`Jaro` distance is a string-edit distance that gives a floating point response in [0,1] where 0 represents two completely dissimilar strings and 1 represents identical strings.

`Jaro-Winkler` is a modification/improvement to Jaro distance, like Jaro it gives a floating point response in [0,1] where 0 represents two completely dissimilar strings and 1 represents identical strings.

https://jellyfish.readthedocs.io
https://en.wikipedia.org/wiki/Levenshtein_distance


```python
#################### jellyfish ####################
compare_time = time.time()
jw_highest = jaro_highest = 0
dl_lowest = 999
dl_similarity = jw_similarity = jaro_similarity = 0
dl_match = jw_match = jaro_match = ''

#array = df_f2[column2].values

for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        dl_similarity = jf.levenshtein_distance(search1, search2)
        jw_similarity = jf.jaro_winkler(search1, search2)
        jaro_similarity = jf.jaro_distance(search1, search2)
        if (dl_lowest > dl_similarity):
            dl_lowest = dl_similarity
            dl_match = search2
        if (jw_highest < jw_similarity):
            jw_highest = jw_similarity
            jw_match = search2
        if (jaro_highest < jaro_similarity):
            jaro_highest = jaro_similarity
            jaro_match = search2
    df_f1.at[i,'DL Match Percentage'] = dl_lowest
    df_f1.at[i,'DL Closest Match'] = dl_match
    df_f1.at[i,'jw Match Percentage'] = jw_highest
    df_f1.at[i,'jw Closest Match'] = jw_match
    df_f1.at[i,'jaro Match Percentage'] = jaro_highest
    df_f1.at[i,'jaro Closest Match'] = jaro_match
    jw_highest = jaro_highest = 0
    dl_lowest = dl_lowest = 999
    dl_match = jw_match = jaro_match = ''

df_combined['DL Match Percentage'] = df_f1['DL Match Percentage']
df_combined['DL Closest Match'] = df_f1['DL Closest Match']
df_combined['jw Match Percentage'] = df_f1['jw Match Percentage']
df_combined['jw Closest Match'] = df_f1['jw Closest Match']
df_combined['jaro Match Percentage'] = df_f1['jaro Match Percentage']
df_combined['jaro Closest Match'] = df_f1['jaro Closest Match']
df_f1.to_excel(writer, sheet_name='Jellyfish', index=None)
df_f1.drop(columns=['DL Match Percentage', 'DL Closest Match','jw Match Percentage','jw Closest Match', 'jaro Match Percentage','jaro Closest Match'], inplace=True)
print('finished JellyFish Analysis - %s seconds' % (time.time() - compare_time))
```

    finished JellyFish Analysis - 8.689714908599854 seconds



```python
jf_dl = pd.DataFrame(df_combined[['Control', 'DL Match Percentage', 'DL Closest Match']])
jf_dl[(jf_dl['DL Match Percentage'] < 40)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>DL Match Percentage</th>
      <th>DL Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>43</th>
      <td>Additional required training</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>46</th>
      <td>A fine for a Partner</td>
      <td>34.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>82</th>
      <td>Deloitte U.S. Firm shall:</td>
      <td>39.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>85</th>
      <td>Implement timeout for user sessions</td>
      <td>37.0</td>
      <td>MFA must be required for establishing all remo...</td>
    </tr>
    <tr>
      <th>108</th>
      <td>Accounts for Cloud-based applications shall be:</td>
      <td>38.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>112</th>
      <td>Deloitte U.S. Firm Users</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>114</th>
      <td>External Users</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>122</th>
      <td>Text messages to a registered mobile device.</td>
      <td>36.0</td>
      <td>Threats must be monitored in real time and mit...</td>
    </tr>
    <tr>
      <th>123</th>
      <td>Time-based one-time passwords.</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>140</th>
      <td>Asset name</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>31.0</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Who can create data</td>
      <td>36.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>155</th>
      <td>If and how long data can be archived</td>
      <td>31.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>34.0</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>12.0</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>0.0</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>6.0</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>9.0</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>20.0</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>225</th>
      <td>IPS solutions for cloud environments must:</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Support automatic intrusion detection, alertin...</td>
      <td>26.0</td>
      <td>Automatic intrusion detection, alerting and re...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>22.0</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>230</th>
      <td>Notify network layer/ volumetric DDoS attacks.</td>
      <td>37.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>27.0</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>281</th>
      <td>Manage exception monitoring for non-EA accounts.</td>
      <td>39.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>287</th>
      <td>Deloitte’s technology service desk</td>
      <td>35.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>288</th>
      <td>Publicly available news sites</td>
      <td>38.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>289</th>
      <td>Social media</td>
      <td>38.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>326</th>
      <td>Penalties for data breaches.</td>
      <td>35.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Perimeter security controls to be implemented</td>
      <td>32.0</td>
      <td>All network communications must be encrypted</td>
    </tr>
  </tbody>
</table>
</div>




```python
jf_jw = pd.DataFrame(df_combined[['Control', 'jw Match Percentage', 'jw Closest Match']])
jf_jw[(jf_jw['jw Match Percentage'] >0.85)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>jw Match Percentage</th>
      <th>jw Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>Cloud security must be integrated into Deloitt...</td>
      <td>0.851744</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>The Deloitte U.S. Firm shall assign specific r...</td>
      <td>0.854011</td>
      <td>The application must avoid using user-controll...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Cloud Risk assessments shall be performed prio...</td>
      <td>0.851577</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>The organization shall periodically monitor cl...</td>
      <td>0.852378</td>
      <td>The application must be designed to direct use...</td>
    </tr>
    <tr>
      <th>39</th>
      <td>All personnel shall acknowledge their review a...</td>
      <td>0.857352</td>
      <td>All data must be encrypted in transit as well ...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>0.885886</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>0.876854</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>Cloud-hosted application owners must identify ...</td>
      <td>0.856696</td>
      <td>Cloud hosted applications with dependencies on...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.896224</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.885763</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.925810</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.886400</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.920636</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>175</th>
      <td>All engagement or client data stored within cl...</td>
      <td>0.850660</td>
      <td>All incidents must be reported via a single co...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.958155</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.889035</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>239</th>
      <td>Production cloud-hosted applications and envir...</td>
      <td>0.855317</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>244</th>
      <td>Application code is scanned and validated prio...</td>
      <td>0.860171</td>
      <td>Application and protocol layer inspections mus...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.901027</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>258</th>
      <td>A formal change request process is in place an...</td>
      <td>0.851175</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.870505</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>304</th>
      <td>All aspects of cloud-based solutions deemed to...</td>
      <td>0.851200</td>
      <td>All custom application code and components dep...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>A Business Impact Assessment (BIA) must be per...</td>
      <td>0.865190</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>309</th>
      <td>Cloud-based applications must be self-containe...</td>
      <td>0.857845</td>
      <td>Cloud hosted applications with dependencies on...</td>
    </tr>
    <tr>
      <th>310</th>
      <td>Cloud-based applications must have built-in re...</td>
      <td>0.853936</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>358</th>
      <td>Cloud Service Providers must have a documented...</td>
      <td>0.852031</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
jf_jaro = pd.DataFrame(df_combined[['Control', 'jaro Match Percentage', 'jaro Closest Match']])
jf_jaro[(jf_jaro['jaro Match Percentage'] >0.775)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>jaro Match Percentage</th>
      <th>jaro Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>0.809810</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>0.794757</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>An inventory of key cloud information assets s...</td>
      <td>0.776089</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.827040</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.809605</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.876350</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.810667</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.867726</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.930259</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>0.777169</td>
      <td>Appropriate personnel must be alerted automati...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.815059</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>191</th>
      <td>Secrets shall be stored in a secure repository...</td>
      <td>0.782206</td>
      <td>Security for all other data must be provided b...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>0.786906</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.835045</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.784176</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>301</th>
      <td>For high severity security cloud incidents, a ...</td>
      <td>0.776809</td>
      <td>Session locks must be enforced after a defined...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>A Business Impact Assessment (BIA) must be per...</td>
      <td>0.775317</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>329</th>
      <td>A cloud service provider must provide customer...</td>
      <td>0.775152</td>
      <td>Security for all other data must be provided b...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Risks related to cloud service providers must ...</td>
      <td>0.775847</td>
      <td>Session locks must be enforced after a defined...</td>
    </tr>
  </tbody>
</table>
</div>



Since the JellyFish library used three different algorithms, each producing different results, there had to be some manual analysis to determine which scores produced the best results, which are shown above.

### NLTK Library
This is the most popular library for natural language processing in Python.  It contains many modules to perform various types of analysis.  The ones that I will use for this analysis specifically are for tokenizing words, stemming words, removing stop words, and performing a cosine similarity.

Cosine similarity is a metric used to determine how similar documents are by performing some math algorithms, that are beyond the scope of this analysis.


https://dev.to/coderasha/compare-documents-similarity-using-python-nlp-4odp <br>
https://www.machinelearningplus.com/nlp/cosine-similarity/


```python
warnings.filterwarnings("ignore", category=UserWarning)
compare_time = time.time()

#################### Functions for cosine comparison ####################
stemmer = nltk.stem.porter.PorterStemmer()
wnl = WordNetLemmatizer()
stop_words=stopwords.words('english')
normalized_stop_words = normalize(str(stop_words))
vectorizer = TfidfVectorizer(tokenizer=normalize, stop_words='english')

#################### Loop to compare arrays ####################
highest = 0
similarity = 0
match = ''
for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        similarity = cosine_sim(search1, search2)
        if (highest < similarity):
            highest = similarity
            match = search2
    df_f1.at[i,'Match Percentage'] = highest
    df_f1.at[i,'Closest Match'] = match
    highest = 0
    match=''

df_combined['Cosine Match Percentage'] = df_f1['Match Percentage']
df_combined['Cosine Closest Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='Cosine Similarity', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
warnings.filterwarnings("default", category=UserWarning)

print('finished Cosine Analysis - %s seconds' % (time.time() - compare_time))
```

    finished Cosine Analysis - 162.02032113075256 seconds



```python
cosine = pd.DataFrame(df_combined[['Control', 'Cosine Match Percentage', 'Cosine Closest Match']])
cosine[(cosine['Cosine Match Percentage'] > 0.5)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>Cosine Match Percentage</th>
      <th>Cosine Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>93</th>
      <td>Restrict access to cloud services, cloud servi...</td>
      <td>0.526196</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>0.574930</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>0.510149</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>0.575490</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>106</th>
      <td>This access shall be reviewed by the respectiv...</td>
      <td>0.602127</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>IAM solutions for cloud environments must be c...</td>
      <td>0.605403</td>
      <td>User credentials must be stored using cryptogr...</td>
    </tr>
    <tr>
      <th>140</th>
      <td>Asset name</td>
      <td>0.536893</td>
      <td>Assets must be tracked throughout the asset li...</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Owners of cloud-based applications must ensure...</td>
      <td>0.740628</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.647329</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>151</th>
      <td>A data lifecycle for cloud-based data shall be...</td>
      <td>0.570633</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Who can create data</td>
      <td>0.597926</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>154</th>
      <td>Who can share or export data outside of the ap...</td>
      <td>0.586747</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>155</th>
      <td>If and how long data can be archived</td>
      <td>0.552247</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.846015</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>1.000000</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.685145</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>1.000000</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.966489</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>0.658351</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.503103</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Support automatic intrusion detection, alertin...</td>
      <td>0.752320</td>
      <td>Automatic intrusion detection, alerting and re...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>0.867364</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Monitor remote connections, detect aberrant be...</td>
      <td>0.859177</td>
      <td>Aberrant behavior on remote connections must b...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Application code and endpoints for cloud-hoste...</td>
      <td>0.502133</td>
      <td>Application code and endpoints for cloud-hoste...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.844895</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.595114</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
  </tbody>
</table>
</div>



This library takes quite a bit longer to perform than libraries used so far (162 seconds), but seems to produce some of the best results for matches thus far.


```python
len(cosine[(cosine['Cosine Match Percentage'] > 0.5)])
```




    27



### NLTK Library (Synonyms)
This analysis is very similar to the one above, except this takes it one step further and adds on synonyms into the equation.  NLTK has a module that will download synonyms for common words.  I used this module to parse through each input string, and add synonyms to that input string.  This means that if my input had the word `dog`, I would also search for results using the additional synonyms, as shown below.

```
>>> from nltk.corpus import wordnet
>>> syns = wordnet.synsets('dog')
>>> syns
[Synset('dog.n.01'), Synset('frump.n.01'), Synset('dog.n.03'), Synset('cad.n.01'), Synset('frank.n.02'), Synset('pawl.n.01'), Synset('andiron.n.01'), Synset('chase.v.01')]
```

https://www.guru99.com/wordnet-nltk.html <br>
https://www.machinelearningplus.com/nlp/cosine-similarity/


```python
#################### nltk cosine + synonyms ####################
warnings.filterwarnings("ignore", category=UserWarning)
compare_time = time.time()

#################### Functions for cosine comparison ####################
stop_words=stopwords.words('english')
normalized_stop_words = normalize(str(stop_words))
vectorizer2 = TfidfVectorizer(tokenizer=normalize, stop_words=normalized_stop_words)
vectorizer_syn = TfidfVectorizer(tokenizer=normalize_syn, stop_words=normalized_stop_words)

#################### Loop to compare arrays ####################
highest = 0
similarity = 0
match = ''
for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        similarity = cosine_sim_syn(search1, search2)
        if (highest < similarity):
            highest = similarity
            match = search2
    df_f1.at[i,'Match Percentage'] = highest
    df_f1.at[i,'Closest Match'] = match
    highest = 0
    match=''

df_combined['Cosine Synonym Match Percentage'] = df_f1['Match Percentage']
df_combined['Cosine Synonym Closest Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='Cosine Synonym Similarity', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
warnings.filterwarnings("default", category=UserWarning)

print('finished Cosine Synonym Analysis - %s seconds' % (time.time() - compare_time))

```

    finished Cosine Synonym Analysis - 247.038560628891 seconds



```python
cosine_synonym = pd.DataFrame(df_combined[['Control', 'Cosine Synonym Match Percentage', 'Cosine Synonym Closest Match']])
cosine_synonym[(cosine_synonym['Cosine Synonym Match Percentage'] > 0.5)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>Cosine Synonym Match Percentage</th>
      <th>Cosine Synonym Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>93</th>
      <td>Restrict access to cloud services, cloud servi...</td>
      <td>0.530461</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>0.618671</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>User system access is removed upon termination...</td>
      <td>0.539392</td>
      <td>Upon personnel termination, their account must...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>0.582609</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>0.622185</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>106</th>
      <td>This access shall be reviewed by the respectiv...</td>
      <td>0.684193</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>108</th>
      <td>Accounts for Cloud-based applications shall be:</td>
      <td>0.500460</td>
      <td>Upon personnel termination, their account must...</td>
    </tr>
    <tr>
      <th>119</th>
      <td>Authentication techniques employed must be com...</td>
      <td>0.506856</td>
      <td>Individual authenticator must be employed, and...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>IAM solutions for cloud environments must be c...</td>
      <td>0.617383</td>
      <td>User credentials must be stored using cryptogr...</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Owners of cloud-based applications must ensure...</td>
      <td>0.877377</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.677488</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Who can create data</td>
      <td>0.810993</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>153</th>
      <td>Who can access data within a cloud-based appli...</td>
      <td>0.769768</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>154</th>
      <td>Who can share or export data outside of the ap...</td>
      <td>0.763991</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>155</th>
      <td>If and how long data can be archived</td>
      <td>0.766596</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.802201</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.757935</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.561802</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.959822</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.977849</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>0.628126</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.863415</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>0.590242</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Application code and endpoints for cloud-hoste...</td>
      <td>0.581977</td>
      <td>Application code and endpoints for cloud-hoste...</td>
    </tr>
    <tr>
      <th>245</th>
      <td>Applications have a Deloitte custom domain nam...</td>
      <td>0.623933</td>
      <td>All Deloitte hosted applications MUST have a D...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.708977</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>258</th>
      <td>A formal change request process is in place an...</td>
      <td>0.505640</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.826236</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>324</th>
      <td>Circumstances in which data can be seized and ...</td>
      <td>0.511564</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>360</th>
      <td>Perimeter security controls to be implemented</td>
      <td>0.539505</td>
      <td>Route control and forced tunneling must be imp...</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(cosine_synonym[(cosine_synonym['Cosine Synonym Match Percentage'] > 0.5)])
```




    31



After review, I found that the synonyms actually produced more matches, and they did not necessarily result in better matches than the search without synonyms.  This is obviously only indicative of results for my specific dataset, but the results could have been different using different data.

### spaCy Library (Word Vector Similarity)
This library is an open-source library for NLP in Python.  It uses tons of NLP features such as tokenization, parts of speech tagging, lemmatization, similarity, training, etc.  It also includes models for various languages that are used as part of its algorithms to perform the processing.  I used the english large model, which is about 800MB and over 1 million word vectors.  This algorithm also takes a long time, likely due to the model it uses and advanced algorithms being performed.  I tried using the small and medium models, but the results were not very good.

The algorithm I chose to use was the word vector similarity function, which uses context-sensitive tensors using word vectors.

https://spacy.io/ <br>
https://www.geeksforgeeks.org/python-word-similarity-using-spacy/


```python
#################### Functions for spaCy comparison ####################
# this takes a long time  (15-30 mins to complete) and the results aren't good.
# removing for now
#https://spacy.io/usage/vectors-similarity
#################### Loop to compare arrays ####################
compare_time = time.time()
highest = 0
similarity = 0
match = ''

nlp = spacy.load('en_core_web_lg')

for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        a = nlp(search1)
        b = nlp(search2)
        similarity = a.similarity(b)
        if (highest < similarity):
            highest = similarity
            match = search2
    df_f1.at[i,'Match Percentage'] = highest
    df_f1.at[i,'Closest Match'] = match
    highest = 0
    match=''
    #print('row: %s - %s seconds' % (i, (time.time() - compare_time)))

df_combined['spaCy Similarity Match Percentage'] = df_f1['Match Percentage']
df_combined['spaCy Similarity Closest Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='spaCy Similarity Model', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
print('finished spaCy Analysis - %s seconds' % (time.time() - compare_time))
```

    finished spaCy Analysis - 1633.1330080032349 seconds



```python
spacy = pd.DataFrame(df_combined[['Control', 'spaCy Similarity Match Percentage', 'spaCy Similarity Closest Match']])
spacy[(spacy['spaCy Similarity Match Percentage'] > 0.95)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>spaCy Similarity Match Percentage</th>
      <th>spaCy Similarity Closest Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19</th>
      <td>The organization shall develop and roll out cu...</td>
      <td>0.952830</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Role-based cloud security training and awarene...</td>
      <td>0.963444</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Conduct that interferes with the normal and pr...</td>
      <td>0.952426</td>
      <td>Security for all other data must be provided b...</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Compliance with this standard shall be appropr...</td>
      <td>0.952751</td>
      <td>Code-level security reviews must be conducted ...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>Cloud Account Requestor/Owner Responsibilities...</td>
      <td>0.953101</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Cloud Account Remediation Process: Accounts fo...</td>
      <td>0.960725</td>
      <td>The application security must be reviewed, tes...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>0.976805</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>97</th>
      <td>For superuser access, access must be limited t...</td>
      <td>0.957289</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>User system access is removed upon termination...</td>
      <td>0.954497</td>
      <td>Upon personnel termination, their account must...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>0.959879</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>0.970727</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>106</th>
      <td>This access shall be reviewed by the respectiv...</td>
      <td>0.960233</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Cloud-based information systems must be config...</td>
      <td>0.961658</td>
      <td>Code-level security reviews must be conducted ...</td>
    </tr>
    <tr>
      <th>117</th>
      <td>Other authenticators (for e.g., service-specif...</td>
      <td>0.956044</td>
      <td>Security for all other data must be provided b...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>An inventory of key cloud information assets s...</td>
      <td>0.959385</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Assets associated with cloud and cloud process...</td>
      <td>0.952530</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>0.953875</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>0.989749</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.980501</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>1.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>0.957554</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.993713</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.998746</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>0.974853</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>0.977206</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>179</th>
      <td>Encryption must be used when transferring data...</td>
      <td>0.956714</td>
      <td>Data of a sensitive nature, commonly including...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>For certain classifications of data, Deloitte ...</td>
      <td>0.958832</td>
      <td>Data of a sensitive nature, commonly including...</td>
    </tr>
    <tr>
      <th>189</th>
      <td>If Deloitte U.S. Firm’s encryption parameters ...</td>
      <td>0.959301</td>
      <td>Security for all other data must be provided b...</td>
    </tr>
    <tr>
      <th>191</th>
      <td>Secrets shall be stored in a secure repository...</td>
      <td>0.953052</td>
      <td>Protect stored credentials: All authentication...</td>
    </tr>
    <tr>
      <th>212</th>
      <td>Network architecture and data flow diagrams fo...</td>
      <td>0.950337</td>
      <td>Dataflow of applications/systems/services must...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>0.963805</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>242</th>
      <td>An appropriate security and vulnerability revi...</td>
      <td>0.954803</td>
      <td>Penetration Testing must be done all applicati...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.987645</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>256</th>
      <td>A formal change request process shall be estab...</td>
      <td>0.971659</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>258</th>
      <td>A formal change request process is in place an...</td>
      <td>0.973588</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>270</th>
      <td>Logging shall be enabled for components that a...</td>
      <td>0.956056</td>
      <td>Code-level security reviews must be conducted ...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>0.983562</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>302</th>
      <td>Procedures for the identification, collection,...</td>
      <td>0.954237</td>
      <td>Code-level security reviews must be conducted ...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Risks related to cloud service providers must ...</td>
      <td>0.966894</td>
      <td>Risks related to third parties must be identif...</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(spacy[(spacy['spaCy Similarity Match Percentage'] > 0.95)])
```




    39



### spaCy Library (Word Movers Distance)
This library is an open-source library for NLP in Python.  It uses tons of NLP features such as tokenization, parts of speech tagging, lemmatization, similarity, training, etc.  It also includes models for various languages that are used as part of its algorithms to perform the processing.  I used the english large model, which is about 800MB and over 1 million word vectors.  This algorithm also takes a long time, likely due to the model it uses and advanced algorithms being performed.  I tried using the small and medium models, but the results were not very good.

The algorithm used for this section was the word movers distance (WMD).  WMD measures the dissimilarity between two text docuemnts as a mininum distance that the words of one document need to travel to reach the words of the second document.

![WMD](img/wmd.png)

The image bove shows an example of how WMD works.

https://spacy.io/ <br>
http://proceedings.mlr.press/v37/kusnerb15.pdf


```python
#################### Word Movers Distance ####################
compare_time = time.time()
lowest = 999
similarity = 0
match = ''
import spacy
nlp = spacy.load('en_core_web_lg')
nlp.add_pipe(wmd.WMD.SpacySimilarityHook(nlp), last=True)

for i,search1 in df_f1[column1].iteritems():
    for y,search2 in df_f2[column2].iteritems():
        a = nlp(search1)
        b = nlp(search2)
        similarity = a.similarity(b)
        if (similarity < lowest):
            lowest = similarity
            match = search2
    # print('\n\n%s -  \n %s - %s seconds' % (search1, search2, (time.time() - compare_time)))
    df_f1.at[i,'Match Percentage'] = lowest
    df_f1.at[i,'Closest Match'] = match
    lowest = 999
    match = ''

df_combined['WMD Percentage'] = df_f1['Match Percentage']
df_combined['WMD Match'] = df_f1['Closest Match']
df_f1.to_excel(writer, sheet_name='WMD Matcher', index=None)
df_f1.drop(columns=['Match Percentage','Closest Match'], inplace=True)
print('finished Word Movers Distance Analysis - %s seconds' % (time.time() - compare_time))
```

    finished Word Movers Distance Analysis - 1612.0687203407288 seconds



```python
spacyWMD = pd.DataFrame(df_combined[['Control', 'WMD Percentage', 'WMD Match']])
spacyWMD[(spacyWMD['WMD Percentage'] < 5)]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Control</th>
      <th>WMD Percentage</th>
      <th>WMD Match</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>The classification of information processed or...</td>
      <td>4.618134</td>
      <td>All log files must be retained based on Deloit...</td>
    </tr>
    <tr>
      <th>52</th>
      <td>The following are the approved business reason...</td>
      <td>4.695910</td>
      <td>Single Sign-on Capabilities via Deloitte appro...</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Access to high risk unauthorized cloud service...</td>
      <td>4.957110</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>74</th>
      <td>On an annual basis, a review shall be performe...</td>
      <td>4.292993</td>
      <td>An annual review of all users with privileged ...</td>
    </tr>
    <tr>
      <th>89</th>
      <td>Manage the access rights of Deloitte U.S. Firm...</td>
      <td>4.155075</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>93</th>
      <td>Restrict access to cloud services, cloud servi...</td>
      <td>4.578253</td>
      <td>Access to cloud services must be managed throu...</td>
    </tr>
    <tr>
      <th>96</th>
      <td>Access to cloud services or components shall b...</td>
      <td>3.242923</td>
      <td>Access provisioning procedures must be in plac...</td>
    </tr>
    <tr>
      <th>97</th>
      <td>For superuser access, access must be limited t...</td>
      <td>4.410384</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>User system access is removed upon termination...</td>
      <td>3.868181</td>
      <td>Upon personnel termination, their account must...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>The solution's authorization model must contai...</td>
      <td>4.927582</td>
      <td>The DevOps Team must grant access based on lea...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Temporary access to cloud environment resource...</td>
      <td>2.970409</td>
      <td>Temporary and or conditional access to cloud e...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>Privileged access to sensitive cloud resources...</td>
      <td>2.897202</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>106</th>
      <td>This access shall be reviewed by the respectiv...</td>
      <td>3.464033</td>
      <td>Privileged access to sensitive resources is re...</td>
    </tr>
    <tr>
      <th>125</th>
      <td>A hardware device, such as one complying with ...</td>
      <td>4.865202</td>
      <td>Password complexity and expiration policies mu...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>IAM solutions for cloud environments must be c...</td>
      <td>4.500697</td>
      <td>User credentials must be stored using cryptogr...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>An inventory of key cloud information assets s...</td>
      <td>3.929554</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>Assets associated with cloud and cloud process...</td>
      <td>4.692194</td>
      <td>Designated owners for all information assets m...</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Owners of cloud-based applications must ensure...</td>
      <td>1.933579</td>
      <td>Application owners must ensure data persists i...</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Member firm engagement data, if possible, shou...</td>
      <td>2.139845</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>151</th>
      <td>A data lifecycle for cloud-based data shall be...</td>
      <td>4.451331</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>153</th>
      <td>Who can access data within a cloud-based appli...</td>
      <td>4.968754</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>154</th>
      <td>Who can share or export data outside of the ap...</td>
      <td>4.342667</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>156</th>
      <td>Policy and workflow for deleting data used in ...</td>
      <td>4.530714</td>
      <td>A defined data lifecycle exists providing guid...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>Processes and procedures must be in-place to a...</td>
      <td>1.099449</td>
      <td>Processes and procedures are in-place to allow...</td>
    </tr>
    <tr>
      <th>165</th>
      <td>Production data for cloud-hosted applications ...</td>
      <td>0.000000</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>Separate isolated databases must be utilized t...</td>
      <td>0.000000</td>
      <td>Separate isolated databases must be utilized t...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>Sensitive information must not be stored in lo...</td>
      <td>3.743195</td>
      <td>Sensitive information must not be stored in lo...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>Member firm engagement data must be restricted...</td>
      <td>0.000000</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>170</th>
      <td>For component engagements, data must reside in...</td>
      <td>4.477229</td>
      <td>Member firm engagement data is restricted from...</td>
    </tr>
    <tr>
      <th>175</th>
      <td>All engagement or client data stored within cl...</td>
      <td>4.824306</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Unstructured data and documents must be encryp...</td>
      <td>0.454854</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Data pertaining to cloud-hosted applications s...</td>
      <td>2.898780</td>
      <td>Cloud-hosted application database connections ...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>A minimum of AES-256 encryption must be applie...</td>
      <td>2.191841</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>179</th>
      <td>Encryption must be used when transferring data...</td>
      <td>4.511196</td>
      <td>A minimum of AES 256 encryption must be used f...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>For certain classifications of data, Deloitte ...</td>
      <td>4.524586</td>
      <td>Unstructured data and documents must be encryp...</td>
    </tr>
    <tr>
      <th>191</th>
      <td>Secrets shall be stored in a secure repository...</td>
      <td>4.937891</td>
      <td>Protect stored credentials: All authentication...</td>
    </tr>
    <tr>
      <th>207</th>
      <td>Cloud-based environments shall be hardened in ...</td>
      <td>4.965754</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>221</th>
      <td>Intrusion Prevention Systems (IPS) are in plac...</td>
      <td>3.827814</td>
      <td>Intrusion Detection and Intrusion Prevention s...</td>
    </tr>
    <tr>
      <th>223</th>
      <td>Network access shall be restricted and limited...</td>
      <td>4.784213</td>
      <td>Concurrent user-sessions must be restricted/li...</td>
    </tr>
    <tr>
      <th>224</th>
      <td>Security and network configuration standards s...</td>
      <td>4.996677</td>
      <td>Changes to hardening standards must be reviewe...</td>
    </tr>
    <tr>
      <th>226</th>
      <td>Support automatic intrusion detection, alertin...</td>
      <td>1.117747</td>
      <td>Automatic intrusion detection, alerting and re...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>Perform packet filtering and analysis to preve...</td>
      <td>1.198358</td>
      <td>Packet filtering and analysis must be done to ...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Monitor remote connections, detect aberrant be...</td>
      <td>1.615264</td>
      <td>Aberrant behavior on remote connections must b...</td>
    </tr>
    <tr>
      <th>236</th>
      <td>Cloud-hosted applications have a production en...</td>
      <td>4.836402</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>239</th>
      <td>Production cloud-hosted applications and envir...</td>
      <td>4.549718</td>
      <td>Production data for cloud-hosted applications ...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Application code and endpoints for cloud-hoste...</td>
      <td>4.530451</td>
      <td>Cloud hosted applications with dependencies on...</td>
    </tr>
    <tr>
      <th>244</th>
      <td>Application code is scanned and validated prio...</td>
      <td>4.736532</td>
      <td>Standardized application security vulnerabilit...</td>
    </tr>
    <tr>
      <th>245</th>
      <td>Applications have a Deloitte custom domain nam...</td>
      <td>4.482683</td>
      <td>All Deloitte hosted applications MUST have a D...</td>
    </tr>
    <tr>
      <th>248</th>
      <td>Source code for cloud-hosted applications shal...</td>
      <td>4.617714</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>251</th>
      <td>Give real-time alerts on vulnerabilities in th...</td>
      <td>4.870120</td>
      <td>Threats must be monitored in real time and mit...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Cloud-hosted applications shall be deployed us...</td>
      <td>0.844157</td>
      <td>Cloud-hosted applications deployments must adh...</td>
    </tr>
    <tr>
      <th>254</th>
      <td>Deviations from standard baseline configuratio...</td>
      <td>4.869536</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>256</th>
      <td>A formal change request process shall be estab...</td>
      <td>3.854602</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>258</th>
      <td>A formal change request process is in place an...</td>
      <td>3.529906</td>
      <td>A formal change management process must be imp...</td>
    </tr>
    <tr>
      <th>277</th>
      <td>The Deloitte Network Operation Center (DNOC) m...</td>
      <td>4.894454</td>
      <td>Networking capacity must be monitored on an on...</td>
    </tr>
    <tr>
      <th>290</th>
      <td>Cloud threat management solutions must be able...</td>
      <td>2.198754</td>
      <td>Cloud applications must be monitored for unusu...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>A Business Impact Assessment (BIA) must be per...</td>
      <td>4.293568</td>
      <td>A Business Impact Assessment (BIA) must be per...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>Risks related to cloud service providers must ...</td>
      <td>3.633210</td>
      <td>Risks related to third parties must be identif...</td>
    </tr>
    <tr>
      <th>358</th>
      <td>Cloud Service Providers must have a documented...</td>
      <td>4.903331</td>
      <td>A formal change management process must be imp...</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(spacyWMD[(spacyWMD['WMD Percentage'] < 5)])
```




    59



After using all of these different libraries, it became obvious that I needed to see all of the results side by side in order to make a good determination on what levels of tolerance I needed to set for each of the scores for each algorithm.  The code below compiles all of the matches next to each other based on the score tolerance I set and produces the results Excel file.


```python
# Save
df_combined.to_excel(writer, sheet_name='Combined Results', index=None)

column_list = ['WMD Percentage',
                'spaCy Similarirty Match Percentage',
                'Cosine Match Percentage',
                'Cosine Synonym Match Percentage',
                'SequenceMatcher Percentage',
                'jaro Match Percentage',
                'jw Match Percentage',
                'FuzzyWuzzy Percentage',
                'DL Match Percentage',
                'DL Closest Match']
for col in column_list:
    if col not in df_combined.columns:
        df_combined[col] = np.nan

newdf = pd.DataFrame(df_combined[(df_combined['Cosine Match Percentage'] > 0.5) |
        (df_combined['SequenceMatcher Percentage'] > 0.5) |
        (df_combined['jaro Match Percentage'] > 0.775) |
        (df_combined['spaCy Similarity Match Percentage'] > 0.95) |
        (df_combined['DL Match Percentage'] < 40) |
        (df_combined['jw Match Percentage'] > 0.94) |
        (df_combined['Cosine Synonym Match Percentage'] > 0.50) |
        (df_combined['WMD Percentage'] < 5) |
        (df_combined['FuzzyWuzzy Percentage'] > 70)])
newdf.drop(columns=['Cosine Match Percentage', 'spaCy Similarity Match Percentage', 'Cosine Synonym Match Percentage','spaCy Similarirty Match Percentage', 'WMD Percentage', 'SequenceMatcher Percentage', 'DL Match Percentage',  'jaro Match Percentage', 'jw Match Percentage', 'FuzzyWuzzy Percentage'], inplace=True)
newdf.to_excel(writer, sheet_name='High Matches (Combined)', index=None)

writer.save()
writer.close()

print('Script completed analysis')

```

    Script completed analysis


The results of the excel file are linked below.

[Excel link](Results/results.xlsx)

A snippet of the results is shown in the image below.

![results](img/results.png)

### Conclusion
This task proved very valuable in determing methods of analyzing different datasets to find matches.  We looked at a ton of different methods and libraries, and adjusted the score tolerances to get the results that we felt were the best.  

