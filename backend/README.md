### Start Python Backend
```
uvicorn main:app --reload
```

### Create venv python
```
python3 -m venv venv
```

### Activate venv
```
venv\Scripts\activate
```

### Install requirements
```
pip install -r requirements.txt
```

### ENV File
```
LOGS_DATA_FILE="Data_FILE"
GEMINI_API_KEY="API_KEY"
GEMINI_MODEL_NAME="modelVersion:generationMethod" eg "gemini-1.5-flash:generateContent"
```

### Gemini Response
```json
Response body = 

{
  "message": "Welcome to the general chatbot!",
  "gemini_response": {
    "candidates": [
      {
        "content": {
          "parts": [
            {
              "text": "Gemini Answer"
            }
          ],
          "role": "model"
        },
        "finishReason": "STOP",
        "avgLogprobs": -0.2924201439656803
      }
    ],
    "usageMetadata": {
      "promptTokenCount": 115,
      "candidatesTokenCount": 537,
      "totalTokenCount": 652
    },
    "modelVersion": "gemini-1.5-flash"
  }
}
```