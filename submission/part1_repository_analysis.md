# Repository Analysis 


## My judgement parmaters :-
1. To know if a repo is strictly python based or not, I simply looked at the github bar which shows the most used language in the repository. Moreover I looked for `requirements.txt` file to make sure the respositories are strictly python based. 
2. For checking dependencies in Python projects, I mainly looked at the `requirements.txt` and `pyproject.toml` files.
3.  

## Comparison Table :-

|          | aiokafka |  airbyte  | archivematica | beets  | MetaGPT  |
|----------|----------|----------|----------|----------|----------|
| Striclty Python based  | Yes   | No   | Yes   | Yes   | Yes   |
| Primary Purpose | `aiokafka` is a aysync python library used to send and receive messages to kafka. It is mainly used in building scalable systems.   | `airbyte` is a tool that automatically moves data between databases, apps and AI systems. Due to this we do not have to write complex codes for it.   | `archivematica` is an open source tool that helps libraries, museums and all other orgs to safely store and preserve digital files.    | `beets` is a tool for music lovers that automatically keeps your playlist clean and properly arranged without any manual work.   | `metaGPT` is an AI tool that works like a full-fledged software company where different AI work on a simple idea to make it production ready.  |
| Key Dependencies   | `async-timeout` `packaging` `typing_extensions` `cramjam` `gssapi`    | `Gradle` (java)   | `django` `mysql` `elasticsearch` `celery` `boto3`   | `mutagen3` `jellyfish` `unidecode` `musicbrainzngs` `requests` `jinja2`   | `openai` `anthropic` `pydantic` `tenacity` `aiohttp` `tiktoken` `beautifulsoup4` `playwright` `pyYAML`  |
| Architectural Patters   | Event driven architecture that implements kafka (fully aysnc) for distributed systems  | NA  | Django MVC   | Plugins based CLI application (with local sqlite db support)   | agent based with seperate role/action for each agent   |