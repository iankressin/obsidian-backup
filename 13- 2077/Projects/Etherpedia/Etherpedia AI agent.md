## High level architecture
![[etherpedia-ai-agent-arch.excalidraw | {width: 100%}]]

The **AI agent** is responsible for:
- scrapping the web for content related to the topic **Operator** requested.
- it will store all the content in text files and later will summarize in the form of an article.
- the article will go through a revision process where the agent will run review the text and make all the necessary adjustments.
- store the content in a database, or other persistent storage method.

The **Operator** is responsible for: 
- Requesting new articles to **AI agent**
- Approve/decline/ask for changes in the revisions

The **API** should: 
- Expose an interface which anyone can read the contents of articles produced by **AI agent**
- Expose an interface where only allowed agents (apps, websites, etc) can suggest revisions
- Expose an interface where **Operators** can approve/decline/ask for changes in the revisions
### AI agent
The AI agent is built by three parts
- Scrapper
- Writer
- Editor

**Scrapper options:**
- 

**Writer options:**
- 

**Editor options:**
- 