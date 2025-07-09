  // IMPORTANT: Replace with your actual Gemini API key.
  // Ensure this API key has no HTTP referrer restrictions for Replit,
  // or add "*" as an allowed HTTP referrer in Google Cloud Console.
  const API_KEY = "AIzaSyAJ9z-vjFgnd7LYBxEMrA37V7dE9BtV-y4"; // Your API Key here

  async function translateWord() {
    const word = document.getElementById("wordInput").value.trim(); // Trim whitespace
    const output = document.getElementById("output");

    if (!word) {
      output.innerText = "Please enter a word.";
      return;
    }

    output.innerText = "Translating...";

    // Define the prompt for the LLM
    const prompt = `Translate the English word '${word}' into Arabic.
    Provide the Arabic script, its English transliteration, a simple example sentence in Arabic, and its Romanized English version.`;

    // Define the expected JSON schema for the LLM response
    const payload = {
      contents: [{
        parts: [{ text: prompt }]
      }],
      generationConfig: {
        responseMimeType: "application/json", // Explicitly ask for JSON
        responseSchema: {
          type: "OBJECT",
          properties: {
            "arabic_script": { "type": "STRING" },
            "english_transliteration": { "type": "STRING" },
            "example_arabic": { "type": "STRING" },
            "example_romanized": { "type": "STRING" }
          },
          // Ensure the order of properties in the response
          "propertyOrdering": ["arabic_script", "english_transliteration", "example_arabic", "example_romanized"]
        }
      }
    };

    try {
      // Using gemini-2.0-flash for client-side use, which is generally more stable
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
      });

      if (!response.ok) {
        const errorData = await response.json(); // Try to parse error response
        console.error("API Error Response:", errorData);
        throw new Error(`API request failed: ${errorData.error?.message || response.statusText}`);
      }

      const result = await response.json();

      // The response will now directly be the JSON object due to responseMimeType
      // We expect the JSON object to be in result.candidates[0].content.parts[0].text
      const jsonString = result.candidates?.[0]?.content?.parts?.[0]?.text;

      if (!jsonString) {
        throw new Error("No valid JSON content found in LLM response.");
      }

      const json = JSON.parse(jsonString);

      // Validate that all expected properties exist
      if (json.arabic_script && json.english_transliteration &&
          json.example_arabic && json.example_romanized) {
        output.innerHTML = `
          <p>🔤 Arabic: <strong>${json.arabic_script}</strong></p>
          <p>📝 Transliteration: <em>${json.english_transliteration}</em></p>
          <p>🗣️ Example (Arabic): ${json.example_arabic}</p>
          <p>📢 Example (Romanized): ${json.example_romanized}</p>
        `;
      } else {
        throw new Error("Incomplete data received from LLM. Missing one or more fields.");
      }

    } catch (e) {
      console.error("Translation failed:", e);
      output.innerText = `Error: ${e.message}. Please check your API key and browser console for more details.`;
    }
  }

  // You'll need an HTML structure for this to work in Replit.
  // Example HTML (e.g., in index.html):
  /*
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Arabic Translator</title>
      <style>
          body { font-family: sans-serif; padding: 20px; }
          input { padding: 8px; margin-right: 10px; }
          button { padding: 8px 15px; cursor: pointer; }
          #output { margin-top: 20px; border: 1px solid #ccc; padding: 15px; border-radius: 5px; }
      </style>
  </head>
  <body>
      <h1>English to Arabic Translator</h1>
      <input type="text" id="wordInput" placeholder="Enter an English word">
      <button onclick="translateWord()">Translate</button>
      <div id="output"></div>
      <script src="script.js"></script>
  </body>
  </html>
  */
