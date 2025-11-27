---
title: Finalizing and Saving Trace Data
---
This document describes how trace data is finalized and saved after a tracing session. Ongoing trace data is captured, the session is stopped, and the trace is saved to a file. The user is then notified that the trace file is ready for analysis.

# Finalizing and Saving Trace Data

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Flush all trace events"]
    click node1 openCode "examples/sdk/example_custom_data_source.cc:84:85"
    node1 --> node2["Stop tracing session"]
    click node2 openCode "examples/sdk/example_custom_data_source.cc:88:88"
    node2 --> node3["Collect trace data"]
    click node3 openCode "examples/sdk/example_custom_data_source.cc:89:89"
    node3 --> node4["Save trace data to file example_custom_data_source.pftrace"]
    click node4 openCode "examples/sdk/example_custom_data_source.cc:96:98"
    node4 --> node5["Notify user: Trace file example_custom_data_source.pftrace is ready. Analyze with traceconv tool"]
    click node5 openCode "examples/sdk/example_custom_data_source.cc:99:102"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Flush all trace events"]
%%     click node1 openCode "<SwmPath>[examples/sdk/example_custom_data_source.cc](examples/sdk/example_custom_data_source.cc)</SwmPath>:84:85"
%%     node1 --> node2["Stop tracing session"]
%%     click node2 openCode "<SwmPath>[examples/sdk/example_custom_data_source.cc](examples/sdk/example_custom_data_source.cc)</SwmPath>:88:88"
%%     node2 --> node3["Collect trace data"]
%%     click node3 openCode "<SwmPath>[examples/sdk/example_custom_data_source.cc](examples/sdk/example_custom_data_source.cc)</SwmPath>:89:89"
%%     node3 --> node4["Save trace data to file <SwmToken path="examples/sdk/example_custom_data_source.cc" pos="95:11:13" line-data="  const char* filename = &quot;example_custom_data_source.pftrace&quot;;">`example_custom_data_source.pftrace`</SwmToken>"]
%%     click node4 openCode "<SwmPath>[examples/sdk/example_custom_data_source.cc](examples/sdk/example_custom_data_source.cc)</SwmPath>:96:98"
%%     node4 --> node5["Notify user: Trace file <SwmToken path="examples/sdk/example_custom_data_source.cc" pos="95:11:13" line-data="  const char* filename = &quot;example_custom_data_source.pftrace&quot;;">`example_custom_data_source.pftrace`</SwmToken> is ready. Analyze with traceconv tool"]
%%     click node5 openCode "<SwmPath>[examples/sdk/example_custom_data_source.cc](examples/sdk/example_custom_data_source.cc)</SwmPath>:99:102"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/examples/sdk/example_custom_data_source.cc" line="82">

---

<SwmToken path="examples/sdk/example_custom_data_source.cc" pos="82:2:2" line-data="void StopTracing(std::unique_ptr&lt;perfetto::TracingSession&gt; tracing_session) {">`StopTracing`</SwmToken> flushes all trace events, stops the session, reads the trace, and writes it to a fixed file. The flush step is Perfetto-specific and ensures nothing is left in buffers.

```c++
void StopTracing(std::unique_ptr<perfetto::TracingSession> tracing_session) {
  // Flush to make sure the last written event ends up in the trace.
  CustomDataSource::Trace(
      [](CustomDataSource::TraceContext ctx) { ctx.Flush(); });

  // Stop tracing and read the trace data.
  tracing_session->StopBlocking();
  std::vector<char> trace_data(tracing_session->ReadTraceBlocking());

  // Write the result into a file.
  // Note: To save memory with longer traces, you can tell Perfetto to write
  // directly into a file by passing a file descriptor into Setup() above.
  std::ofstream output;
  const char* filename = "example_custom_data_source.pftrace";
  output.open(filename, std::ios::out | std::ios::binary);
  output.write(&trace_data[0], static_cast<std::streamsize>(trace_data.size()));
  output.close();
  PERFETTO_LOG(
      "Trace written in %s file. To read this trace in "
      "text form, run `./tools/traceconv text %s`",
      filename, filename);
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
