# Sahayak AI - Anthropic

Sahayak AI is a multi-agent system designed to assist in educational environments. It provides tools for both school administrators ("Prabhandhak") and teachers ("Shikshak Mitra") to streamline tasks and create engaging learning materials. The platform leverages the power of Anthropic's Claude Sonnet 4.5 model and various open-source libraries to deliver a suite of intelligent features.

## Features

The application is built around distinct AI agents, each tailored for specific tasks:

### Prabhandhak (Administrator) Agent

- **Automated Attendance:** Takes attendance by analyzing a classroom photo. It uses face recognition to identify students and marks them as present or absent.
- **OCR-Powered Question Generation:** Upload an image of a textbook page, and the agent uses Optical Character Recognition (OCR) to extract the text. It then performs Retrieval-Augmented Generation (RAG) against a vector database of NCERT textbooks to generate relevant, high-quality educational questions in JSON format.

### Shikshak Mitra (Teacher's Assistant) Agent

- **Contextual Question Generation:** Given a topic, this agent retrieves relevant information from the NCERT knowledge base and generates a set of distinct questions, each focusing on a different concept.
- **AI-Powered Animation Generation:** Creates educational animations from a simple text prompt using Manim. The agent generates the necessary Python code, renders the video, and makes it available via an API, with an option to upload to Google Cloud Storage.

### Chat Agent

- A conversational AI assistant designed for students from Grade 1 to Grade 6. It can answer questions and act as a learning companion.

## Technology Stack

- **Backend:** FastAPI
- **AI Models:**
  - **LLM:** Anthropic Claude Sonnet 4.5
  - **Embeddings:** Google Vertex AI `text-embedding-004`
- **Vector Database:** Google Firestore Vector Store for RAG
- **Relational Database:** PostgreSQL
- **Face Recognition:** `face_recognition` library with `OpenCV`
- **Animation:** `Manim`
- **Deployment:** Docker, Google Cloud Storage (for video uploads)

## 📂 Project Structure

```
├── app/
│   ├── agents/         # Core logic for different AI agents (Chat, Manim, Prabhandhak, Shikshak Mitra)
│   ├── api/            # FastAPI routers and API endpoints
│   ├── core/           # Application configuration (settings)
│   ├── mcp/            # Manim Control Program (MCP) tools
│   ├── models/         # Pydantic models for API requests/responses
│   ├── services/       # Business logic (e.g., attendance service)
│   └── utils/          # Utility functions (e.g., GCP Storage uploader)
├── train/              # Contains training images for face recognition
├── docker-compose.yml  # Docker configuration for local PostgreSQL database
├── main.py             # FastAPI application entrypoint
├── requirements.txt    # Python dependencies
└── *.sql               # SQL dump for database schema and sample data
```

## 🚀 Getting Started

Follow these steps to set up and run the project locally.

### 1. Prerequisites

- Python 3.9+
- Docker and Docker Compose
- Git

### 2. Clone the Repository

```bash
git clone https://github.com/divyanshkul/Sahayak-AI-Anthropic.git
cd sahayak-ai-anthropic
```

### 3. Set Up Environment Variables

Create a `.env` file in the root directory and populate it with your credentials. See `app/core/config.py` for all possible variables.

```env
# Pragati Backend
APP_NAME="Sahayak AI"
DEBUG=True

# Anthropic/Claude configuration
ANTHROPIC_API_KEY="sk-ant-..."

# Manim configuration
# Path to your manim executable if not in system PATH
# MANIM_EXECUTABLE="/path/to/your/venv/bin/manim"

# GCP Storage configuration (for video uploads)
GCP_BUCKET_NAME="your-gcs-bucket-name"
# Path to your GCP service account JSON key file.
# Note: The agent files contain hardcoded paths which should be updated or managed via this .env setup.
GCP_CREDENTIALS_PATH="/path/to/your/gcp-credentials.json"
```

### 4. Install Dependencies

It is recommended to use a virtual environment.

```bash
python -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`

pip install -r requirements.txt

# The face_recognition_models library may require direct installation from GitHub
pip install git+https://github.com/ageitgey/face_recognition_models
```

### 5. Set Up the Database

Start the PostgreSQL database using Docker Compose.

```bash
docker-compose up -d
```

This will create a PostgreSQL container named `sahayak-postgres` running on port `5432`.

To populate the database with the provided schema and sample data, you can use a PostgreSQL client like `psql` or a GUI tool to import the `Cloud_SQL_Export_2025-08-14 (00_19_35) (1).sql` file.

```bash
# Example using psql
psql -h localhost -U postgres -p 5432 -d multigradeschool < "Cloud_SQL_Export_2025-08-14 (00_19_35) (1).sql"
# Password is 'AquaRegia'
```

### 6. Train Face Recognition

Place labeled images of students (e.g., `student_name.jpg`) in the `train/` directory. The `AttendanceService` will automatically load these on startup to build its known face encodings.

### 7. Run the Application

```bash
uvicorn main:app --host 0.0.0.0 --port 4002 --reload
```

The API will be available at `http://localhost:4002`. You can access the auto-generated documentation at `http://localhost:4002/docs`.

## 📖 API Endpoints

Here are some of the key API endpoints:

### Prabhandhak Agent

- `POST /api/v1/prabhandhak/attendance/upload-photo`

  - **Description:** Upload a classroom photo to take attendance.
  - **Form Data:** `photo` (image file), `class_id` (string).
  - **Returns:** JSON with attendance details and recognized students.

- `POST /api/v1/prabhandhak/ocr/process-image`
  - **Description:** Upload an image of a textbook page for OCR and question generation.
  - **Form Data:** `photo` (image file).
  - **Returns:** JSON containing the extracted text and a list of generated questions.

### Shikshak Mitra Agent

- `POST /api/v1/shikshak-mitra/generation-questions`

  - **Description:** Generate questions based on a text topic.
  - **Body:** `{ "question": "Generate questions about angles" }`
  - **Returns:** A structured JSON object with topics and questions.

- `POST /api/v1/shikshak-mitra/generate-animation`
  - **Description:** Generate a Manim animation from a text prompt.
  - **Body:** `{ "prompt": "Animate a circle transforming into a square" }`
  - **Returns:** JSON with metadata about the generation process, including the public URL if uploaded to GCS.

### Chat Agent

- `POST /api/v1/chat/`
  - **Description:** Chat with the student assistant.
  - **Body:** `{ "text": "What is a prism?" }`
  - **Returns:** The AI's response.
