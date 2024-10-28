Embeddings are numerical representations (vectors) of text that capture semantic meaning. An **embedding model** transforms input text into a **numerical vector** that captures the **semantic meaning** of the text. These vectors allow us to compare the similarity between texts in a way that reflects their meaning rather than just their exact wording.

### How This Works

1. #### Preparing Documents:
  
  Each document is transformed into a string using its `name`, `description`, and `tags`. This text is what gets passed to the embedding model.
  
  ```js
  prepareDocumentText(doc) {
    return `Title: ${doc.name}\nDescription: ${doc.simulationDescription}\nTags: ${doc.simulationTags.join(', ')}`;
  }
  ```
  
  For example, the document:
  
  ```json
  {
    "_id": "SIM-6f36e0ef-4cbf-44a8-bc6b-356ef2298c57",
    "name": "Huygens' Principle",
    "simulationDescription": "This simulation illustrates Huygens' Principle by explaining the propagation of light through various wavefronts and visualizing the secondary wavelets that form a new wavefront.",
    "simulationTags": ["Huygens' Principle", "Wavefronts", "Physics", "Optics", "Light Propagation", "Physics Simulation"]
  }
  
  ```
  
  Gets converted into:
  
  ```textile
  Title: Huygens' Principle
  Description: This simulation illustrates Huygens' Principle by explaining the propagation of light through various wavefronts and visualizing the secondary wavelets that form a new wavefront.
  Tags: Huygens' Principle, Wavefronts, Physics, Optics, Light Propagation, Physics Simulation
  
  ```
  
  The text is converted into a vector using OpenAI's embedding model.
  
2. Embedding Creation:
  
  This string is passed to the OpenAI embedding model, which converts it into a numerical vector.
  
  ```javascript:Untitled-1
  async getEmbedding(text) {
    const response = await openai.embeddings.create({
      model: "text-embedding-3-small",
      input: text
    });
    return response.data[0].embedding;
  }
  
  ```
  
  The model understands and converts the text into an embedding that captures the **semantic meaning** of the document, positioning it in the multi-dimensional vector space.
  
3. Similarity Search:
  
  When a user submits a search query like "physics light propagation," the server creates an embedding of the query in the same way.
  
  ```js
  async search(query, topK = 3, threshold = 0.7) {
    const queryEmbedding = await this.getEmbedding(query);
    const similarities = [];
    // ...
  }
  ```
  
  This query embedding is compared to the stored embeddings of each document using **cosine similarity**, a method that calculates how close the vectors are in the vector space. The closer they are, the more relevant the document is to the query.
  

### If we Used Direct LLM Instead:

```javascript
async function searchWithLLM(query, documents) {
    const prompt = `Given these documents: ${JSON.stringify(documents)}
                    Find the most relevant documents for the query: "${query}"`;

    const response = await openai.chat.completions.create({
        model: "gpt-4",
        messages: [{ role: "user", content: prompt }]
    });

    return response.choices[0].message.content;
}
```

Problems with this approach:

- Much more expensive (uses many more tokens)
- Slower response times
- Limited by context window size
- Less consistent results
- Cannot efficiently handle large document collections

That's why embeddings are the preferred approach for semantic search implementations!

### Approx costing:

With embedding - Total per request: ~$0.0000002 (₹0.000017 INR)

Without embedding - Total per request: ~$0.0075 (₹0.625 INR)

### Cost Comparison for 10,000 Requests:

- With Embeddings: $2 (₹170 INR)
  
- Without Embeddings: $75 (₹6,250 INR)
