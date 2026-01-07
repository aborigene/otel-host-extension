# OpenTelemetry Dynatrace Extension

A Dynatrace extension that enables comprehensive monitoring of OpenTelemetry host metrics with custom topology mapping, automated alerting, and intelligent entity relationships.

## Overview

This extension enhances Dynatrace's observability capabilities by ingesting and processing OpenTelemetry host and process metrics. It creates a custom topology model that maps OpenTelemetry data to Dynatrace entities, establishing meaningful relationships between hosts and processes while providing automated alerting for critical system events.

## Features

### 🗺️ Custom Topology Mapping

The extension defines two primary entity types that model your OpenTelemetry infrastructure:

#### **Host Entities (`otel:host2`)**
- **Unique Identification**: Hosts are identified using a composite pattern: `otel_host_{host.ip}-{host.id}-{host.name}`
- **Automatic Discovery**: Detects hosts from metrics prefixed with `system.` from the OpenTelemetry hostmetrics receiver
- **Rich Attributes**:
  - Operating System type
  - Host ID and architecture
  - IP addresses
  - Security context
- **Visual Representation**: Custom host icon in Dynatrace UI

#### **Process Entities (`otel:process2`)**
- **Unique Identification**: Processes are identified by: `otel_process_{host.ip}-{host.id}-{host.name}-{process.command_line}`
- **Automatic Discovery**: Detects processes from metrics prefixed with `process.` from the OpenTelemetry hostmetrics receiver
- **Rich Attributes**:
  - Process command line
  - Process ID (PID)
  - Executable name
  - Security context
- **Visual Representation**: Custom process icon in Dynatrace UI

### 🔗 Entity Relationships

The extension establishes intelligent relationships between entities:

1. **Process-to-Host**: `RUNS_ON` relationship connecting processes to their host machines
2. **Host Unification**: `SAME_AS` relationship linking OpenTelemetry hosts to native Dynatrace host entities for unified monitoring

### 📊 Monitored Metrics

The extension specifically tracks:

- **`system.uptime`**: Monitors system uptime to detect reboots and availability issues
- **`system.cpu.load_average.1m`**: Tracks 1-minute CPU load average
- **`system.cpu.utilization`**: Monitors CPU utilization percentage
- All host metrics from the OpenTelemetry hostmetrics receiver
- All process metrics from the OpenTelemetry hostmetrics receiver

### 🚨 Pre-configured Alerts

Three intelligent alert configurations are included:

#### 1. **Host Reboot Detection** (`alert-001-host-uptime-reset.json`)
- **Metric**: `system.uptime`
- **Condition**: Triggers when uptime drops BELOW 250 seconds
- **Detection**: 3 violating samples out of 5
- **Purpose**: Automatically detects system reboots by identifying sudden drops in uptime
- **Event Type**: Custom Alert

#### 2. **High CPU Load** (`alert-002-high-cpu-load.json`)
- **Metric**: `system.cpu.load_average.1m`
- **Condition**: Triggers when load average is ABOVE 0.5
- **Detection**: 3 violating samples out of 5
- **Purpose**: Identifies sustained high CPU load that may impact performance
- **Event Type**: Custom Alert

#### 3. **High CPU Utilization** (`alert-003-host-cpu-usage.json`)
- **Metric**: `system.cpu.utilization`
- **Condition**: Triggers when CPU utilization exceeds 80%
- **Detection**: 3 violating samples out of 5
- **Purpose**: Monitors CPU saturation to prevent resource exhaustion
- **Event Type**: Custom Alert

All alerts include:
- 5-sample monitoring window
- 5-sample dealerting threshold to reduce alert flapping
- Dimension-based alerting per host entity
- Disabled alerting on missing data to avoid false positives

### 📋 Alert Template

The extension also includes an alert template demonstrating static threshold anomaly detection:

- **Analyzer**: `dt.statistics.ui.anomaly_detection.StaticThresholdAnomalyDetectionAnalyzer`
- **Query**: Uses DQL (Dynatrace Query Language) to aggregate metrics by host entity
- **Configurable**: Threshold, alert condition, and violating samples can be customized
- **Event Generation**: Creates availability events with custom names and descriptions

## Requirements

- **Dynatrace version**: 1.327.0 or higher
- **OpenTelemetry Collector**: Configured with the hostmetrics receiver
- **Required Dimensions**: Metrics must include `otel.scope.name` dimension identifying the hostmetrics receiver

## Installation

1. Download the extension package (version 0.0.24)
2. Upload to your Dynatrace environment via the Hub
3. Activate the extension
4. Configure your OpenTelemetry Collector to send metrics to Dynatrace

## Configuration

### OpenTelemetry Collector Setup

Ensure your OpenTelemetry Collector configuration includes:

```yaml
receivers:
  hostmetrics:
    scrapers:
      cpu:
      memory:
      disk:
      filesystem:
      load:
      network:
      process:

exporters:
  otlphttp:
    endpoint: "https://{your-dynatrace-environment}/api/v2/otlp"
    headers:
      Authorization: "Api-Token {your-token}"

service:
  pipelines:
    metrics:
      receivers: [hostmetrics]
      exporters: [otlphttp]
```

### Required Metric Attributes

Metrics should include these attributes for proper entity identification:

- `host.name`: Hostname for display
- `host.id`: Unique host identifier
- `host.ip`: IP address
- `os.type`: Operating system type
- `host.arch`: System architecture
- `process.command_line`: Full process command (for processes)
- `process.pid`: Process ID (for processes)
- `process.executable.name`: Executable name (for processes)
- `otel.scope.name`: Must contain `github.com/open-telemetry/opentelemetry-collector-contrib/receiver/hostmetricsreceiver`

## Usage

Once installed and configured:

1. **Topology Visualization**: Navigate to "Smartscape" to view your OpenTelemetry hosts and processes
2. **Metric Analysis**: Access host metrics through the entity pages or Metrics browser
3. **Alert Management**: Review and customize the pre-configured alerts in Settings > Anomaly Detection > Custom Events
4. **Relationship Exploration**: Use Dynatrace's topology view to explore process-to-host relationships

## Extension Structure

```
opentelemetry-extension/
├── extension/
│   ├── extension.yaml          # Main extension configuration
│   └── alerts/
│       ├── alert-001-host-uptime-reset.json
│       ├── alert-002-high-cpu-load.json
│       └── alert-003-host-cpu-usage.json
├── config/                     # Configuration files (if needed)
└── dist/                       # Built extension package
```

## Customization

### Adjusting Alert Thresholds

Edit the JSON files in the `alerts/` directory to modify:
- `threshold`: The numeric value that triggers alerts
- `violatingSamples`: Number of samples that must breach the threshold
- `samples`: Total monitoring window size
- `dealertingSamples`: Number of normal samples required to clear the alert
- `alertCondition`: "ABOVE" or "BELOW"

### Extending Topology

Modify [extension/extension.yaml](extension/extension.yaml) to:
- Add new entity types
- Define additional relationships
- Include more attributes
- Add metric metadata

### Creating New Alerts

Use the alert template structure in [extension/extension.yaml](extension/extension.yaml) as a reference to create additional alert definitions.

## Troubleshooting

### Entities Not Appearing

1. Verify OpenTelemetry Collector is sending metrics to Dynatrace
2. Check that metrics include the required `otel.scope.name` dimension
3. Ensure all required attributes are present in the metric data
4. Verify extension is activated in Dynatrace Hub

### Alerts Not Triggering

1. Confirm metrics are being ingested (check Metrics browser)
2. Verify alert thresholds are appropriate for your environment
3. Check that the entity dimension exists in the metric data
4. Review alert configuration for the correct metric ID

### Relationship Issues

1. Ensure both host and process metrics are being collected
2. Verify the dimension keys match between related entities
3. Check that the hostmetrics receiver is properly configured in OpenTelemetry

## Version

**Current Version**: 0.0.24

## Author

**Dynatrace**

## License

Refer to your Dynatrace licensing agreement for usage terms.

## Support

For issues and questions:
- Check Dynatrace documentation
- Review OpenTelemetry Collector logs
- Contact Dynatrace support with extension details

## Contributing

Contributions and suggestions are welcome. Please ensure:
- Extension version is updated appropriately
- Alert configurations are tested
- Documentation is updated for new features
- Backward compatibility is maintained

---

**Note**: This extension is designed to work specifically with metrics from the OpenTelemetry hostmetrics receiver. Ensure your collector is configured correctly to emit the required dimensions and metric names.
