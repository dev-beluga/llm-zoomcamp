# HOMEWORK 1 ANSWERS

## Ans1. Running Elastic

Run Elastic Search 8.17.6, and get the cluster information. If you run it on localhost, this is how you do it:

```console
curl localhost:9200
```

What's the `version.build_hash` value?

Check for build_hash value which is found in version Object containing version details

```console
dbcbbbd0bc4924cfeb28929dc05d82d662c527b7
```

## Ans2. Indexing the data

Index the data in the same way as was shown in the course videos. Make the `course` field a keyword and the rest should be text. 

Which function do you use for adding your data to elastic?

```text
[x] `index`
```

```bash
es_client.index(index=index_name, document=doc)
```

## Ans3. Searching

Now let's search in our index.

We will execute a query "How do execute a command on a Kubernetes pod?".

Use only `question` and `text` fields and give `question` a boost of 4, and use `"type": "best_fields"`.

What's the score for the top ranking result?

```bash
[x] 44.50
```

```python
search_query = {
            "size": 5,
            "query": {
                "bool": {
                    "must": {
                        "multi_match": {
                            "query": "How do execute a command on a Kubernetes pod?",
                            "fields": ["question^4", "text"],
                            "type": "best_fields"
                        }
                    }
                    
                }
            }
        }
    
response = es_client.search(index=index_name, body=search_query)

top_result = response['hits']['hits'][0]['_score']
top_result

44.50556

```

## Ans4. Filtering

Now ask a different question: "How do copy a file to a Docker container?".

This time we are only interested in questions from `machine-learning-zoomcamp`.

Return 3 results. What's the 3rd question returned by the search engine?

```bash
[x] How do I copy files from a different folder into docker container’s working directory?
```

```bash
search_query = {
            "size": 3,
            "query": {
                "bool": {
                    "must": {
                        "multi_match": {
                            "query": "How do copy a file to a Docker container?",
                            "fields": ["question^4", "text"],
                            "type": "best_fields"
                        }
                    },
                    "filter": {
                        "term": {
                            "course": "machine-learning-zoomcamp"
                        }
                    }
                }
            }
        }
    
response = es_client.search(index=index_name, body=search_query)

result_docs = []

for hit in response['hits']['hits']:
    # print(hit)
    result_docs.append({
            'source': hit['_source'],
            'score': hit['_score']
            })

result_docs[2]['source']['question']
```

```console
'How do I copy files from a different folder into docker container’s working directory?'

```

## Ans5. Building a prompt

Now we're ready to build a prompt to send to an LLM.

What's the length of the resulting prompt? (use the `len` function)

* 946
* 1446
* 1946
* 2446

We will use the context template to build the context on the records returned from  the question 4.

```bash
context = ""


for doc in result_docs:
    context += context_template.format(question=doc['source']['question'],text=doc['source']['text'])+"\n\n"    
```

Use the context to build a prompt from the  template.

```python
prompt = prompt_template.format(question=query,context=context)

#use len() to find the length of the prompt
print(len(prompt))
```

```console
#Output
1448
```

So, from the available choices 1446 is more closet with my answer.

```bash
[x] 1446
```

## Ans6. Tokens

The OpenAI python package uses `tiktoken` for tokenization:

```bash
pip install tiktoken
```
Let's calculate the number of tokens in our query:

```python
encoding = tiktoken.encoding_for_model("gpt-4o")
```

Use the `encode` function. How many tokens does our prompt have?

* 120
* 220
* 320
* 420

```python
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o")

encoding

#Output
<Encoding 'o200k_base'>

tokens = encoding.encode(prompt)

# print(f"Prompt: {prompt}")
print(f"Number of tokens: {len(tokens)}")
```

Calculate tokens for the complete prompt using `encode` function

```console
Number of tokens: 321
```
I will choose the closest answer.

```bash
[x] 320
```

## Ans7. Bonus: generating the answer (ungraded)

Let's send the prompt to OpenAI. What's the response?  

Note: I am using groq , it have identical API structure with OpenAI.

```python
api_key = os.getenv("GROQ_API_KEY")
client = Groq(api_key=api_key)

response = client.chat.completions.create(
model="llama-3.3-70b-versatile",
messages=[
    {
        "role": "user",
        "content": prompt
    }
],

)

answer = response.choices[0].message.content
print(answer)
```

This is the answer using llama LLM model.

```console
To copy a file to a Docker container, you can use the `docker cp` command. The basic syntax is as follows: 
`docker cp /path/to/local/file container_id:/path/in/container` 

You can find the `container_id` by running the `docker ps` command.
```

## Ans8. Bonus: calculating the costs (ungraded)

Suppose that on average per request we send 150 tokens and receive back 250 tokens.

How much will it cost to run 1000 requests?

On June 16, 2025, the prices for gpt4o are:

```number
* Input: $0.005 / 1K tokens
Input tokens: 150 × 1,000 = 150,000
Input price: 150,000 ÷ 1,000 × $0.005 = $0.75

* Output: $0.015 / 1K tokens
Output tokens: 250 × 1,000 = 250,000
Output price: 250,000 ÷ 1,000 × $0.015 = $3.75

Total Cost for 1,000 Requests with GPT-4o would be around $4.50
​
 
But if we used Groq LLaMA 3.3 70B the estimated cost would be ~$0.21 cent for the same prompt, which is 21x more cheaper but with lower reasoning and multilingual capabilities.
```

