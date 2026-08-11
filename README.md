# 🎥 Multimodal Video RAG System

A **Multimodal Retrieval-Augmented Generation (RAG)** system that processes YouTube videos by extracting their **visual and textual information**, indexing the extracted data into a vector database, retrieving relevant content, and using a **multimodal LLM** to answer questions about the video.

---

## 🚀 Project Overview

Traditional RAG systems mainly work with text documents.

This project extends the RAG architecture to **video data** by processing multiple modalities:

* 🎥 Video
* 🖼️ Images / video frames
* 🔊 Audio
* 📝 Transcribed text
* 📊 Video metadata

The system converts the video into searchable information and retrieves the most relevant text and visual information for a user's query.

### Architecture

```text
                    YouTube Video
                         │
                         ▼
                  Download Video
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        Extract Frames         Extract Audio
              │                     │
              ▼                     ▼
           Images              Speech → Text
              │                     │
              └──────────┬──────────┘
                         ▼
                  Text + Images
                         │
                         ▼
                    Embeddings
                         │
                         ▼
                      LanceDB
                         │
                         ▼
                  Vector Retrieval
                         │
                  ┌──────┴──────┐
                  ▼             ▼
              Text Results   Image Results
                  │             │
                  └──────┬──────┘
                         ▼
                 Multimodal LLM
                         │
                         ▼
                    Final Answer
```

---

## ✨ Features

* Download videos from YouTube
* Extract audio from videos
* Extract video frames/images
* Convert speech into text using Whisper
* Store processed information for retrieval
* Vector-based similarity retrieval
* Retrieve both text and image information
* Use metadata as additional context
* Generate answers using a multimodal LLM
* Built using LlamaIndex
* LanceDB vector store integration

---

## 🧠 Technologies Used

| Component             | Technology                 |
| --------------------- | -------------------------- |
| RAG Framework         | LlamaIndex                 |
| Vector Database       | LanceDB                    |
| Video Download        | PyTube                     |
| Video Processing      | MoviePy                    |
| Speech-to-Text        | OpenAI Whisper             |
| Deep Learning         | PyTorch                    |
| Image Processing      | Pillow / scikit-image      |
| Visualization         | Matplotlib                 |
| Multimodal Embeddings | CLIP integration installed |
| LLM                   | OpenAI Multimodal LLM      |
| Audio Processing      | FFmpeg / PyDub             |
| Language              | Python                     |

---

## 🔍 Retrieval Architecture

The retrieval component uses **dense/vector similarity retrieval**.

The basic process is:

```text
User Query
    │
    ▼
Query Embedding
    │
    ▼
Vector Similarity Search
    │
    ▼
Nearest Relevant Vectors
    │
    ▼
Top-K Results
    │
    ├── Text
    │
    └── Images
```

The system uses LanceDB as the vector store and retrieves the most similar content according to the configured `top_k` values.

### Current Retrieval Configuration

```python
retriever_engine = index.as_retriever(
    similarity_top_k=1,
    image_similarity_top_k=3
)
```

This is intended to retrieve:

* Top 1 relevant text result
* Top 3 relevant image results

---

## 🧩 Embedding Model

The project includes the LlamaIndex CLIP embedding integration:

```bash
pip install llama-index-embeddings-clip
```

CLIP is designed for multimodal representation and can represent images and text in a shared embedding space.

### Important

The current notebook installs the CLIP integration but does **not yet explicitly instantiate/configure the CLIP embedding model**.

Therefore, CLIP should currently be considered the **intended multimodal embedding component**, rather than claiming that the current implementation definitively uses a configured CLIP model.

---

## 🔎 Search Method vs Retrieval vs Vector Database

These components are different:

```text
CLIP
 ↓
Embedding Model

Dense Similarity
 ↓
Retrieval Method

LanceDB
 ↓
Vector Database / Vector Store

HNSW / IVF / etc.
 ↓
Possible ANN Indexing Algorithms
```

The current project explicitly uses:

**Vector database:** LanceDB

**Retrieval:** Dense/vector similarity retrieval

**Top-K:** Configured through LlamaIndex

The current implementation does **not explicitly configure HNSW, IVF, BM25, or hybrid search**.

---

## 🎬 Video Processing Pipeline

### 1. Download Video

The system downloads the YouTube video using PyTube.

```text
YouTube URL
     ↓
PyTube
     ↓
MP4 Video
```

Video metadata is also collected:

```text
Title
Description
Length
Views
Author
Publish Date
```

---

### 2. Extract Video Frames

MoviePy is used to extract frames from the video.

Example:

```text
Video
 ↓
Frame 0001
Frame 0002
Frame 0003
...
```

The intended extraction rate is approximately:

```text
0.2 FPS
```

which corresponds to approximately one frame every 5 seconds.

---

### 3. Extract Audio

The video's audio track is extracted:

```text
input_video.mp4
       ↓
output_audio.wav
```

---

### 4. Speech-to-Text

The extracted audio is processed using Whisper:

```text
Audio
  ↓
Whisper
  ↓
Transcript
  ↓
output_text.txt
```

The transcript becomes the textual information available to the RAG pipeline.

---

## 📦 Installation

Clone the repository:

```bash
git clone <your-repository-url>
cd <your-project-folder>
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install llama-index-vector-stores-lancedb
pip install llama-index-multi-modal-llms-openai
pip install llama-index-embeddings-clip
pip install llama-index-readers-file
pip install llama-index
pip install -U openai-whisper
pip install lancedb
pip install moviepy
pip install pytube
pip install pydub
pip install SpeechRecognition
pip install ffmpeg-python
pip install soundfile
pip install torch torchvision
pip install matplotlib
pip install scikit-image
pip install ftfy
pip install regex
pip install tqdm
```

---

## 🔑 API Configuration

The multimodal LLM requires an API key.

Do **not** hard-code API keys inside your source code.

Instead, use an environment variable.

Example:

```env
OPENAI_API_KEY=your_api_key_here
```

Then load it from Python.

Never commit `.env` files containing real API keys to GitHub.

Add this to `.gitignore`:

```gitignore
.env
.venv/
__pycache__/
*.pyc
data/
```

---

## 📁 Project Structure

A recommended structure for this project is:

```text
multimodal-video-rag/
│
├── data/
│   ├── video_data/
│   │   └── input_video.mp4
│   │
│   └── mixed_data/
│       ├── frame0001.png
│       ├── frame0002.png
│       ├── output_text.txt
│       └── ...
│
├── notebooks/
│   └── multimodal_video_rag.ipynb
│
├── src/
│   ├── ingestion/
│   │   ├── youtube.py
│   │   └── video_processing.py
│   │
│   ├── extraction/
│   │   ├── audio.py
│   │   ├── frames.py
│   │   └── transcription.py
│   │
│   ├── retrieval/
│   │   └── retriever.py
│   │
│   └── generation/
│       └── multimodal_llm.py
│
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

---

## 💬 Example Query

After indexing the video, a user can ask:

```text
What is the main topic of the video?
```

The system performs:

```text
Question
   ↓
Retriever
   ↓
Relevant text
+
Relevant images
+
Video metadata
   ↓
Multimodal LLM
   ↓
Answer
```

---

## 🧪 Example Use Cases

This architecture can be extended for:

### Educational Videos

```text
"What concepts are explained in this lecture?"
```

### Technical Tutorials

```text
"What code is shown around the database section?"
```

### YouTube Research

```text
"What are the main arguments made in this video?"
```

### Video Search

```text
"Show me the part where the speaker explains transformers."
```

### Visual Question Answering

```text
"What diagram is shown when the speaker explains attention?"
```

---

## 🛠️ Current Limitations

The current implementation is a prototype and still requires several improvements before being considered production-ready.

### Current issues to address

* Explicit CLIP embedding configuration
* Correct LlamaIndex imports/API usage
* Correct LanceDB configuration
* Consistent video filename/path handling
* Proper frame extraction invocation
* Better error handling
* Environment-variable API configuration
* Metadata management
* Chunking of transcripts
* Better retrieval evaluation
* Reranking
* Hybrid keyword + vector retrieval
* Production database configuration
* Efficient video processing
* Duplicate frame filtering

---

## 🔮 Future Improvements

The system can be upgraded into a more advanced multimodal RAG architecture.

### 1. Hybrid Retrieval

Combine:

```text
BM25 / Keyword Search
        +
Dense Vector Search
```

to improve retrieval accuracy.

---

### 2. Reranking

Add a reranker after initial retrieval:

```text
Query
 ↓
Initial Retrieval
 ↓
Top 20
 ↓
Reranker
 ↓
Top 5
 ↓
LLM
```

---

### 3. Better Multimodal Embeddings

Experiment with modern multimodal embedding models instead of relying only on the initial CLIP setup.

---

### 4. Temporal Video Retrieval

Store timestamps with every frame:

```text
frame_001
timestamp = 00:01:20

frame_002
timestamp = 00:01:25
```

Then the system can answer:

> "At what point does the speaker explain RAG?"

with a timestamp.

---

### 5. Multimodal Chunking

Instead of treating text and images independently:

```text
Text Chunk
+
Image
+
Timestamp
+
Audio Segment
```

can be stored as a unified multimodal retrieval unit.

---

### 6. Evaluation

Add retrieval and generation metrics such as:

```text
Recall@K
Precision@K
MRR
Answer Relevance
Faithfulness
Context Relevance
```

This would make the project much closer to an **industry-level RAG system**.

---

## 🏗️ Final Architecture

The long-term architecture is:

```text
                     VIDEO
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
           Frames    Audio    Metadata
             │         │
             │       Whisper
             │         │
             │         ▼
             │       Text
             │         │
             └────┬────┘
                  │
                  ▼
          Multimodal Embeddings
                  │
                  ▼
               LanceDB
                  │
                  ▼
          ┌───────────────┐
          │   Retrieval   │
          └───────┬───────┘
                  │
          ┌───────┴────────┐
          ▼                ▼
      Text Results     Image Results
          │                │
          └───────┬────────┘
                  ▼
           Multimodal LLM
                  │
                  ▼
             Final Answer
```

---

## 🎯 Project Goal

The goal of this project is to build a **multimodal RAG system capable of understanding and retrieving information from video content**, rather than limiting retrieval to traditional text documents.

The system combines:

**Video Processing + Speech-to-Text + Image Understanding + Vector Retrieval + Multimodal Generation**

to create a question-answering system over video content.

---

## 📌 Project Status

**Status:** 🚧 Prototype / Under Development

The core architecture is established, but the implementation still requires debugging and modernization of several LlamaIndex, LanceDB, embedding, and multimodal API components before it can be considered production-ready.
