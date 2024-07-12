# audio-to-audio-translator
Translator audio from one language to another

# Speech-to-Text and Text-to-Speech Web Service

This repository provides a web service that converts speech to text using Hugging Face's Whisper model and then converts text back to speech using the coqui XTTS-v2 model. The web service is built using Flask and can be accessed publicly via ngrok.

## Features

- **Automatic Speech Recognition (ASR):** Converts speech to text using the Whisper model.
- **Text-to-Speech (TTS):** Converts text to speech using the coqui XTTS-v2 model.
- **Web Service:** Provides endpoints to handle audio files and return synthesized speech.

## Prerequisites

- Python 3.7 or later
- pip (Python package installer)

## Installation

1. **Clone the repository:**
    ```bash
    git clone https://github.com/yourusername/speech-to-text-to-speech.git
    cd speech-to-text-to-speech
    ```

2. **Install the required libraries:**
    ```bash
    pip install git+https://github.com/huggingface/transformers
    pip install TTS
    pip install pyngrok
    pip install flask
    ```

3. **Download an example audio file (optional):**
    ```bash
    wget -O audio.mp3 http://www.moviesoundclips.net/movies1/darkknight/criminal.mp3
    ```

## Usage

1. **Initialize Whisper and XTTS-v2 Models:**

    ```python
    from transformers import pipeline
    import torch
    from TTS.api import TTS

    # Check if CUDA is available and set the device accordingly
    device = 0 if torch.cuda.is_available() else -1

    whisper = pipeline('automatic-speech-recognition', model='openai/whisper-medium', device=device)

    # Initialize TTS
    tts = TTS("tts_models/multilingual/multi-dataset/xtts_v2", gpu=False)
    ```

2. **Set up Flask and ngrok:**

    ```python
    from pyngrok import ngrok
    from flask import Flask, request, send_file

    ngrok.set_auth_token("your-ngrok-auth-token")

    app = Flask(__name__)

    @app.route('/', methods=['GET'])
    def index():
        return "Hello, World!"

    @app.route('/generate', methods=['POST'])
    def generate():
        audio_file = request.files['audio']
        ref_audio = request.files['ref_audio']
        audio_file.save(audio_file.filename)
        ref_audio.save(ref_audio.filename)

        text = whisper(audio_file.filename)
        st = text["text"]

        tts.tts_to_file(text=st, file_path="latest_output.wav", speaker_wav=ref_audio.filename, language="en")
        return send_file("latest_output.wav", mimetype="audio/wav", as_attachment=True)

    if __name__ == '__main__':
        public_url = ngrok.connect(5000).public_url
        print("Public URL:", public_url)
        app.run()
    ```

3. **Run the Flask Application:**
    ```bash
    python app.py
    ```

4. **Access the Web Service:**
    The public URL will be displayed in the terminal. Use this URL to access the web service and test the endpoints.

## Endpoints

- **GET /**: Returns a simple "Hello, World!" message.
- **POST /generate**: Accepts two files - `audio` (the input audio file for ASR) and `ref_audio` (a reference audio file for TTS). Returns a synthesized speech audio file.

## Example Usage

To use the `/generate` endpoint, you can use a tool like `curl` or Postman:

```bash
curl -X POST -F "audio=@audio.mp3" -F "ref_audio=@ref_audio.wav" http://<your-ngrok-url>/generate --output latest_output.wav
