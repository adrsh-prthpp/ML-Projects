# Bengaluru House Price Prediction API

A compact Flask inference service that estimates Bengaluru home prices from
location, square footage, bedroom count, and bathroom count using a persisted
scikit-learn model.

## Features

- Loads a trained model and encoded feature metadata
- Lists supported locations
- Exposes price estimation through a Flask API
- Keeps inference logic separate from HTTP routing

## Architecture

```text
HTTP request -> Flask route -> input normalization
             -> feature vector -> persisted model -> predicted price
```

## Tech stack

- Python
- Flask
- NumPy
- scikit-learn
- Gunicorn-compatible deployment

## Repository structure

```text
Model/              Model and feature metadata
Server/
  server.py         Flask routes
  util.py           Artifact loading and prediction
  artifacts/        Runtime copies of model metadata
```

## Installation

```bash
cd Server
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
python server.py
```

## Usage

Call the location endpoint to discover supported categories, then submit
`total_sqft`, `location`, `bhk`, and `bath` to the estimation endpoint. Consult
`Server/server.py` for the exact local routes and form-field names.

## Project status

**Functional inference prototype.** Python syntax validation passes. The
repository does not currently include the training notebook, raw dataset,
automated tests, API schema, or deployment configuration, so earlier
end-to-end-training claims cannot be reproduced from this repository alone.

## Screenshot / demo

Add a short API demo using curl or Postman and a response generated from
non-sensitive sample inputs.

## Future improvements

- Add the reproducible training pipeline and dataset provenance
- Add request validation and consistent JSON error responses
- Add unit tests and model-loading failure tests
- Remove duplicated model artifacts
- Add Docker and an OpenAPI-compatible API layer
- Document evaluation metrics from a held-out set
