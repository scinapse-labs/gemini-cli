# Observability with OpenTelemetry

Gemini CLI provides built-in support for OpenTelemetry, transforming every agent
interaction into a rich stream of logs, metrics, and traces. This three-pillar
approach gives you the high-fidelity visibility needed to understand agent
behavior, optimize performance, and ensure reliability across your entire
workflow.

Whether you are debugging a complex tool interaction locally or monitoring
enterprise-wide usage in the cloud, Gemini CLI's observability system provides
the actionable intelligence needed to move from "black box" AI to predictable,
high-performance systems.

## OpenTelemetry integration

Gemini CLI integrates with **[OpenTelemetry]**, a vendor-neutral,
industry-standard observability framework.

The observability system provides:

- **Universal compatibility**: Export to any OpenTelemetry backend (Google
  Cloud, Jaeger, Prometheus, Datadog, etc.).
- **Standardized data**: Use consistent formats and collection methods across
  your toolchain.
- **Future-proof integration**: Connect with existing and future observability
  infrastructure.
- **No vendor lock-in**: Switch between backends without changing your
  instrumentation.

[OpenTelemetry]: https://opentelemetry.io/

## Configuration

You control telemetry behavior through the `.gemini/settings.json` file.
Environment variables can override these settings.

| Setting        | Environment Variable             | Description                                         | Values            | Default                 |
| -------------- | -------------------------------- | --------------------------------------------------- | ----------------- | ----------------------- |
| `enabled`      | `GEMINI_TELEMETRY_ENABLED`       | Enable or disable telemetry                         | `true`/`false`    | `false`                 |
| `target`       | `GEMINI_TELEMETRY_TARGET`        | Where to send telemetry data                        | `"gcp"`/`"local"` | `"local"`               |
| `otlpEndpoint` | `GEMINI_TELEMETRY_OTLP_ENDPOINT` | OTLP collector endpoint                             | URL string        | `http://localhost:4317` |
| `otlpProtocol` | `GEMINI_TELEMETRY_OTLP_PROTOCOL` | OTLP transport protocol                             | `"grpc"`/`"http"` | `"grpc"`                |
| `outfile`      | `GEMINI_TELEMETRY_OUTFILE`       | Save telemetry to file (overrides `otlpEndpoint`)   | file path         | -                       |
| `logPrompts`   | `GEMINI_TELEMETRY_LOG_PROMPTS`   | Include prompts in telemetry logs                   | `true`/`false`    | `true`                  |
| `useCollector` | `GEMINI_TELEMETRY_USE_COLLECTOR` | Use external OTLP collector (advanced)              | `true`/`false`    | `false`                 |
| `useCliAuth`   | `GEMINI_TELEMETRY_USE_CLI_AUTH`  | Use CLI credentials for telemetry (GCP target only) | `true`/`false`    | `false`                 |

**Note on boolean environment variables:** For boolean settings like `enabled`,
setting the environment variable to `true` or `1` enables the feature.

For detailed configuration information, see the
[Configuration guide](../reference/configuration.md).

## Google Cloud telemetry

You can export telemetry data directly to Google Cloud Trace, Cloud Monitoring,
and Cloud Logging.

### Prerequisites

You must complete several setup steps before enabling Google Cloud telemetry.

1. Set your Google Cloud project ID:
   - To send telemetry to a separate project:

     **macOS/Linux**

     ```bash
     export OTLP_GOOGLE_CLOUD_PROJECT="your-telemetry-project-id"
     ```

     **Windows (PowerShell)**

     ```powershell
     $env:OTLP_GOOGLE_CLOUD_PROJECT="your-telemetry-project-id"
     ```

   - To send telemetry to the same project as inference:

     **macOS/Linux**

     ```bash
     export GOOGLE_CLOUD_PROJECT="your-project-id"
     ```

     **Windows (PowerShell)**

     ```powershell
     $env:GOOGLE_CLOUD_PROJECT="your-project-id"
     ```

2. Authenticate with Google Cloud using one of these methods:
   - **Method A: Application Default Credentials (ADC)**: Use this method for
     service accounts or standard `gcloud` authentication.
     - For user accounts:
       ```bash
       gcloud auth application-default login
       ```
     - For service accounts:

       **macOS/Linux**

       ```bash
       export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/service-account.json"
       ```

       **Windows (PowerShell)**

       ```powershell
       $env:GOOGLE_APPLICATION_CREDENTIALS="C:\path\to\your\service-account.json"
       ```

   - **Method B: CLI Auth** (Direct export only): This is the simplest method
     for local users. It uses the same OAuth credentials as Gemini CLI. Set
     `useCliAuth: true` in your configuration. See
     [Authenticate with CLI credentials](#authenticate-with-cli-credentials) for
     details.

3. Ensure your account or service account has these IAM roles:
   - Cloud Trace Agent
   - Monitoring Metric Writer
   - Logs Writer

4. Enable the required Google Cloud APIs:
   ```bash
   gcloud services enable \
     cloudtrace.googleapis.com \
     monitoring.googleapis.com \
     logging.googleapis.com \
     --project="$OTLP_GOOGLE_CLOUD_PROJECT"
   ```

### Authenticate with CLI credentials

By default, the Google Cloud telemetry collector uses Application Default
Credentials (ADC). You can configure it to use the same OAuth credentials that
you use to log in to Gemini CLI. This is useful when you don't have ADC setup.

To enable this, set the `useCliAuth` property in your `telemetry` settings:

```json
{
  "telemetry": {
    "enabled": true,
    "target": "gcp",
    "useCliAuth": true
  }
}
```

> **Note:** This setting requires **Direct export** (in-process exporters).

Gemini CLI automatically uses your credentials to authenticate with Google Cloud
Trace, Metrics, and Logging APIs.

### Direct export

We recommend using direct export to send telemetry directly to Google Cloud
services.

1. Enable telemetry in `.gemini/settings.json`:
   ```json
   {
     "telemetry": {
       "enabled": true,
       "target": "gcp"
     }
   }
   ```
2. Run Gemini CLI and send prompts.
3. View logs, metrics, and traces in the Google Cloud Console. See
   [View Google Cloud telemetry](#view-google-cloud-telemetry) for details.

### View Google Cloud telemetry

After you enable telemetry and run Gemini CLI, you can view your data in the
Google Cloud Console.

- **Logs:** [Logs Explorer](https://console.cloud.google.com/logs/)
- **Metrics:**
  [Metrics Explorer](https://console.cloud.google.com/monitoring/metrics-explorer)
- **Traces:** [Trace Explorer](https://console.cloud.google.com/traces/list)

For detailed information on how to use these tools, see the following official
Google Cloud documentation:

- [View and analyze logs with Logs Explorer](https://cloud.google.com/logging/docs/view/logs-explorer-interface)
- [Create charts with Metrics Explorer](https://cloud.google.com/monitoring/charts/metrics-explorer)
- [Find and explore traces](https://cloud.google.com/trace/docs/finding-traces)

#### Monitoring dashboards

Gemini CLI provides a pre-configured
[Google Cloud Monitoring](https://cloud.google.com/monitoring) dashboard to
visualize your telemetry.

Find this dashboard under **Google Cloud Monitoring Dashboard Templates** as
"**Gemini CLI Monitoring**".

![Gemini CLI Monitoring Dashboard Overview](/docs/assets/monitoring-dashboard-overview.png)

To learn more, see
[Instant insights: Gemini CLI’s pre-configured monitoring dashboards](https://cloud.google.com/blog/topics/developers-practitioners/instant-insights-gemini-clis-new-pre-configured-monitoring-dashboards/).

## Local telemetry

You can capture telemetry data locally for development and debugging.

We recommend using file-based output for local development:

1. Enable telemetry in `.gemini/settings.json`:
   ```json
   {
     "telemetry": {
       "enabled": true,
       "target": "local",
       "outfile": ".gemini/telemetry.log"
     }
   }
   ```
2. Run Gemini CLI and send prompts.
3. View logs and metrics in `.gemini/telemetry.log`.

For advanced local telemetry setups (such as Jaeger or Genkit), see the
[Local development guide](../local-development.md#viewing-traces).

## Logs, metrics, and traces

This section describes the structure of logs, metrics, and traces generated by
Gemini CLI.

Gemini CLI includes `session.id`, `installation.id`, `active_approval_mode`, and
`user.email` (when authenticated) as common attributes on all data.

### Logs

Logs provide timestamped records of specific events. Gemini CLI logs events
across several categories.

#### Sessions

Session logs capture startup configuration and prompt submissions.

- `gemini_cli.config`: Emitted at startup with the CLI configuration.
- `gemini_cli.user_prompt`: Emitted when you submit a prompt.
  - **Attributes**: `prompt_length`, `prompt_id`, `prompt`, `auth_type`.

#### Approval mode

These logs track changes to and usage of different approval modes.

##### Lifecycle

- `approval_mode_switch`: Logs when you change the approval mode.
- `approval_mode_duration`: Records time spent in an approval mode.

##### Execution

- `plan_execution`: Logs when you execute a plan and switch from plan mode to
  active execution.

#### Tools

Tool logs capture executions, truncation, and edit behavior.

- `gemini_cli.tool_call`: Emitted for each tool (function) call.
- `gemini_cli.tool_output_truncated`: Logs when tool output is truncated.
- `gemini_cli.edit_strategy`: Records the chosen edit strategy.
- `gemini_cli.edit_correction`: Records the result of an edit correction.
- `gen_ai.client.inference.operation.details`: Provides detailed GenAI operation
  data aligned with OpenTelemetry conventions.

#### Files

File logs track operations performed by tools.

- `gemini_cli.file_operation`: Emitted for each file creation, read, or update.

#### API

API logs capture requests, responses, and errors from Gemini API.

- `gemini_cli.api_request`: Request sent to Gemini API.
- `gemini_cli.api_response`: Response received from Gemini API.
- `gemini_cli.api_error`: Logs when an API request fails.
- `gemini_cli.malformed_json_response`: Logs when a JSON response cannot be
  parsed.

#### Model routing

These logs track how Gemini CLI selects and routes requests to models.

- `gemini_cli.slash_command`: Logs slash command execution.
- `gemini_cli.slash_command.model`: Logs model selection via slash command.
- `gemini_cli.model_routing`: Records model router decisions and reasoning.

#### Chat and streaming

These logs track chat context compression and streaming chunk errors.

- `gemini_cli.chat_compression`: Logs chat context compression events.
- `gemini_cli.chat.invalid_chunk`: Logs invalid chunks received in a stream.
- `gemini_cli.chat.content_retry`: Logs retries due to content errors.
- `gemini_cli.chat.content_retry_failure`: Logs when all content retries fail.
- `gemini_cli.conversation_finished`: Logs when a conversation session ends.

#### Resilience

Resilience logs record fallback mechanisms and recovery attempts.

- `gemini_cli.flash_fallback`: Logs switch to a flash model fallback.
- `gemini_cli.ripgrep_fallback`: Logs fallback to standard grep.
- `gemini_cli.web_fetch_fallback_attempt`: Logs web-fetch fallback attempts.
- `gemini_cli.agent.recovery_attempt`: Logs attempts to recover from agent
  errors.

#### Extensions

Extension logs track lifecycle events and settings changes.

- `gemini_cli.extension_install`: Logs when you install an extension.
- `gemini_cli.extension_uninstall`: Logs when you uninstall an extension.
- `gemini_cli.extension_enable`: Logs when you enable an extension.
- `gemini_cli.extension_disable`: Logs when you disable an extension.

#### Agent runs

Agent logs track the lifecycle of agent executions.

- `gemini_cli.agent.start`: Logs when an agent run begins.
- `gemini_cli.agent.finish`: Logs when an agent run completes.

#### IDE

IDE logs capture connectivity events for the IDE companion.

- `gemini_cli.ide_connection`: Logs IDE companion connections.

#### UI

UI logs track terminal rendering issues.

- `kitty_sequence_overflow`: Logs terminal control sequence overflows.

#### Miscellaneous

- `gemini_cli.rewind`: Logs when the conversation state is rewound.
- `gemini_cli.conseca.verdict`: Logs security verdicts from ConSeca.
- `gemini_cli.hook_call`: Logs execution of lifecycle hooks.

### Metrics

Metrics provide numerical measurements of behavior over time.

#### Custom metrics

Gemini CLI exports several custom metrics.

##### Sessions

- `gemini_cli.session.count`: Incremented once per CLI startup.

##### Tools

- `gemini_cli.tool.call.count`: Counts tool calls.
- `gemini_cli.tool.call.latency`: Measures tool call latency (in ms).

##### API

- `gemini_cli.api.request.count`: Counts all API requests.
- `gemini_cli.api.request.latency`: Measures API request latency (in ms).

##### Token usage

- `gemini_cli.token.usage`: Counts input, output, thought, cache, and tool
  tokens.

##### Files

- `gemini_cli.file.operation.count`: Counts file operations.
- `gemini_cli.lines.changed`: Counts added or removed lines.

##### Chat and streaming

- `gemini_cli.chat_compression`: Counts compression operations.
- `gemini_cli.chat.invalid_chunk.count`: Counts invalid stream chunks.
- `gemini_cli.chat.content_retry.count`: Counts content error retries.
- `gemini_cli.chat.content_retry_failure.count`: Counts requests where all
  retries failed.

##### Model routing

- `gemini_cli.slash_command.model.call_count`: Counts model selections.
- `gemini_cli.model_routing.latency`: Measures routing decision latency.
- `gemini_cli.model_routing.failure.count`: Counts routing failures.

##### Agent runs

- `gemini_cli.agent.run.count`: Counts agent runs.
- `gemini_cli.agent.duration`: Measures agent run duration.
- `gemini_cli.agent.turns`: Counts turns per agent run.

##### Approval mode

- `gemini_cli.plan.execution.count`: Counts plan executions.

##### UI

- `gemini_cli.ui.flicker.count`: Counts terminal flicker events.

##### Performance

Gemini CLI provides detailed performance metrics for advanced monitoring.

- `gemini_cli.startup.duration`: Measures startup time by phase.
- `gemini_cli.memory.usage`: Measures heap and RSS memory.
- `gemini_cli.cpu.usage`: Measures CPU usage percentage.
- `gemini_cli.tool.queue.depth`: Measures tool execution queue depth.
- `gemini_cli.tool.execution.breakdown`: Breaks down tool time by phase.

#### GenAI semantic convention

These metrics follow standard [OpenTelemetry GenAI semantic conventions].

- `gen_ai.client.token.usage`: Counts tokens used per operation.
- `gen_ai.client.operation.duration`: Measures operation duration in seconds.

[OpenTelemetry GenAI semantic conventions]:
  https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-metrics.md

### Traces

Traces provide an "under-the-hood" view of agent and backend operations. Use
traces to debug tool interactions and optimize performance.

Every trace captures rich metadata via standard span attributes:

- `gen_ai.operation.name`: High-level operation (for example, `tool_call`,
  `llm_call`, `user_prompt`, `system_prompt`, `agent_call`, or
  `schedule_tool_calls`).
- `gen_ai.agent.name`: Set to `gemini-cli`.
- `gen_ai.agent.description`: The service agent description.
- `gen_ai.input.messages`: Input data or metadata.
- `gen_ai.output.messages`: Output data or results.
- `gen_ai.request.model`: Request model name.
- `gen_ai.response.model`: Response model name.
- `gen_ai.prompt.name`: The prompt name.
- `gen_ai.tool.name`: Executed tool name.
- `gen_ai.tool.call_id`: Unique ID for the tool call.
- `gen_ai.tool.description`: Tool description.
- `gen_ai.tool.definitions`: Tool definitions in JSON format.
- `gen_ai.usage.input_tokens`: Number of input tokens.
- `gen_ai.usage.output_tokens`: Number of output tokens.
- `gen_ai.system_instructions`: System instructions in JSON format.
- `gen_ai.conversation.id`: The CLI session ID.

For more details on semantic conventions for events, see the
[OpenTelemetry documentation](https://github.com/open-telemetry/semantic-conventions/blob/8b4f210f43136e57c1f6f47292eb6d38e3bf30bb/docs/gen-ai/gen-ai-events.md).
