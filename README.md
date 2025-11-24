# Agentic AI Root Cause Observability

## Introductio

This project explores how agent-based AI systems can perform automated root cause localization in distributed applications by leveraging observability data such as logs, metrics, and traces collected through the OpenTelemetry standard. Using Google’s Agent Development Kit (ADK) together with the OpenTelemetry demo application, the goal is to design an AI agent capable of retrieving telemetry, integrating multi-modal signals, and reasoning about system behavior to propose likely root causes for IT incidents.

The research emphasizes several core components:

- **Multi-modal telemetry fusion:** combining logs, metrics, and traces into a unified context to improve diagnostic accuracy.
- **Service dependency graph analysis:** using trace relationships to understand request flows and identify where failures originate.
- **Benchmarking and comparison:** evaluating the agentic approach against traditional correlation-based or rule-based baselines.
- **Fault injection experiments:** deliberately introducing errors into the system to generate realistic failure scenarios and measure the agent’s root-cause ranking performance.
- **Robustness analysis:** assessing how well the agent performs under noisy, incomplete, or partially missing telemetry—common in real-world monitoring systems.

This document outlines a conceptual framework that explains how telemetry is produced, propagated, stored, and correlated using trace IDs, how dependency graphs emerge from spans, and how an agent built using Google’s ADK can construct contextual prompts for an LLM to perform incident analysis. It also highlights architectural considerations, the challenges of multi-modal data fusion, secure and scalable data management, and a rigorous evaluation strategy. Together, these components demonstrate how agentic AI can meaningfully improve the speed, accuracy, and reliability of root cause analysis in complex distributed environments.
## Observability with OpenTelemetry in Distributed Systems

### Overview
OpenTelemetry is an open-source observability framework that provides a standardized way to collect, process, and export telemetry data (traces, metrics, and logs) from distributed systems. It enables developers to gain insights into the performance and behavior of their applications, making it easier to diagnose issues and optimize performance.

### Architecture Components
![OpenTelemetry Logo](images/Obs_Otel_Architecture.png)
### Logging Mechanism
1. 1. **When a request enters the system, the first service that receives it (the entrypoint service) generates a unique trace_id.**
    
    This trace_id is propagated to all downstream services to correlate their spans under the same distributed trace.
    
2. **Each service creates a span** and logs relevant telemetry, including:
    - trace_id
    - span_id
    - parent_id
    - timestamps
    - status (OK, ERROR)
    - attributes and events
3. **Each service exports its telemetry asynchronously to an OpenTelemetry Collector**, which processes and forwards the data to a storage backend for analysis.

### Telemetry Data Types
1. **Traces**: Represent end-to-end execution of a request. A trace consists of multiple spans, one per operation or service call.
2. **Metrics**: Numbers that represent system performance over time:
latency, CPU usage, memory, throughput, error rate, etc.
3. **Logs**: Unstructured or semi-structured text containing details about events, errors, warnings, or checkpoints.
...

### Sample Trace Data
According to the OpenTelemetry documentation, a sample trace data in JSON format looks like this:
```json
{
  "name": "hello-greetings",
  "context": {
    "trace_id": "5b8aa5a2d2c872e8321cf37308d69df2",
    "span_id": "5fb397be34d26b51"
  },
  "parent_id": "051581bf3cb55c13",
  "start_time": "2022-04-29T18:52:58.114304Z",
  "end_time": "2022-04-29T22:52:58.114561Z",
  "attributes": {
    "http.route": "some_route2"
  },
  "events": [
    {
      "name": "hey there!",
      "timestamp": "2022-04-29T18:52:58.114561Z",
      "attributes": {
        "event_attributes": 1
      }
    },
    {
      "name": "bye now!",
      "timestamp": "2022-04-29T18:52:58.114585Z",
      "attributes": {
        "event_attributes": 1
      }
    }
  ]
}
```

### OpenTelemetry Database
Database should be able to store large volumes of telemetry data with high availability and scalability.
Database should support efficient querying and analysis of telemetry data.


## Discover root cause of an Incident (by Human)
1. **Follow the trace ID** to reconstruct the full request path across all services and identify where delays or errors occurred.
2. **Query the telemetry backend** for all logs, metrics, and spans associated with the trace ID and timestamps.
3. **Build the service dependency graph** using parent–child span relationships to visualize how the request propagated.
4. **Analyze the combined telemetry** to determine which service produced the first failure, why it happened, and how it affected downstream services.

## Discover the root cause of an Incident (by AI Agent)
1. The AI agent **uses the trace ID** to retrieve all related telemetry using its tools (logs, metrics, traces, span relationships).
2. The agent **constructs a contextual prompt** that includes:
    - the trace ID
    - logs linked to the failing spans
    - relevant metrics (latency spikes, error counts, resource usage)
    - the reconstructed service dependency graph
    - any detected anomalies or errors
3. The agent **sends this prompt to the LLM** through Google ADK.
4. The LLM processes the prompt and produces a structured root-cause explanation and possible remediation steps.
5. The agent **evaluates and refines the output** (e.g., checking for hallucinations or inconsistencies).
6. The agent **formats the final analysis** into a human-readable summary.
7. The agent **presents the result to operators**, supporting faster detection, diagnosis, and resolution of incidents.
A simple prompt template could be:
   ```shell
    Agent Role: You are an AI agent specialized in analyzing distributed system telemetry data.
    Task: Investigate the root cause of an incident using the provided trace ID.
    Trace ID: <trace_id>
    Logs: <logs associated with trace_id>
    Metrics: <metrics associated with trace_id>
    Traces: <traces associated with trace_id>
    Service Dependency Graph: <service dependency graph> 
    Output Format: Provide a detailed analysis of the root cause, affected services, and recommended actions to resolve the incident.
   ```

## Agentic AI Architectural Components
![Agentic AI Architecture](images/Obs_Agentic_AI.png)

The architecture consists of three main layers: the **observability pipeline**, the **agentic RCA system**, and the **LLM reasoning engine**.

### **1. Observability Pipeline**

Distributed services instrumented with OpenTelemetry generate **logs**, **metrics**, and **traces**.

This telemetry is sent to the **OpenTelemetry Collector**, which receives, processes, and exports the data to a scalable telemetry database.

All telemetry is correlated using a shared **trace_id**.

### **2. Incident Detection and Triggering**

When an incident occurs (e.g., an error or latency spike), an incident detection system forwards an alert to the Agentic AI component, including the **trace_id** of the failing request.

### **3. Agentic AI**

The agent uses its tools to:

- query logs, metrics, and traces associated with the trace_id
- reconstruct the service dependency graph from span relationships
- identify anomalies in latency or error status
- prepare a structured prompt that includes all relevant telemetry

The agent then submits this contextual prompt to the LLM via Google’s Agent Development Kit (ADK).

### **4. LLM Reasoning and Response**

The LLM analyzes the multi-modal telemetry, identifies the most likely root cause, explains how the issue propagated across services, and recommends remediation actions.

The agent validates the response, formats it for human consumption, and sends it back to the incident analysis system.

## Google ADK
The Google Agent Development Kit (ADK) provides a framework for building intelligent, tool-using agents that can retrieve data, perform reasoning, and interact with large language models. In this architecture, ADK is used to implement an RCA (Root Cause Analysis) agent that queries telemetry, constructs prompts, and performs LLM-driven analysis.

A pseudo-code example of an agent using Google ADK:
```python
START RootCauseAgent

FUNCTION run_analysis(trace_id):

    # 1. Collect telemetry
    telemetry = tools.fetch_telemetry(trace_id)
        # → logs, metrics, traces

    # 2. Build Service Dependency Graph
    graph = tools.build_service_graph(telemetry.traces)

    # 3. Create prompt for LLM
    prompt = """
        You are a root-cause analysis AI.
        Trace ID: {trace_id}
        Logs: {logs}
        Metrics: {metrics}
        Traces: {traces}
        Service Graph: {graph}
        Task: Identify root cause and recommend fixes.
    """

    # 4. Send prompt to the LLM (Gemini via ADK)
    llm_response = llm.generate_content(prompt)

    # 5. Format results for human operators
    result = tools.format_response(llm_response)

    return result

END FUNCTION

```

## Agentic AI Challenges and Considerations
1. Get logs, metrics, and traces associated with trace IDs efficiently.
2. Discovering relevant telemetry data from large datasets based on trace IDs.
3. Context management to ensure the AI agent has sufficient information for analysis.
4. Handling multi-modal data (logs, metrics, traces, dashboards, images, and ...) effectively.
5. Ensuring the accuracy and reliability of the AI agent's analysis.
6. Evaluating the performance of the AI agent against traditional root cause analysis methods.
7. Dealing with noisy or incomplete telemetry data.
8. Fine-tuning the LLM for domain-specific knowledge in distributed systems and observability.

## Security and Privacy Considerations
1. Ensure that sensitive telemetry data is handled securely and in compliance with privacy regulations.
2. Following GDPR and other relevant data protection laws when processing telemetry data.
3. Encrypting sensitive data during transmission and storage.

## Evaluation Strategy
1. Conduct controlled fault-injection experiments to create incidents in the distributed system.
2. Measure the accuracy of root cause ranking by comparing the AI agent's analysis with known root causes.
3. Analyze the robustness of the AI agent under noisy or incomplete telemetry data.
4. Compare the performance of the agentic approach against simpler correlation-based or rule-based baselines.
5. Collect feedback from human operators on the usefulness and clarity of the AI agent's analysis.

## Conclusion
This project investigates how agent-based AI systems can automate root cause localization in distributed systems by leveraging multi-modal observability data collected through OpenTelemetry. The proposed architecture demonstrates how an AI agent can retrieve telemetry, construct contextual reasoning prompts, and collaborate with a large language model to identify failures, highlight affected services, and recommend corrective actions.

By benchmarking the agentic approach against traditional rule-based and correlation-driven methods, the research aims to show improvements in detection accuracy, robustness under incomplete telemetry, and overall time-to-resolution for incidents. The findings highlight the potential of combining observability standards with advanced AI reasoning to significantly enhance operational reliability in complex, large-scale systems.

