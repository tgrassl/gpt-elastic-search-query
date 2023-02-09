# gpt-elastic-search-query
This example shows how you can create elastic search queries with a LLM like GPT and natural language.

[Playground](https://platform.openai.com/playground/p/BHqy0KpwlWbZ9hhiaiQlVVNM?model=code-davinci-002)

## 1. Get available fields
First we need to get all available fields in the index, e.g. `log.level, kubernetes.labels.service, trace.id, message, container.id`. 

These could be fetched using the elastic search api and will be used later in the prompt.

## 2. Generate prompt
Here's the base prompt that we are going to use:
```
"""
Create a elastic search json query for the following text using the fields <<FIELDS>>. Only return the json and only use term instead of match.
<<QUERY_PROMPT>>
"""
```

`<<FIELDS>>` is replaced with the list of available fields.

`<<QUERY_PROMPT>>` is the input query prompt.

`only use term instead of match` specifies if we want fuzzy (match) or exact (term) matching.

## 3. Generate query
Now we put it all together and create an elastic search query using natural language!

Model: `code-davinci-002` or `text-davinci-003`

Query: `Get all "TypeError" error logs for the service "acme-prod" or "acme-qa" and traceid x746362fbh`

Prompt:
```
"""
Create a elastic search json query for the following text using the fields log.level, kubernetes.labels.service, trace.id, message, container.id. Only return the json and only use term instead of match.
Get all "TypeError" error logs for the service "acme-prod" or "acme-qa" and traceid x746362fbh
"""
```

Result:
```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "log.level": "error"
                    }
                },
                {
                    "term": {
                        "message": "TypeError"
                    }
                },
                {
                    "bool": {
                        "should": [
                            {
                                "term": {
                                    "kubernetes.labels.service": "acme-prod"
                                }
                            },
                            {
                                "term": {
                                    "kubernetes.labels.service": "acme-test"
                                }
                            }
                        ]
                    }
                },
                {
                    "term": {
                        "trace.id": "x746362fbh"
                    }
                }
            ]
        }
    }
}
```

__Another example:__

Query: `Get logs for the "nginx" container of service "acme-dev"`

Result:
```json
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "log.level": "info"
                    }
                },
                {
                    "term": {
                        "kubernetes.labels.service": "acme-dev"
                    }
                },
                {
                    "term": {
                        "container.id": "nginx"
                    }
                }
            ]
        }
    }
}
```
