# SAR Wind Field Prediction API 

An end-to-end Machine Learning pipeline and serverless API that generates high-resolution ocean wind field maps from Sentinel-1 Synthetic Aperture Radar (SAR) imagery.

By leveraging Google Earth Engine (GEE) and a custom ResNet architecture, this API can dynamically fetch, process, and map meteorological wind data for coastal regions on the fly.

---

## Features

- **Serverless GEE Streaming:** Bypasses hard-drive bottlenecks by streaming heavy 10m-resolution SAR tiles directly into Python memory via the Earth Engine API.
- **Smart Regional Scanning:** Automatically searches a predefined bounding box and time-shifts to find the nearest satellite pass with optimal sea-to-land ratios.
- **Physics-Corrected Preprocessing:** Dynamically converts GEE Decibel (dB) data to Linear scale to match the neural network's training distribution.
- **Adaptive Vector Mapping:** Generates meteorological-grade quiver plots using physical gradient calculations to dynamically cluster wind arrows in turbulent zones while keeping uniform regions clean.

---

##  Architecture

### 1. Data Pipeline (`data_preprocessing.py`)

- Processes Sentinel-1 SAR imagery.
- Slices large `.tif` files into overlapping 49×49 patches.
- Performs land masking and NaN removal.
- Generates training-ready NumPy arrays.

### 2. Deep Learning Model (`train_resnet.py`)

- Implements a custom **M64RN4 Residual Network**.
- Trained using a custom angular loss function:

```python
1.0 - cos(y_pred - y_true)
```

- Handles the cyclical nature of 360° wind direction prediction.

### 3. Production API (`api.py`)

- FastAPI-based microservice.
- Integrates the trained model with live Google Earth Engine data retrieval.
- Produces real-time wind field predictions and visualizations.

---

##  Installation

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/sar-wind-field-api.git
cd sar-wind-field-api
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Download Model Weights

Due to GitHub storage limitations, trained model weights are not stored in the repository.

Create a `models/` directory and place your trained model inside:

```text
models/
└── best_resnet_model.keras
```

---

##  Data Requirements

This repository does not include raw satellite imagery or ERA5 weather datasets due to their large size.

To train the model or run the complete pipeline, organise your data as follows:

```text
sar-wind-field-api/
├── data/
│   ├── raw/                # Sentinel-1 .tif files
│   └── processed/          # Generated .npy patches
├── models/
│   └── best_resnet_model.keras
├── api.py
├── train_resnet.py
├── data_preprocessing.py
└── era5_gujarat_wind.nc
```

---

##  Usage

### 1. Authenticate Google Earth Engine

Before launching the API, authenticate your local environment with Google Earth Engine:

```bash
earthengine authenticate
```

### 2. Start the API Server

Launch the FastAPI application:

```bash
python api.py
```

Alternatively, if using Uvicorn:

```bash
uvicorn api:app --host 0.0.0.0 --port 8000
```

### 3. Generate a Wind Field Map

Open the interactive Swagger documentation:

```text
http://localhost:8000/docs
```

Send a POST request to the `/predict_regional` endpoint:

```json
{
  "date": "2021-02-14"
}
```

The API will return a dynamically generated high-resolution `.png` map of the predicted ocean wind field.

---

##  Repository Structure

```text
sar-wind-field-api/
├── api.py
├── train_resnet.py
├── data_preprocessing.py
├── requirements.txt
├── README.md
├── validation_report.csv
├── image_1.png
├── image2.png
├── CEC_2_ATHARVA_24113027.pdf

```

---

##  Project Philosophy

This repository focuses on the **code and methodology** behind SAR-based wind field prediction rather than the underlying datasets.

By keeping large satellite imagery, weather datasets, and model weights outside version control, the repository remains:

- Lightweight
- Professional
- Easy to clone
- Reproducible
- Suitable for open-source collaboration

The logic lives in the repository, while the data payload remains securely stored in external storage or cloud infrastructure.

---

## License

This project is intended for research and educational purposes. Please ensure compliance with Sentinel-1, Google Earth Engine, and ERA5 data usage policies when deploying or redistributing derived products
