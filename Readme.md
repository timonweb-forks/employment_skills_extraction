#Skill Set Mapping to Jobs


## Step 1.   Candidate Feature Set 

We looked at N-grams in the range [2,4] that  starts with trigger words such as 'perform','deliver', ''ability', 'avail' 'experience','demonstrate' or contain words such as  knowledge', 'licen', 'educat', 'able', 'cert'  etc. and harvested a large set of n-grams.

See: `candidate_features.tsv`


## Step 2. Coarse clustering 

We performed a coarse clustering using KNN on __stemmed__  N-grams, and generated  20 clusters.

They roughly clustered around the following hand-labeled themes. At this stage we found some interesting clusters such as `disabled veterans & minorities`. 


| Cluster                           | Number of Features   |
|--------------------------------|------|
| truly unknown                  | 7971 |
| knowledge                      | 5199 |
| unknown                        | 4050 |
| education                      | 2944 |
| previous experience            | 2931 |
| skills                         | 1844 |
| unknown/able                   | 1807 |
| unknown/ability                | 1266 |
| license                        | 1070 |
| accountable and accounts       | 404  |
| delivery                       | 360  |
| communication                  | 343  |
| required years of experience   | 313  |
| disabled veterans & minorities | 311  |
| sales experience               | 305  |
| education experience           | 297  |
| drivers license                | 281  |
| customer service               | 262  |
| tables/microsoft/cleaning      | 218  |
| computer skills                | 169  |
| dependable                     | 161  |
| merchandise                    | 60   |

Big clusters such as Skills, Knowledge, Education required further granular clustering. 

## Step 3. Reclustering using semantic mapping of keywords 

Within the big clusters, we performed further re-clustering and mapping of semantically related words.

For this, we used `python-nltk`s `wordnet.synset` feature.  With this semantically related   key phrases such as 'arithmetic skills',  'basic math',  'mathematical ability' could be mapped to  a single cluster. 

The end result of this process is a mapping of 
a `skill tag` to several `feature words` that can be matched in the job description text. 

For example: 

|skill tag | feature words |
-----------| -------|
| Analytic and Mathematical Ability | math, mathematics, arithmetic, analytic, analytical, ... | 


## Step 4. Matching Skill Tag to Job description 

At this step, for each skill tag we build a tiny vectorizer on its feature words, and apply the same vectorizer on the job description and compute the dot product. A value greater than zero of the dot product indicates at least one of the feature words is present in the job description. If so, we associate this skill tag with the job description. 

## Step 5: Convert the operation in Step 4 to an API call. 

Under `api/` we built an API that given a Job ID will return matched skills.

##Usage:

To fire up the API, run 

`python server.py`

## Tests 

Under `unittests/` run `python test_server.py`

##Dependencies

* A job description call: The API makes a call with the `job_id` to get the  job description text. This could be a database call, but in the `method=local` context, it read it off the local file `/model/job_desc.json`
* Skill to feature map:   `model/skillname_to_skilltags.tsv`  
	* Each line in file has the format `skill name:keyword_1,keyword_2,....,keyword_n`. e.g. `sales skills:sale,sales,selling,retail`
* Stop words: `/ model / skill_stop_words.txt` provides a way  to ignore or blacklist words in the vectorization steps.

## Call and  Response:

The API is called with a `json` payload of the format:
`{"job_id": "10000038"}`


If the job id/description is not found, the API returns an error 
` ERROR: job text could not be retrieved`

If the job description could be retrieved and skills could be matched, it returns a response like:

```Response:
  {"0": {"skill": "interpersonal and communication skills", "score": 2, "matched_tags": ["communication", "interpersonal"]}, "1": {"skill": "sales skills", "score": 4, "matched_tags": ["selling", "retail", "sale", "sales"]}}
```

Here, two skills could be matched to the job, namely "interpersonal and communication skills"  and "sales skills". It also shows which keywords matched the description and a score (number of matched keywords) for father introspection. 

