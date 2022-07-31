# Thefuzz library experiment, fuzzy logic in action
Less define the problema and idea
This code was a helping hand for  a call center
This helping center normally input the location of the customer. 
It can be the house, office or in the middle of the street.
The customer normally give data in a fast way which cause problems and errors for the employee.
It create problems to define the specific geolocation of a call or event.

This code will help by joining new geo points with previous ones using Fuzzy Logic.
Normal comparison are less effective here. They tends to 
For example:
Old Record: The traffic accident occured in the 45th Street, in the Santa Lucia Bridge, Section 1.
New Record: by the brige, thre is a collision between vehcules, by the sec 1, 45 street.

A person will not have any problem undestanding this sentence, even with the gramatical errors, 
but a computer will have a hard time.
As a solution I will use Levenshtein distance in Python and a library called
thefuzz(originally fuzzywuzzy)
I will use 5 diferent sentences. I will check the ratios to see which generate a better solution or threshold.

```python
import numpy as np 
import pandas as pd 
```


```python
from thefuzz import fuzz
from thefuzz import process
```

## Some important concepts of the library used
ratio: To measure the Levenshtein similirity ratio between two strings.partial_ratio: It uses the shortest string, then compare it with substrings of the same length.token_sort_ratio: It tokenize the strings, set the strings to lowercase and clean of puntuations.token_set_ratio: It takes out the common tokens instead of just tokenizing the strings, sorting, and then calculating the ratio.
## Simple examples


```python
word1 = 'HELLO FUZZYWUZZY'
word2 = 'hello fuzzywuzzy'
```


```python
fuzz.ratio(word1,word2)
```




    6




```python
fuzz.partial_ratio(word1,word2)
```




    6




```python
fuzz.token_sort_ratio(word1,word2)
```




    100




```python
fuzz.token_set_ratio(word1,word2)
```




    100



## Creating our dataset


```python
# list of previous points
prelist = ['There is a hurt person nearby to the Santa Lucia school, in the 67 street', 
        'A traffic accident in the 96 avenue, with direction to the 45 street',
        'There is a kid who fall in a hole in the 75 stret , sec 23, near to the Alfred Cafe',
        'there is a collision between vehicules at the traffic line of the 45 section, 67 street',
        'A taxi crashed against a school bus in the San Juan Avenue, with direction to the 67 streeet'
      ]
  
# list of new points
newlist = ['in the 45 street there is a person blackout, at the 96 avenue',
          'at the section 23 of the 75 street there is a oldman that needs medical attention',
          'collision in the 67 street, with direction to the school',
          'a motorcicle crash in the 67 streett, 45 sec, medical support required',
          'At the 67th street, a collision between vehicules around the San Juan Ave.']
```


```python
df_fuzzy = pd.DataFrame(list(zip(prelist, newlist)),columns =['previous', 'new'])
df_fuzzy[:5]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>previous</th>
      <th>new</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>There is a hurt person nearby to the Santa Luc...</td>
      <td>in the 45 street there is a person blackout, a...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A traffic accident in the 96 avenue, with dire...</td>
      <td>at the section 23 of the 75 street there is a ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>There is a kid who fall in a hole in the 75 st...</td>
      <td>collision in the 67 street, with direction to ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>there is a collision between vehicules at the ...</td>
      <td>a motorcicle crash in the 67 streett, 45 sec, ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A taxi crashed against a school bus in the San...</td>
      <td>At the 67th street, a collision between vehicu...</td>
    </tr>
  </tbody>
</table>
</div>



## Using 'Fuzz' to find Ratios


```python
def measureRatio(row):
    return fuzz.ratio(str(row['previous']).lower(),str(row['new']).lower())

def measurePartialRatio(row):
    return fuzz.partial_ratio(str(row['previous']).lower(),str(row['new']).lower())

def measureTokenSortRatio(row):
    return fuzz.token_sort_ratio(row['previous'],row['new'])

def measureTokenSetRatio(row):
    return fuzz.token_set_ratio(row['previous'],row['new'])
```


```python
df_fuzzy['measureRatio'] = df_fuzzy.apply(lambda row: measureRatio(row), axis = 1)
```


```python
df_fuzzy['measurePartialRatio'] = df_fuzzy.apply(lambda row: measurePartialRatio(row), axis = 1)
```


```python
df_fuzzy['measureTokenSortRatio'] = df_fuzzy.apply(lambda row: measureTokenSortRatio(row), axis = 1)
```


```python
df_fuzzy['measureTokenSetRatio'] = df_fuzzy.apply(lambda row: measureTokenSetRatio(row), axis = 1)
```


```python
df_fuzzy[:]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>previous</th>
      <th>new</th>
      <th>measureRatio</th>
      <th>measurePartialRatio</th>
      <th>measureTokenSortRatio</th>
      <th>measureTokenSetRatio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>There is a hurt person nearby to the Santa Luc...</td>
      <td>in the 45 street there is a person blackout, a...</td>
      <td>51</td>
      <td>48</td>
      <td>61</td>
      <td>71</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A traffic accident in the 96 avenue, with dire...</td>
      <td>at the section 23 of the 75 street there is a ...</td>
      <td>44</td>
      <td>44</td>
      <td>54</td>
      <td>54</td>
    </tr>
    <tr>
      <th>2</th>
      <td>There is a kid who fall in a hole in the 75 st...</td>
      <td>collision in the 67 street, with direction to ...</td>
      <td>49</td>
      <td>59</td>
      <td>53</td>
      <td>52</td>
    </tr>
    <tr>
      <th>3</th>
      <td>there is a collision between vehicules at the ...</td>
      <td>a motorcicle crash in the 67 streett, 45 sec, ...</td>
      <td>43</td>
      <td>46</td>
      <td>48</td>
      <td>49</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A taxi crashed against a school bus in the San...</td>
      <td>At the 67th street, a collision between vehicu...</td>
      <td>45</td>
      <td>50</td>
      <td>58</td>
      <td>56</td>
    </tr>
  </tbody>
</table>
</div>


As we can see, depending of our objective, we should select the measure that fills the most our needs. Then we select a threshold. For example: in the Token Set Ratio, I will select a threshold of 45 for the ratio.
## Using 'Process' to find better matches


```python
process.extract('collision in the 67 street, with direction to the school', df_fuzzy['new'],scorer=fuzz.token_sort_ratio)
```




    [('collision in the 67 street, with direction to the school', 100, 2),
     ('in the 45 street there is a person blackout, at the 96 avenue', 54, 0),
     ('At the 67th street, a collision between vehicules around the San Juan Ave.',
      52,
      4),
     ('at the section 23 of the 75 street there is a oldman that needs medical attention',
      51,
      1),
     ('a motorcicle crash in the 67 streett, 45 sec, medical support required',
      50,
      3)]



In this case the best solution is the first one because it gave a 100 in similirity.
