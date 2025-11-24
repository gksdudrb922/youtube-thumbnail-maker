# YouTube Thumbnail Maker

An AI-powered system that analyzes video files to automatically generate
thumbnail designs optimized for increasing view counts, then refines them into
final versions based on user selection and feedback.

## Overview

This project uses LangGraph to orchestrate a multi-step workflow that:

1. Extracts and processes audio from video files
2. Transcribes the audio using OpenAI Whisper
3. Generates comprehensive summaries of the video content
4. Creates multiple thumbnail concept designs using AI
5. Collects user feedback and selection
6. Generates a high-quality final thumbnail incorporating user preferences

## Features

- **Automatic Audio Extraction**: Extracts audio from video files with speed
  optimization
- **AI-Powered Transcription**: Uses OpenAI Whisper for accurate
  speech-to-text conversion
- **Intelligent Summarization**: Chunk-based processing with comprehensive final
  summary generation
- **Multiple Thumbnail Concepts**: Generates 5 different thumbnail design
  concepts in parallel
- **User Feedback Integration**: Interactive workflow that pauses for user
  input
- **High-Quality Final Output**: Produces professional-grade thumbnails
  optimized for YouTube

## Architecture

The system is built using LangGraph's state machine pattern, with the
following workflow:

```text
START
  ↓
Extract Audio (FFmpeg)
  ↓
Transcribe Audio (OpenAI Whisper)
  ↓
Dispatch Summarizers (Parallel chunk processing)
  ↓
Mega Summary (Combine all summaries)
  ↓
Dispatch Artists (Generate 5 thumbnail concepts in parallel)
  ↓
Human Feedback (Interactive selection)
  ↓
Generate HD Thumbnail (Final refinement)
  ↓
END
```

## Installation

### Prerequisites

- Python 3.13 or higher
- FFmpeg (for audio extraction)
- OpenAI API key

### Setup

1. Clone the repository:

```bash
git clone <repository-url>
cd youtube-thumbnail-maker
```

1. Install dependencies using `uv`:

```bash
uv sync
```

1. Create a `.env` file in the project root with your OpenAI API key:

```env
OPENAI_API_KEY=your_api_key_here
```

1. Ensure FFmpeg is installed and available in your PATH:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

## Usage

### Running the Graph

The main workflow is defined in `graph.py`. You can run it using LangGraph CLI
or programmatically.

#### Using LangGraph CLI

1. Start the LangGraph server:

```bash
langgraph dev
```

1. The graph will be available at the configured endpoint.

#### Using Python/Jupyter

See `main.ipynb` for an example of running the graph interactively:

```python
from graph import graph

config = {"configurable": {"thread_id": 1}}

# Start the workflow
result = graph.invoke(
    {"video_file": "space.mp4"},
    config=config
)

# When prompted, provide feedback
response = {
    "user_feedback": "Give it a photo realistic, 3d style.",
    "chosen_prompt": 2,
}

# Resume with user feedback
graph.invoke(Command(resume=response), config=config)
```

### Workflow Steps

1. **Input**: Provide a video file path (e.g., `space.mp4`)
2. **Audio Extraction**: The system extracts audio and speeds it up 2x for
   faster processing
3. **Transcription**: Audio is transcribed using OpenAI Whisper
4. **Summarization**: Transcription is chunked and summarized in parallel,
   then combined
5. **Thumbnail Generation**: 5 thumbnail concepts are generated based on the
   summary
6. **User Selection**: The workflow pauses for you to:
   - Select your preferred thumbnail (1-5)
   - Provide feedback for refinement
7. **Final Generation**: A high-quality thumbnail is generated incorporating
   your feedback

### Output Files

- `thumbnail_1.jpg` through `thumbnail_5.jpg`: Initial concept thumbnails
- `thumbnail_final.jpg`: Final high-quality thumbnail
- `*.mp3`: Extracted audio file (same name as video, with .mp3 extension)

## Dependencies

### Core Dependencies

- `langchain[openai]>=1.0.7`: LLM integration and chat models
- `langgraph>=1.0.3`: State machine and workflow orchestration
- `langgraph-cli[inmem]>=0.4.7`: CLI tools and in-memory checkpointing
- `langsmith>=0.4.43`: LangChain observability
- `openai[aiohttp]>=2.8.1`: OpenAI API client
- `python-dotenv>=1.2.1`: Environment variable management
- `tiktoken>=0.12.0`: Token counting utilities

### Development Dependencies

- `ipykernel>=7.1.0`: Jupyter notebook support

## Project Structure

```text
youtube-thumbnail-maker/
├── graph.py              # Main LangGraph workflow definition
├── langgraph.json        # LangGraph configuration
├── main.ipynb            # Example usage in Jupyter notebook
├── pyproject.toml         # Project dependencies and metadata
├── README.md             # This file
└── .env                  # Environment variables (create this)
```

## How It Works

### State Management

The workflow uses a `State` TypedDict that tracks:

- `video_file`: Input video file path
- `audio_file`: Extracted audio file path
- `transcription`: Full text transcription
- `summaries`: List of chunk summaries (accumulated)
- `thumbnail_prompts`: List of thumbnail design prompts
- `thumbnail_sketches`: List of generated thumbnail file paths
- `final_summary`: Combined comprehensive summary
- `user_feedback`: User's refinement feedback
- `chosen_prompt`: Selected thumbnail prompt

### Key Functions

- **`extract_audio`**: Uses FFmpeg to extract and speed up audio
- **`transcribe_audio`**: OpenAI Whisper transcription with domain-specific
  prompts
- **`dispatch_summarizer`**: Splits transcription into 500-character chunks
  for parallel processing
- **`summarize_chunk`**: Summarizes individual chunks using GPT-4o-mini
- **`mega_summary`**: Combines all chunk summaries into a comprehensive
  overview
- **`dispatch_artists`**: Launches 5 parallel thumbnail generation tasks
- **`generate_thumbnails`**: Creates thumbnail concepts using GPT image
  generation
- **`human_feedback`**: Interactive node that pauses for user input
- **`generate_hd_thumbnail`**: Produces final high-quality thumbnail with user
  feedback

### Parallel Processing

The workflow leverages LangGraph's `Send` mechanism for parallel execution:

- Multiple summarization chunks are processed simultaneously
- Five thumbnail concepts are generated in parallel
- This significantly reduces total processing time

## Configuration

### LangGraph Configuration

The `langgraph.json` file configures:

- Graph dependencies
- Graph definitions
- Environment file location

### Environment Variables

Required in `.env`:

- `OPENAI_API_KEY`: Your OpenAI API key

## Notes

- The system uses GPT-4o-mini for text processing and GPT image generation for
  thumbnails
- Audio is sped up 2x during extraction to reduce transcription time
- Transcription includes domain-specific prompts for better accuracy
- Initial thumbnails are generated with `quality="low"` for speed
- Final thumbnail uses `quality="high"` for production-ready output
- The workflow uses in-memory checkpointing for state persistence

## License

This project is licensed under the terms of the MIT license.
