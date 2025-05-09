# Simple QA Chatbot Demo

FastAPI-sovellus, joka vastaa kysymyksiin paikallisesta JSON-pohjaisesta QA-tietokannasta fuzzy-hakua käyttäen.

## Asennus

```bash
git clone https://github.com/yourusername/simple-qa-chatbot.git
cd simple-qa-chatbot
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
````

## Käyttö

```bash
uvicorn app:app --reload
```

Palvelin käynnistyy osoitteeseen `http://127.0.0.1:8000` ja dokumentaatio löytyy `http://127.0.0.1:8000/docs`.

```

---

### requirements.txt

```

fastapi
uvicorn

````

---

### app.py

```python
import json
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from difflib import SequenceMatcher

# Ladataan QA-parit JSON-tiedostosta
with open("qa_pairs_en.json", encoding="utf-8") as f:
    qa_pairs = json.load(f)

# FastAPI-sovellus
app = FastAPI()
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Pyyntön skeema
class ChatRequest(BaseModel):
    question: str

# Fuzzy-match-funktio
def find_best_match(question: str) -> tuple[str, float]:
    best_ans, best_score = "", 0.0
    for qa in qa_pairs:
        parts = qa.get("text", "").split("Answer:")
        if len(parts) != 2:
            continue
        q_txt = parts[0].replace("Question:", "").strip()
        ans = parts[1].strip()
        score = SequenceMatcher(None, question.lower(), q_txt.lower()).ratio()
        if score > best_score:
            best_score, best_ans = score, ans
    return best_ans, best_score

@app.post("/chat")
async def chat(req: ChatRequest):
    ans, score = find_best_match(req.question)
    if score >= 0.8:
        return {"answer": ans, "score": round(score, 2)}
    raise HTTPException(status_code=404, detail="No good match found")
````

---

### qa\_pairs\_en.json

```json
[
  {"text": "Question: What is FastAPI? Answer: FastAPI is a modern, high-performance Python web framework for building APIs."},
  {"text": "Question: How do you install dependencies? Answer: Run `pip install -r requirements.txt` after activating your virtual environment."}
]
```
