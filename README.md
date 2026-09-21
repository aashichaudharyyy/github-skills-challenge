# AIOps Assessment

## 1. AIOps scenario
This project monitors a payment service that handles customer transactions. The service records response time, CPU usage, memory usage, and log messages so the team can detect slowdowns and failures early. The purpose of the assessment is to show a basic AIOps workflow: operational data is analysed, unusual behaviour is detected, an event is created, and the event moves through a simple producer-topic-consumer pipeline.

## 2. Operational data description
The repository contains synthetic service telemetry in [data/service_data.json](data/service_data.json). Each record is a single observation with:
- timestamp
- service
- response_time_ms
- cpu_percent
- memory_percent
- log_level
- message

The data is a simple time series with one-minute intervals, starting at 2026-09-20T10:00:00 and ending at 2026-09-20T10:09:00.

### Metrics
The metric fields are:
- response_time_ms
- cpu_percent
- memory_percent

### Log information
The log-related fields are:
- service
- log_level
- message

## 3. Observations from logs and metrics
### Normal behaviour
Normal records show:
- response_time_ms around 120-150 ms
- cpu_percent around 42-50%
- memory_percent around 51-57%
- log_level = INFO
- message = Payment request processed successfully

These records show the service is working normally.

### Unusual behaviour
The unusual records are at 10:05 and 10:06:
- response_time_ms jumps to 610 and 640 ms
- cpu_percent rises to 75% and 94%
- memory_percent rises to 70% and 91%
- log_level changes to ERROR
- message shows timeout errors

This is a clear sign of service degradation or failure. The log messages match the metrics and show that the service is timing out under load.

## 4. Anomaly-detection findings
The detection logic flags a record when one or more of the following occurs:
- response time is above the threshold
- CPU use is above the threshold
- memory use is above the threshold
- an ERROR log is present

The abnormal records at 10:05 and 10:06 are detected as anomalies. The reasons include high response time, high CPU use, high memory use, and error log detection. The normal records are not flagged.

## 5. Event-processing flow
The project uses a simple AIOps workflow:

Operational data -> anomaly detection -> event -> producer -> topic -> consumer -> AIOps output

In this simulation:
- the anomaly detector checks each record
- if a record is abnormal, it creates an event
- the producer sends the event to an in-memory topic
- the consumer receives the event from that topic
- the downstream AIOps component records the processed issue

## 6. Result of the final workflow execution
The validation logic in the project expects the following output:
- records processed: 10
- anomalies detected: 2
- events consumed: 2

These two anomalies correspond to the timeout and database connection failures seen in the service telemetry.

## 7. Issues identified and corrected
The detector originally checked for WARNING logs, but the data uses ERROR logs.
The condition was corrected to detect ERROR.

The producer and consumer originally used different EventTopic objects. The
consumer therefore received zero events. Both components were changed to use
the same anomaly-events topic.

## 8. Limitation and possible improvement
This AIOps approach is rule-based and uses fixed thresholds. It works well for this small dataset, but it may miss new patterns or create false alarms when the system changes over time. A better version would use dynamic baselines or trend-based thresholds instead of fixed values.

# 9. Steps required 
python3 -m json.tool data/service_data.json
python3 src/aiops_pipeline.py
pyton3 -m pytest -q
python3 src/aiops_pipeline.py