
# ADR-0005: Isolation Forest for Initial Anomaly Detection

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI must identify unusual application behaviour using machine learning.

Application logs contain textual messages, timestamps, severity levels, service information and optional performance attributes.

Feeding arbitrary log messages directly into a numerical anomaly detection model would not provide a suitable initial feature representation.

The platform therefore needs a reproducible feature engineering and anomaly detection approach.

## Decision

Use the Isolation Forest algorithm from Scikit-learn as the initial anomaly detection model.

The model will analyze meaningful numerical features extracted from application telemetry over defined time windows.

The Python FastAPI ML service will own model training, persistence, metadata and inference.

The Java backend will coordinate anomaly analysis and integrate prediction results into application workflows.

## Feature Engineering

The initial model will use numerical features such as:

- Error rate per time window.
- Request count per time window.
- Average request duration.
- Maximum request duration.
- P95 request duration when sufficient observations exist.
- Error count.
- Warning count.
- Unique service count.
- Unique endpoint count.
- HTTP 4xx rate.
- HTTP 5xx rate.

Feature availability must be validated.

Missing values must be handled explicitly rather than silently interpreted as zero.

Arbitrary log message text will not be passed directly to Isolation Forest.

## Model Training

The training pipeline must:

1. Load documented training data.
2. Validate the input schema.
3. Handle missing and invalid values.
4. Generate numerical features.
5. Separate training and evaluation data.
6. Train the Isolation Forest model.
7. Evaluate the model.
8. Persist the trained artifact using Joblib.
9. Save model metadata and version information.

Training must be reproducible through documented instructions.

## Training Data

Training data may include clearly labelled synthetic telemetry representing normal and unusual application behaviour.

Synthetic data must be explicitly identified.

Synthetic evaluation results must not be presented as production accuracy.

The dataset generation process, assumptions and limitations must be documented.

## Prediction API

The FastAPI ML service will expose:

- `GET /health`
- `GET /model/info`
- `POST /predict`

Prediction responses should include:

- Anomaly score.
- Anomaly flag.
- Model version.
- Prediction timestamp.
- Input feature summary.

The service must document how its anomaly score is interpreted.

## Evaluation

The model must be evaluated using:

- Precision.
- Recall.
- F1 score.
- Confusion matrix.

Evaluation must use an appropriate held-out dataset.

No benchmark score or production accuracy claim may be fabricated.

The selected decision threshold and evaluation assumptions must be documented.

## Model Persistence and Versioning

The trained model will be stored as a versioned artifact using Joblib.

Model metadata should include:

- Model version.
- Algorithm name.
- Training timestamp.
- Feature schema version.
- Training dataset reference.
- Model configuration.
- Evaluation results.

Inference must use a known and compatible feature schema.

## Operational Behaviour

Machine learning inference must run asynchronously relative to log ingestion.

If the ML service is unavailable:

- Log ingestion must continue.
- Accepted log events must not be lost.
- Inference may be retried safely.
- Exhausted retries must follow the configured failure workflow.
- ML service health must be reported separately.
- An inference failure must never be treated as an anomaly.

## Alternatives Considered

### Fixed Threshold Rules Only

Not selected as the sole anomaly detection approach because the initial product requires a trained machine learning model.

Threshold-based rules will still be supported independently through alert management.

### Direct Text Classification

Not selected for the initial version because raw log messages require a separate text processing and labelled training strategy.

### Deep Learning Models

Not selected initially because they introduce additional training, infrastructure and operational complexity.

Isolation Forest provides a practical starting point for numerical telemetry features.

## Consequences

### Positive

- Practical unsupervised anomaly detection approach.
- Works with numerical telemetry features.
- Available through Scikit-learn.
- Relatively simple model training and persistence.
- Independent Python ML runtime.
- Clear separation between feature engineering and inference.

### Negative

- Model quality depends on feature quality.
- Anomaly results may contain false positives.
- Training data may not represent real production behaviour.
- Threshold selection requires evaluation.
- Concept drift may require future retraining.
- Isolation Forest does not explain root causes by itself.

## Implementation Notes

- Use Python and Scikit-learn.
- Implement feature validation with Pydantic schemas.
- Separate training and inference logic.
- Persist models with Joblib.
- Version model artifacts.
- Add Pytest coverage.
- Document score interpretation.
- Integrate with the Java backend through the ML API.
- Never block ingestion on inference.

## Review Conditions

Revisit this decision when:

- Sufficient real telemetry becomes available.
- Model evaluation reveals unacceptable performance.
- Feature distributions change significantly.
- Production feedback justifies another model.
- A more suitable anomaly detection strategy is demonstrated through measurable evaluation.
