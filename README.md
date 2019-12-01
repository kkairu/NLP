
# spaCy Text Analysis App

Text Source - Kenya BBI Report 2019
https://d2s5ggbxczybtf.cloudfront.net/bbireport.pdf


## 1. Select Text File


```python
# Using the Tkinter graphical user interface (GUI) to select text file

from tkinter import filedialog
from tkinter import *
import os

root = Tk()
cwd = os.getcwd() # get current working directory
root.filename = filedialog.askopenfilename(initialdir = cwd,title = "Select file",filetypes = (("all files","*.*"),("plain text","*.txt"),('pdf','*.pdf')))

print(root.filename)
```

    D:/AnacondaProjects/NLP/Kenya BBI Report 2019.pdf
    

## 2. Extract Text


```python
# Extracting text from the file using Apache Tika

# **** #encodedData.close() # closes the file reading data  --- REMOVED BY KENNEDY KK 01-dec-2019
#      To sort 'bytes' object has no attribute 'close' == BUG to be sorted in 1.23 by @	chrismattmann

import tika
#tika.TikaClientOnly = True

from tika import unpack
#from tika import parser

parsed = unpack.from_file(root.filename)
#parsed = parser.from_file(root.filename, xmlContent=True)

# print(parsed["metadata"])
text=parsed["content"]   #text extracted from the file, as a continuous string
```

## 3. Process text - spaCy


```python
import spacy
#loading the SpaCy 'small' model
nlp = spacy.load("en_core_web_sm")

#Processing the text
doc = nlp(text)

#obtaining entity information
ent_text=[]
ent_label=[]
ent_sentence=[]
for entity in doc.ents:
    ent_text.append(entity.text) #the entity's text
    ent_label.append(entity.label_) # the type of entity, a string e.g. PERSON,GPE, etc
    ent_sentence.append(entity.sent.text.replace('\n',' ')) # a sentence with the entity, as one line with newline - \n - removed

len(ent_label),len(ent_text),len(ent_sentence),len(doc.ents) # check number of entities + related info
```




    (6596, 6596, 6596, 6596)



## 4. Export extracted info to file


```python
#Saving the information as a dataframe
import pandas as pd
df=pd.DataFrame({'Entity': ent_text,'Type': ent_label,'Context':ent_sentence})
df.head()
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
      <th>Entity</th>
      <th>Type</th>
      <th>Context</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>THE PRESIDENTIAL</td>
      <td>ORG</td>
      <td>A REPORT BY THE PRESIDENTIAL</td>
    </tr>
    <tr>
      <th>1</th>
      <td>UNITY ADVISORY  \nOCTOBER</td>
      <td>ORG</td>
      <td>ON BUILDING BRIDGES TO UNITY ADVISORY   OCTOBE...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>DATE</td>
      <td>ON BUILDING BRIDGES TO UNITY ADVISORY   OCTOBE...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Kenyatta International Convention Centre</td>
      <td>ORG</td>
      <td>Kenyatta International Convention Centre, Nair...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nairobi</td>
      <td>GPE</td>
      <td>Kenyatta International Convention Centre, Nair...</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Separating info for each entity. This gives a list of (label, label_dataframe) tuples.
entities=list(df.groupby('Type'))

#Exporting the entity info into an excel file, with a sheet for each type:
save_dir=filedialog.askdirectory(title='Select destination directory') # a GUI to select destination directory
with pd.ExcelWriter(save_dir+'/BBI-mining.xlsx') as writer:
    for x in entities:
        x[1].to_excel(writer, sheet_name=x[0], index=False)
        
print("=== DONE ===")
```
