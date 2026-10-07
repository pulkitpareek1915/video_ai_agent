# 🎬 AI Video Assistant

An end-to-end LLM application that turns **YouTube videos, meetings, and local audio/video files** into structured insights and lets you **chat with the content using Retrieval-Augmented Generation (RAG)**.

Paste a link or file path, and the app produces a transcript, title, summary, action items, key decisions, and open questions. Then ask follow-up questions grounded strictly in the transcript.

---

## ✨ Features

- **Flexible input**: YouTube URLs (via `yt-dlp`) or local audio/video files
- **Multilingual transcription**
  - English → OpenAI Whisper (runs locally)
  - Hinglish → Sarvam AI speech-to-text-translate API (outputs English)
- **Long-content handling**: audio is converted to 16 kHz mono WAV and split into 10-minute chunks
- **AI-generated insights**
  - Short professional title
  - Bullet-point summary using map-reduce summarization
  - Action items (task, owner, deadline)
  - Key decisions
  - Open questions / follow-ups
- **Chat with your video (RAG)**: transcript is chunked, embedded, stored in ChromaDB, and retrieved to answer questions. If the answer isn't in the transcript, the assistant says so instead of guessing
- **Two interfaces**: Streamlit web app with live pipeline progress and chat history, plus a CLI

---

## 🏗️ Architecture

```
YouTube URL / Local File
        │
        ▼
 Audio Processing (yt-dlp, FFmpeg, pydub)
   → WAV conversion → 10-min chunks
        │
        ▼
 Transcription
   ├── English  → Whisper (local)
   └── Hinglish → Sarvam AI API
        │
        ▼
 Full Transcript
   ├──► Summarizer       (LangChain map-reduce, Mistral)
   ├──► Extractor        (action items, decisions, open questions)
   └──► RAG Engine
          Text splitter → HuggingFace embeddings → ChromaDB
          → Retriever (top-k=4) → Mistral → Answer
```

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.10+ |
| LLM | Mistral AI (`mistral-small-latest`) |
| Orchestration | LangChain (LCEL) |
| Speech-to-Text | OpenAI Whisper, Sarvam AI API |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` (Sentence-Transformers) |
| Vector Database | ChromaDB |
| Audio | yt-dlp, FFmpeg, pydub |
| UI | Streamlit |
| Config | python-dotenv |

---

## 📁 Project Structure

```
AI-Video-Assistant/
├── app.py                  # Streamlit web app
├── main.py                 # CLI pipeline + chat
├── test.py                 # Quick pipeline test script
├── Requirements.txt
├── core/
│   ├── transcriber.py      # Whisper / Sarvam transcription
│   ├── summarizer.py       # Title + map-reduce summary
│   ├── extractor.py        # Action items, decisions, questions
│   ├── vector_store.py     # Chunking, embeddings, ChromaDB
│   └── rag_engine.py       # RAG chain (LCEL) and Q&A
└── utils/
    └── audio_processor.py  # Download, convert, chunk audio
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10 or higher
- [FFmpeg](https://ffmpeg.org/download.html) installed and available on your `PATH`
- A [Mistral AI](https://console.mistral.ai/) API key
- A [Sarvam AI](https://www.sarvam.ai/) API key (only needed for Hinglish audio)

### Installation

```bash
git clone https://github.com/<your-username>/AI-Video-Assistant.git
cd AI-Video-Assistant

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r Requirements.txt
```

### Configuration

Create a `.env` file in the project root:

```env
MISTRAL_API_KEY=your_mistral_api_key
SARVAM_API_KEY=your_sarvam_api_key      # optional, for Hinglish

# Optional overrides
WHISPER_MODEL=small                      # tiny | base | small | medium | large
SARVAM_STT_MODEL=saaras:v2.5
```

---

## ▶️ Usage

### Web app (Streamlit)

```bash
streamlit run app.py
```

1. Paste a YouTube URL or a local file path in the sidebar
2. Choose the language (`english` or `hinglish`)
3. Click **Analyse** and watch the pipeline progress
4. Review the summary, action items, decisions, and open questions
5. Ask questions in the chat box, e.g. *"What were the main decisions made?"*

### Command line

```bash
python main.py
```

Enter a URL or file path, select a language, read the generated report, then chat with the content. Type `exit` to quit.

---

## 🔍 How It Works

1. **Ingest**: `yt-dlp` downloads YouTube audio, or `pydub` converts a local file to 16 kHz mono WAV. Audio is split into 10-minute chunks
2. **Transcribe**: each chunk goes to Whisper (English) or Sarvam AI (Hinglish, in 25-second pieces to respect the API limit). Chunk transcripts are joined into one transcript
3. **Summarize**: the transcript is split (3000 chars, 200 overlap), each piece is summarized, and the partial summaries are combined into a final bullet-point summary
4. **Extract**: dedicated prompts pull out action items (with owner and deadline), key decisions, and unresolved questions
5. **Index**: the transcript is split into 500-character chunks, embedded with MiniLM, and stored in ChromaDB
6. **Answer**: a question retrieves the top 4 chunks, which are passed to Mistral with a prompt that restricts answers to the retrieved context

---

## ⚠️ Known Limitations

- ChromaDB uses a single persisted collection, so indexing a new video adds to existing data instead of replacing it. Delete the `vector_db/` folder between videos for clean results
- Processing is sequential, so very long videos can take a while, especially with the local Whisper model on CPU
- Retrieval uses plain similarity search without re-ranking
- No speaker diarization, so action-item owners depend on names mentioned in the audio
- No automated evaluation suite yet

---

## 🛣️ Roadmap

- [ ] Per-video vector collections
- [ ] FastAPI backend with `/process` and `/ask` endpoints
- [ ] Docker support
- [ ] Parallel chunk summarization
- [ ] Evaluation set for answer accuracy and latency
- [ ] Export results to PDF / Markdown
- [ ] Speaker diarization

---

## 🤝 Contributing

Contributions are welcome. Fork the repo, create a feature branch, and open a pull request.

## 📄 License

This project is licensed under the MIT License. Add a `LICENSE` file to the repository before publishing.

## 🙏 Acknowledgements

[OpenAI Whisper](https://github.com/openai/whisper) · [LangChain](https://www.langchain.com/) · [Mistral AI](https://mistral.ai/) · [ChromaDB](https://www.trychroma.com/) · [Sarvam AI](https://www.sarvam.ai/) · [Streamlit](https://streamlit.io/) · [yt-dlp](https://github.com/yt-dlp/yt-dlp)
