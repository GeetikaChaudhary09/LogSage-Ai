
# LogSage AI — Machine Learning Pipeline

## 1. Overview

LogSage AI uses a dedicated Python machine learning service to identify unusual application behaviour from numerical telemetry features.

The initial anomaly detection model will use Scikit-learn Isolation Forest.

The ML service remains separate from the main Java backend.

**Status:** Planned. Model training, evaluation, inference and backend integration will be implemented and verified incrementally.

## 2. Technology Stack

| Technology | Responsibility |
|---|---|
| Python 3.14 | ML development |
| FastAPI | Inference API |
| Scikit-learn | Isolation Forest |
| NumPy | Numerical computation |
| Pandas | Dataset processing |
| Pydantic | Request and response validation |
| Joblib | Model persistence |
| Pytest | Automated testing |

Dependency versions will be selected and compatibility-tested before installation.

## 3. ML Architecture

The planned ML pipeline:

```text
Application Logs
       |
       v
Log Processing
       |
       v
Time Window Aggregation
       |
       v
Feature Engineering
       |
       v
Feature Validation
       |
       v
Isolation Forest Model
       |
       v
Anomaly Score and Prediction
       |
       v
Backend Anomaly Processing
       |
       v
Incident and Alert Evaluation
```

The model will process engineered numerical features rather than arbitrary raw log message text.

## 4. Anomaly Detection Model

Initial model:

**Isolation Forest**

Isolation Forest identifies unusual observations by isolating samples in a feature space.

The model is suitable as an initial unsupervised anomaly detection approach.

It does not automatically understand application semantics or guarantee that every flagged observation represents a real incident.

## 5. Feature Engineering

Initial candidate features:

| Feature | Description |
|---|---|
| error_rate | Proportion of error events in a time window |
| request_count | Total requests or request events |
| avg_duration_ms | Average request duration |
| max_duration_ms | Maximum request duration |
| p95_duration_ms | 95th percentile request duration |
| error_count | Number of error events |
| warn_count | Number of warning events |
| unique_services | Number of distinct services |
| unique_endpoints | Number of distinct endpoints |
| 4xx_rate | Proportion of HTTP 4xx responses |
| 5xx_rate | Proportion of HTTP 5xx responses |

Feature definitions, aggregation windows and missing-value policies must be documented consistently across training and inference.

## 6. Training Dataset

The initial training dataset will be reproducible and clearly documented.

Synthetic data may be used to develop the first model.

Synthetic examples may represent:

- Normal application traffic
- Increased error frequency
- Unusual request latency
- Sudden traffic spikes
- Abnormal service activity

Synthetic data must always be labelled as synthetic.

Training results from synthetic data must not be presented as production accuracy.

## 7. Training Pipeline

Planned training workflow:

1. Load the documented dataset.
2. Validate required columns and data types.
3. Handle missing and invalid values.
4. Apply consistent feature engineering.
5. Separate training and evaluation datasets.
6. Train the Isolation Forest model.
7. Evaluate model behaviour.
8. Persist the trained model.
9. Save model metadata.
10. Record the model version.

Training must be reproducible using documented commands and configuration.

## 8. Model Evaluation

Evaluation will include:

- Precision
- Recall
- F1 score
- Confusion matrix

Where labelled evaluation data is available, predictions will be compared with actual labels.

Because Isolation Forest is an unsupervised model, evaluation requires a separately labelled evaluation dataset or another explicitly documented validation method.

No benchmark score will be invented or assumed.

## 9. Model Persistence and Versioning

Joblib will be used for model persistence.

Model metadata should contain:

- Model name
- Model version
- Training timestamp
- Feature schema version
- Training dataset identifier
- Training configuration
- Evaluation results
- Scikit-learn version

The model and metadata must be loaded and validated safely at service startup.

## 10. ML API

The FastAPI service will expose:

### Health check

```http
GET /health
```

Returns service availability.

### Model information

```http
GET /model/info
```

Returns model metadata, version and feature information.

### Prediction

```http
POST /predict
Content-Type: application/json
```

A prediction request will contain validated numerical features.

The prediction response will contain:

- anomaly_score
- is_anomaly
- model_version
- predicted_at
- input_feature_summary

The score interpretation will be documented according to the actual model implementation.

## 11. Backend Integration

The Java backend will communicate with the ML service through its internal service interface.

The ML service will not directly modify the main application database.

The backend remains responsible for:

- Processing prediction results
- Persisting anomaly records
- Linking results to projects and supporting evidence
- Evaluating incident conditions
- Managing alert workflows

## 12. Failure Handling

If the ML service is unavailable:

- Log ingestion must continue when the ingestion infrastructure is operational.
- Events must not be silently discarded.
- ML processing must retry safely.
- Repeated failures must enter a dead-letter workflow.
- Service health must be displayed separately.
- Failed inference must never be interpreted as an anomaly.

Prediction processing must be idempotent where appropriate.

## 13. Model Limitations

Isolation Forest has limitations:

- Anomaly results depend on the quality of input features.
- Unusual does not necessarily mean harmful.
- Behaviour changes may require model retraining.
- Synthetic data cannot establish production accuracy.
- Poorly chosen time windows can hide important patterns.
- Model drift must be monitored.

Predictions should be presented as investigation signals, not definitive root-cause explanations.

## 14. Testing Strategy

Planned tests:

- Feature schema validation
- Missing-value handling
- Invalid input rejection
- Training pipeline tests
- Model loading tests
- Prediction endpoint tests
- Model metadata tests
- Reproducibility tests
- Backend integration tests
- ML service failure handling

## 15. Implementation Status

This document defines the planned ML pipeline.

The ML capability will be considered implemented only after:

1. Dataset and feature definitions are documented.
2. Training pipeline executes successfully.
3. Model is persisted and loaded successfully.
4. Evaluation is performed and reported honestly.
5. FastAPI endpoints work.
6. Automated tests pass.
7. Backend integration is verified.
8. Failure handling is tested.
