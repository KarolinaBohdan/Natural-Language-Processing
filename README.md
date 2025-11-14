# Party–Topic Network Analysis of EU Parliament Speeches (2009)

This project builds a political knowledge graph from European Parliament debate speeches and analyzes the relationships between political parties and the topics they discuss. The connections are defined by the stance (critical, supportive or neutral) political parties take on the topics.  
We perform LLM-based information extraction, construct a bipartite Party–Topic network, compute centrality measures and network-level statistics, and visualize the structure of political discourse.

---

## 1. Dataset Used

**Dataset:**  
European Parliament Plenary Speeches (2009–2023)  
HuggingFace: RJuro/eu_debates

This dataset contains ~87,000 speeches with rich metadata:

- speaker name and party affiliation  
- debate title and date  
- speech text (in 23 EU languages)  
- English machine translations  
- language and role information  

The dataset is originally based on the corpus by Chalkidis & Brandl (2024) and preserved in JSONL/Parquet format.

---

## 2. Subset Choice and Rationale

For this project, we use a filtered subset:

- **Year:** 2009  
- **Sample size:** 1000 randomly sampled speeches  

### Why 2009?

- It is the *smallest year* in the dataset  
- Lower data volume allows fast iteration and experimentation  
- Still includes multiple parties and diverse debate topics  
- Suitable for building and testing the entire pipeline without computational overload

### Why sample 1000 speeches?

- LLM extraction is costly and slow  
- 1000 items is a good tradeoff between representativeness and feasibility  
- Ensures a manageable graph size while still revealing meaningful patterns

---

## 3. LLM Extraction Process

### 3.1 Goal

Transform each raw speech into structured information required for network analysis:

- `topic_name`  
- `stance`  
- `mentioned_topics`  
- `summary`  

These fields enable connections between parties and topics.

---

### 3.2 Step-by-Step Process

#### 1. Preprocessing

- Load the dataset  
- Filter for year 2009  
- Randomly sample 1000 speeches  

#### 2. Structured LLM Extraction

Each speech is sent to an LLM using a JSON schema, enforcing consistent formatting.  
The model returns the four target fields.

Typical prompt → LLM returns JSON:

```json
{
  "topic_name": "...",
  "stance": "...",
  "mentioned_topics": ["..."],
  "summary": "..."
}
```

#### 3. LLM Extraction Process

We used an LLM to extract structured information from each speech, generating four fields: *topic_name*, *stance*, *mentioned_topics*, and *summary*.

We performed LLM-based structured extraction using Gemini 2.5 Flash-Lite.

The workflow:

- Filter the dataset to speeches from 2009 and randomly sample 1000 items.  
- Send each speech (English text or translation) to the LLM using a structured JSON prompt.  
- Validate outputs, fix malformed rows, and re-run missing entries.  
- Merge the cleaned results back into the dataset.  

This creates a consistent and standardized representation of each speech for downstream analysis.

---

#### 4. Network Construction

We construct a *bipartite Party–Topic network*, where:

- One node set represents political parties  
- The other node set represents extracted topics  
- An edge indicates that a party discussed a topic  
- Edges also include the average stance expressed by each party toward that topic (color)  
- Edge weights reflect how many speeches link the party to that topic  

Nodes and edges are added in NetworkX, and the final interactive visualization is exported with PyVis as **party_topic_network.html**.

---

#### 5. Analysis

We compute several network measures to understand the structure of parliamentary discourse:

- **Degree centrality:**  
Parties with the highest degree centrality are PPE and S&D. They are connected to a large amount of topics. The most connected topic is climate change, showing it had high salience.

- **Betweenness centrality:**  
All nodes have a betweenness centrality of zero. This result is expected, because the Party–Topic network is a strictly bipartite structure where edges only connect party.

Together, these analyses show which topics dominated 2009 debates and how parties positioned themselves within the issue landscape.

---

## 4. Summary

The interactive bipartite graph helps illustrate how different political groups engage with major issues. Each orange node represents a party, each blue node represents a topic, and the thickness of the edges reflects how many speeches the party made on that topic. For example:
Climate Change is the most discussed topic, especially covered by the S&D and PPE.
Large Parties have a broader range of topics they cover.

For example, *Climate Change (General)* appears as one of the most central topics, with the thickest connections coming from S&D and PPE, indicating that these parties allocate substantial attention to this issue. The colors of the edges also provide sentiment information: green edges represent supportive, grey neutral, and red critical stances. In this case, both PPE and S&D mostly speak about climate change in a supportive or neutral tone.

The structure also shows that larger parties tend to be connected to a broader range of topics (many outgoing edges), whereas smaller groups have fewer issue engagements. This makes the visualization useful for quickly identifying topic salience, party issue breadth, and how parties position themselves on specific policy areas.

---

## 5. Limitations

- **Reading “between the lines”**  
  - nuances cannot be ideally understood by the LLM  
  - in our case as most texts are short or medium length. It should have done fine  

- **Hallucination**  
  - in longer texts it might hallucinate  

- **Expensive**  
  - we could only use 1000 speeches because otherwise there weren’t enough free credits  

- **Takes a lot of time**  
  - for the sample of 1000 speeches it ran for 2.5 hours  

- **Classification of the stances**  
  - groups might be confusing (e.g., topic is violence against women and the stance is supportive most of the times meaning people support a resolution against violence)  
  - classification does not represent the nuances of the speeches  
  - more detailed classifications are at risk of having lower precision and recall  

- **Topic granularity**  
  - LLM gives us very detailed main topics, which increases the accuracy of the model  
  - in further processing we face challenges because we have a high number of unique topics  
  - we would need to cluster these main topics into thematic fields  
  - our prompt should have been more precise  
  - e.g., “climate_change” and “climate_change_policy” should be clustered under the same label  
