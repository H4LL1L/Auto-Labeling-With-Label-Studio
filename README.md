# Label Studio ML Backend — Automated Labeling

Accelerate annotation by connecting Label Studio to machine learning models for one‑click pre‑labeling. Use off‑the‑shelf examples or plug in your own trained model to pre‑annotate videos, images, and text; then collaborate in the UI to review and refine.

- Faster iterations: turn minutes into seconds with auto‑labeling
- Collaborative workflow: review, correct, and discuss in one place
- Open source: extend freely; no vendor lock‑in
- Extensible: add, train, and deploy your own models

## How It Works

- The ML backend runs a web service exposing `/predict`.
- Label Studio sends tasks to the model URL; predictions return as pre‑annotations in the UI.
- Optional training loop via webhooks: implement `fit()` to update the model as labels arrive.
- Modes:
  - Pre‑annotation
  - Interactive inference
  - Training (webhook‑driven)
 
    
      1- We begin by importing our videos.
<p align="center"><img src="image1.png" alt=""/></p>
<p align="center"><img src="image2.png" alt=""/></p>
       2- Then we just click on the video
<p align="center"><img src="image3.png" alt=""/></p>
<p align="center"><img src="image4.png" alt=""/></p>
      When we click, there is an ML model in the background that automatically labels the video. We use our own trained model to make our specific labels, but with         this method we can automatically label everything from a mug to a person.
<p align="center"><img src="image5.png" alt=""/></p>
<p align="center"><img src="image6.png" alt=""/></p>
      The first version of the video is as above, after clicking, these are the labels that it shows us with one click.
<p align="center"><img src="image7.png" alt=""/></p>
      We can also use these opensource (except for watsonx_llm) models and we can write our own ML model backend. 

      
## Quick Start

Start an example model (sklearn text classifier) with Docker and connect it to Label Studio.

1. ML Backend (9090)

```bash
git clone https://github.com/HumanSignal/label-studio-ml-backend.git
cd label-studio-ml-backend/label_studio_ml/examples/sklearn_text_classifier
docker-compose up
# Service: http://localhost:9090
# Health:  curl http://localhost:9090/
```

2. Label Studio (8080)

```bash
pip install -U label-studio
label-studio start -p 8080
# UI: http://localhost:8080
```

3. Connect the model

- Project > Settings > Model > Add Model
- URL: `http://localhost:9090`
- From Docker to host (Mac/Windows): `http://host.docker.internal:9090`

Note: For training or downloading assets via API, set:

- `LABEL_STUDIO_HOST` (e.g., http://localhost:8080)
- `LABEL_STUDIO_API_KEY` (from your user settings)

## Run Without Docker (Development)

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# Start example backend (9090)
python label_studio_ml/examples/sklearn_text_classifier/_wsgi.py -p 9090
```

## API Overview

- Health: `GET /` or `/health` → `{"status":"UP","model_class":"..."}`
- Predict: `POST /predict`

  - Request (minimal):
    ```json
    {
      "tasks": [{ "data": { "text": "great movie!" } }],
      "label_config": "<View>...</View>",
      "project": "1.1700000000",
      "params": {}
    }
    ```
  - Response: `{"results": [ ... Label Studio prediction format ... ]}`

- Training/Webhook: `POST /webhook`
  - Events: `ANNOTATION_CREATED`, `ANNOTATION_UPDATED`, `ANNOTATION_DELETED`, `START_TRAINING`
  - Implement `fit(event, data)` in your model to handle updates

## Add Your Own Model

1. Scaffold

```bash
label-studio-ml init my_ml_backend --script path/to/your_model.py:YourModelClass
```

2. Implement core methods

- `predict(tasks, **kwargs)`: return predictions in LS format
- `fit(event, data, **kwargs)`: update the model as labels arrive (optional)

3. Run

```bash
label-studio-ml start my_ml_backend -p 9090
```

## Example Models

- NLP: `sklearn_text_classifier`, `bert_classifier`, `huggingface_ner`, `gliner`
- OCR: `easyocr`, `tesseract`
- Vision: `yolo`, `segment_anything_model`, `segment_anything_2_image`, `grounding_dino`, `grounding_sam`
- LLM: `huggingface_llm`, `watsonx_llm`
- Other: `flair`, `nemo_asr` …

All under `label_studio_ml/examples/*`, each with `docker-compose.yml` and `_wsgi.py`.

## Environment Variables (Common)

- `LABEL_STUDIO_HOST`: Label Studio host used for training data downloads (e.g., http://localhost:8080)
- `LABEL_STUDIO_API_KEY`: API key for Label Studio (required for training)
- `BASIC_AUTH_USER`, `BASIC_AUTH_PASS`: Basic auth for the model server
- `LOG_LEVEL`, `WORKERS`, `THREADS`: Server logging and worker settings

## Security

- Don’t expose the backend directly to the internet; use `BASIC_AUTH` if needed.
- Keep secrets in `.env`; never commit API keys to Git.

## Why This Approach

- Speed: pre‑labeling drastically reduces manual effort
- Teamwork: collaborate on review and corrections in a single UI
- Freedom: fully open‑source; models are extensible
- Scale: label more data faster to iterate models quickly

> Performance numbers are indicative and depend on data and workload.

## Requirements

- Python 3.8+
- (Recommended) Docker & Docker Compose
- Network access between Label Studio and the ML backend

## License

LABEL STUDİO MIT
