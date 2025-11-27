---
title: Finalizing and Saving Trace Data
---
This document describes how trace data is finalized and saved at the end of a tracing session. All trace events are captured, the data is saved to a file, and the user is notified that the trace file is ready for analysis.

# Finalizing and Saving Trace Data

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Flush trace events to ensure all data is captured"]
    click node1 openCode "examples/sdk/example_startup_trace.cc:86:87"
    node1 --> node2["Stop tracing and retrieve trace data"]
    click node2 openCode "examples/sdk/example_startup_trace.cc:90:91"
    node2 --> node3["Save trace data to file example_startup_trace.pftrace"]
    click node3 openCode "examples/sdk/example_startup_trace.cc:98:101"
    node3 --> node4["Notify user that trace file is ready"]
    click node4 openCode "examples/sdk/example_startup_trace.cc:102:105"
    node4 --> node5["Suggest reading trace in text form with provided tool"]
    click node5 openCode "examples/sdk/example_startup_trace.cc:103:105"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Flush trace events to ensure all data is captured"]
%%     click node1 openCode "<SwmPath>[examples/sdk/example_startup_trace.cc](examples/sdk/example_startup_trace.cc)</SwmPath>:86:87"
%%     node1 --> node2["Stop tracing and retrieve trace data"]
%%     click node2 openCode "<SwmPath>[examples/sdk/example_startup_trace.cc](examples/sdk/example_startup_trace.cc)</SwmPath>:90:91"
%%     node2 --> node3["Save trace data to file <SwmToken path="examples/sdk/example_startup_trace.cc" pos="97:11:13" line-data="  const char* filename = &quot;example_startup_trace.pftrace&quot;;">`example_startup_trace.pftrace`</SwmToken>"]
%%     click node3 openCode "<SwmPath>[examples/sdk/example_startup_trace.cc](examples/sdk/example_startup_trace.cc)</SwmPath>:98:101"
%%     node3 --> node4["Notify user that trace file is ready"]
%%     click node4 openCode "<SwmPath>[examples/sdk/example_startup_trace.cc](examples/sdk/example_startup_trace.cc)</SwmPath>:102:105"
%%     node4 --> node5["Suggest reading trace in text form with provided tool"]
%%     click node5 openCode "<SwmPath>[examples/sdk/example_startup_trace.cc](examples/sdk/example_startup_trace.cc)</SwmPath>:103:105"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/examples/sdk/example_startup_trace.cc" line="84">

---

<SwmToken path="examples/sdk/example_startup_trace.cc" pos="84:2:2" line-data="void StopTracing(std::unique_ptr&lt;perfetto::TracingSession&gt; tracing_session) {">`StopTracing`</SwmToken> kicks off by flushing any buffered trace events using Perfetto's <SwmToken path="examples/sdk/example_startup_trace.cc" pos="86:1:1" line-data="  CustomDataSource::Trace(">`CustomDataSource`</SwmToken>::Trace and <SwmToken path="examples/sdk/example_startup_trace.cc" pos="87:13:17" line-data="      [](CustomDataSource::TraceContext ctx) { ctx.Flush(); });">`ctx.Flush()`</SwmToken>, making sure nothing is left behind. Then it stops the tracing session, grabs all the trace data, and writes it out to a file called <SwmToken path="examples/sdk/example_startup_trace.cc" pos="97:11:13" line-data="  const char* filename = &quot;example_startup_trace.pftrace&quot;;">`example_startup_trace.pftrace`</SwmToken>. The file name is just a constant here, so every run overwrites the previous trace unless you change it. This function does more than just 'stop'—it handles the full wrap-up and export of trace data.

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
  const char* filename = "example_startup_trace.pftrace";
  output.open(filename, std::ios::out | std::ios::binary);
  output.write(trace_data.data(),
               static_cast<std::streamsize>(trace_data.size()));
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
