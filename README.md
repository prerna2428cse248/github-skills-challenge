# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## AIOps Scenario

This project monitors a synthetic `payment-service`. Its operational data
contains response-time, CPU, memory, and log telemetry. AIOps identifies
unusual behaviour and moves each anomaly through an event-processing pipeline:

```text
Operational data -> anomaly detection -> event -> producer -> topic -> consumer -> AIOps output
```

## Operational Data

The data is stored in [`data/service_data.json`](data/service_data.json).
Metrics are `response_time_ms`, `cpu_percent`, and `memory_percent`. Log
information is represented by `log_level` and `message`. Each observation has
an ISO-style `timestamp` and the `payment-service` name.

Records from 10:00 through 10:04 and 10:07 through 10:09 are normal. Their
response times are about 120-150 ms, CPU is below 50%, memory is below 60%,
and the log level is `INFO`.

The two anomalous observations are:

| Timestamp | Evidence |
| --- | --- |
| 2026-09-20T10:05:00 | 610 ms response time and an `ERROR` payment timeout log |
| 2026-09-20T10:06:00 | 640 ms response time, 94% CPU, 91% memory, and an `ERROR` database timeout log |

## Detection and Event Flow

[`src/anomaly_detector.py`](src/anomaly_detector.py) flags response time above
500 ms, CPU above 80%, memory above 80%, and `ERROR` log entries. It creates an
event containing the timestamp, service, type, reasons, and source record.

[`src/event_producer.py`](src/event_producer.py) publishes the event to the
in-memory [`EventTopic`](src/event_topic.py). The
[`EventConsumer`](src/event_consumer.py) reads from that same topic, and
[`src/aiops_pipeline.py`](src/aiops_pipeline.py) prints the processed events.

## Issues Corrected

The detector originally checked for `WARNING`, but the supplied data uses
`ERROR`. The detector now recognizes the relevant error logs.

The producer originally published to `service-events` while the consumer read
from a separate `anomaly-events` topic. The consumer now uses the producer's
topic, allowing both detected events to be consumed.

## Final Result

The corrected pipeline processes 10 records, detects 2 anomalies, publishes 2
events, consumes 2 events, and displays both payment-service incidents with
their timestamps and reasons.

Run the pipeline with:

```bash
python src/aiops_pipeline.py
```

## Validation Results

The test suite covers normal and anomalous records, all detector reasons,
producer and topic behaviour, event consumption, the complete pipeline, and
the command-line output path.

```bash
python -m pytest --cov=src --cov-report=term-missing --verbose
```

Final local result:

```text
16 passed
TOTAL 100%
```

## Reproduce

```bash
python -m venv .venv/calculations
source .venv/calculations/bin/activate
python -m pip install -r requirements.txt
python -m pip install pytest coverage pytest-cov
python src/aiops_pipeline.py
python -m pytest --cov=src --cov-report=term-missing --verbose
```

## Limitation

The detector uses fixed thresholds and does not learn a baseline from
historical behaviour. The topic is in memory, so events are lost when the
process stops. A production implementation could use adaptive thresholds and
a durable event broker.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

