---
title: Trace Data Profile Conversion
---
This document describes how trace data is converted into heap, performance, or Java heap profiles based on the selected mode. The resulting profiles enable analysis and visualization of memory allocations, CPU activity, or Java heap usage.

# Dispatching Profile Conversion Based on Mode

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start trace to profile conversion"]
    click node1 openCode "src/traceconv/pprof_builder.cc:1182:1190"
    node1 --> node2{"Which profile type is requested?"}
    click node2 openCode "src/traceconv/pprof_builder.cc:1190:1198"
    node2 -->|"Heap Profile"| node3["Building and Filtering Heap Profiles"]
    
    node2 -->|"Perf Profile"| node4["Switching to Perf Profile Conversion"]
    
    node2 -->|"Java Heap Profile"| node5["Processing Java Heap Graphs"]
    
    node2 -->|"Unknown"| node6["Report unknown conversion option"]
    click node6 openCode "src/traceconv/pprof_builder.cc:1199:1200"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node3 goToHeading "Building and Filtering Heap Profiles"
node3:::HeadingStyle
click node4 goToHeading "Switching to Perf Profile Conversion"
node4:::HeadingStyle
click node5 goToHeading "Processing Java Heap Graphs"
node5:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start trace to profile conversion"]
%%     click node1 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1182:1190"
%%     node1 --> node2{"Which profile type is requested?"}
%%     click node2 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1190:1198"
%%     node2 -->|"Heap Profile"| node3["Building and Filtering Heap Profiles"]
%%     
%%     node2 -->|"Perf Profile"| node4["Switching to Perf Profile Conversion"]
%%     
%%     node2 -->|"Java Heap Profile"| node5["Processing Java Heap Graphs"]
%%     
%%     node2 -->|"Unknown"| node6["Report unknown conversion option"]
%%     click node6 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1199:1200"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node3 goToHeading "Building and Filtering Heap Profiles"
%% node3:::HeadingStyle
%% click node4 goToHeading "Switching to Perf Profile Conversion"
%% node4:::HeadingStyle
%% click node5 goToHeading "Processing Java Heap Graphs"
%% node5:::HeadingStyle
```

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1182">

---

In <SwmToken path="src/traceconv/pprof_builder.cc" pos="1182:2:2" line-data="bool TraceToPprof(trace_processor::TraceProcessor* tp,">`TraceToPprof`</SwmToken>, we start by checking the conversion mode and immediately delegate to the corresponding profile conversion function. For heap profiles, we call <SwmToken path="src/traceconv/pprof_builder.cc" pos="1192:5:5" line-data="      return heap_profile::TraceToHeapPprof(tp, output, annotate_frames, pid,">`TraceToHeapPprof`</SwmToken> to handle the specifics of extracting and serializing heap allocation data. This separation lets each profile type be processed with its own logic, keeping things clean and maintainable.

```c++
bool TraceToPprof(trace_processor::TraceProcessor* tp,
                  std::vector<SerializedProfile>* output,
                  ConversionMode mode,
                  uint64_t flags,
                  uint64_t pid,
                  const std::vector<uint64_t>& timestamps) {
  bool annotate_frames =
      flags & static_cast<uint64_t>(ConversionFlags::kAnnotateFrames);
  switch (mode) {
    case (ConversionMode::kHeapProfile):
      return heap_profile::TraceToHeapPprof(tp, output, annotate_frames, pid,
                                            timestamps);
```

---

</SwmSnippet>

## Building and Filtering Heap Profiles

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start conversion of trace data to heap profiles"]
    click node1 openCode "src/traceconv/pprof_builder.cc:763:767"
    subgraph loop1["For each heap profile allocation in trace"]
      node2{"Is allocation for target process and timestamp?"}
      click node2 openCode "src/traceconv/pprof_builder.cc:783:788"
      node2 -->|"Yes"| node3{"Heap type: malloc, Java, or generic?"}
      node2 -->|"No (Skip allocation)"| node2
      node3 -->|"Malloc"| node4["Generate malloc heap profile"]
      node3 -->|"Java"| node5["Generate Java heap profile"]
      node3 -->|"Generic"| node6["Generate generic heap profile"]
      click node3 openCode "src/traceconv/pprof_builder.cc:794:800"
      node4 --> node7{"Did profile verification succeed?"}
      node5 --> node7
      node6 --> node7
      click node7 openCode "src/traceconv/pprof_builder.cc:790:791"
      node7 -->|"Yes"| node8["Add profile to output"]
      node7 -->|"No"| node8
      click node8 openCode "src/traceconv/pprof_builder.cc:814:816"
      node8 --> node2
    end
    loop1 -->|"All allocations processed"| node9["Report any verification issues"]
    click node9 openCode "src/traceconv/pprof_builder.cc:824:829"
    node9 --> node10["Return success or failure"]
    click node10 openCode "src/traceconv/pprof_builder.cc:830:831"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start conversion of trace data to heap profiles"]
%%     click node1 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:763:767"
%%     subgraph loop1["For each heap profile allocation in trace"]
%%       node2{"Is allocation for target process and timestamp?"}
%%       click node2 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:783:788"
%%       node2 -->|"Yes"| node3{"Heap type: malloc, Java, or generic?"}
%%       node2 -->|"No (Skip allocation)"| node2
%%       node3 -->|"Malloc"| node4["Generate malloc heap profile"]
%%       node3 -->|"Java"| node5["Generate Java heap profile"]
%%       node3 -->|"Generic"| node6["Generate generic heap profile"]
%%       click node3 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:794:800"
%%       node4 --> node7{"Did profile verification succeed?"}
%%       node5 --> node7
%%       node6 --> node7
%%       click node7 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:790:791"
%%       node7 -->|"Yes"| node8["Add profile to output"]
%%       node7 -->|"No"| node8
%%       click node8 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:814:816"
%%       node8 --> node2
%%     end
%%     loop1 -->|"All allocations processed"| node9["Report any verification issues"]
%%     click node9 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:824:829"
%%     node9 --> node10["Return success or failure"]
%%     click node10 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:830:831"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="763">

---

In <SwmToken path="src/traceconv/pprof_builder.cc" pos="763:4:4" line-data="static bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,">`TraceToHeapPprof`</SwmToken>, we loop through heap profile allocations from the trace, filtering by PID and timestamp if needed. For each allocation, we pick the right set of views based on the heap type, set up the sample types, and serialize the profile. Only allocations matching the filters are processed, and the output is tailored to the heap's semantics.

```c++
static bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,
                             std::vector<SerializedProfile>* output,
                             bool annotate_frames,
                             uint64_t target_pid,
                             const std::vector<uint64_t>& target_timestamps) {
  trace_processor::StringPool interner;
  LocationTracker locations =
      PreprocessLocations(tp, &interner, annotate_frames);

  bool any_fail = false;
  Iterator it = tp->ExecuteQuery(
      "select distinct hpa.upid, hpa.ts, p.pid, hpa.heap_name "
      "from heap_profile_allocation hpa, "
      "process p where p.upid = hpa.upid;");
  while (it.Next()) {
    GProfileBuilder builder(locations, &interner);
    uint64_t upid = static_cast<uint64_t>(it.Get(0).AsLong());
    uint64_t ts = static_cast<uint64_t>(it.Get(1).AsLong());
    uint64_t profile_pid = static_cast<uint64_t>(it.Get(2).AsLong());
    const char* heap_name = it.Get(3).AsString();
    if ((target_pid > 0 && profile_pid != target_pid) ||
        (!target_timestamps.empty() &&
         std::find(target_timestamps.begin(), target_timestamps.end(), ts) ==
             target_timestamps.end())) {
      continue;
    }

    if (!VerifyPIDStats(tp, profile_pid))
      any_fail = true;

    std::vector<View> views;
    if (base::StringView(heap_name) == "libc.malloc") {
      views.assign(std::begin(kMallocViews), std::end(kMallocViews));
    } else if (base::StringView(heap_name) == "com.android.art") {
      views.assign(std::begin(kJavaSamplesViews), std::end(kJavaSamplesViews));
    } else {
      views.assign(std::begin(kGenericViews), std::end(kGenericViews));
    }

    std::vector<std::pair<std::string, std::string>> sample_types;
    for (const View& view : views) {
      sample_types.emplace_back(view.type, view.unit);
    }
    builder.WriteSampleTypes(sample_types);

    std::vector<Iterator> view_its =
        BuildViewIterators(tp, upid, ts, heap_name, views);
    std::string profile_proto;
    if (WriteAllocations(&builder, &view_its)) {
      profile_proto = builder.CompleteProfile(tp);
    }
    output->emplace_back(
        SerializedProfile{ProfileType::kHeapProfile, profile_pid,
                          std::move(profile_proto), heap_name});
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="819">

---

<SwmToken path="src/traceconv/pprof_builder.cc" pos="763:4:4" line-data="static bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,">`TraceToHeapPprof`</SwmToken> returns true if the SQL iterator was valid, otherwise false.

```c++
  if (!it.Status().ok()) {
    PERFETTO_DFATAL_OR_ELOG("Invalid iterator: %s",
                            it.Status().message().c_str());
    return false;
  }
  if (any_fail) {
    PERFETTO_ELOG(
        "One or more of your profiles had an issue. Please consult "
        "https://perfetto.dev/docs/data-sources/"
        "native-heap-profiler#troubleshooting");
  }
  return true;
}
```

---

</SwmSnippet>

## Switching to Perf Profile Conversion

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Preprocess trace locations"] --> node2["Log trace issues"]
  click node1 openCode "src/traceconv/pprof_builder.cc:1138:1140"
  node2 --> node3["Get all processes from trace"]
  click node2 openCode "src/traceconv/pprof_builder.cc:1142:1142"
  click node3 openCode "src/traceconv/pprof_builder.cc:1145:1145"

  subgraph loop1["For each process in trace data"]
    node3 --> node4{"Is target_pid set and matches process?"}
    click node4 openCode "src/traceconv/pprof_builder.cc:1149:1150"
    node4 -->|"Yes"| node5["Build and store performance profile"]
    click node5 openCode "src/traceconv/pprof_builder.cc:1152:1175"
    node4 -->|"No"| node8["Next process"]
    click node8 openCode "src/traceconv/pprof_builder.cc:1146:1176"
    node5 --> node8
  end
  node8 --> node6["Return all generated profiles"]
  click node6 openCode "src/traceconv/pprof_builder.cc:1176:1176"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Preprocess trace locations"] --> node2["Log trace issues"]
%%   click node1 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1138:1140"
%%   node2 --> node3["Get all processes from trace"]
%%   click node2 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1142:1142"
%%   click node3 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1145:1145"
%% 
%%   subgraph loop1["For each process in trace data"]
%%     node3 --> node4{"Is <SwmToken path="src/traceconv/pprof_builder.cc" pos="766:3:3" line-data="                             uint64_t target_pid,">`target_pid`</SwmToken> set and matches process?"}
%%     click node4 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1149:1150"
%%     node4 -->|"Yes"| node5["Build and store performance profile"]
%%     click node5 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1152:1175"
%%     node4 -->|"No"| node8["Next process"]
%%     click node8 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1146:1176"
%%     node5 --> node8
%%   end
%%   node8 --> node6["Return all generated profiles"]
%%   click node6 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1176:1176"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1194">

---

Back in <SwmToken path="src/traceconv/pprof_builder.cc" pos="1182:2:2" line-data="bool TraceToPprof(trace_processor::TraceProcessor* tp,">`TraceToPprof`</SwmToken>, after heap profiles, we call <SwmToken path="src/traceconv/pprof_builder.cc" pos="1195:5:5" line-data="      return perf_profile::TraceToPerfPprof(tp, output, annotate_frames, pid);">`TraceToPerfPprof`</SwmToken> for perf profile modes to handle CPU profiling data.

```c++
    case (ConversionMode::kPerfProfile):
      return perf_profile::TraceToPerfPprof(tp, output, annotate_frames, pid);
```

---

</SwmSnippet>

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1134">

---

<SwmToken path="src/traceconv/pprof_builder.cc" pos="1134:4:4" line-data="static bool TraceToPerfPprof(trace_processor::TraceProcessor* tp,">`TraceToPerfPprof`</SwmToken> groups perf samples by process, filtering by PID if needed. For each process, it builds a profile by querying callsite IDs, adds each sample, and serializes the result. This way, each output profile is process-specific and contains only relevant samples.

```c++
static bool TraceToPerfPprof(trace_processor::TraceProcessor* tp,
                             std::vector<SerializedProfile>* output,
                             bool annotate_frames,
                             uint64_t target_pid) {
  trace_processor::StringPool interner;
  LocationTracker locations =
      PreprocessLocations(tp, &interner, annotate_frames);

  LogTracePerfEventIssues(tp);

  // Aggregate samples by upid when building profiles.
  std::map<uint64_t, ProcessInfo> process_map = GetProcessMap(tp);
  for (const auto& p : process_map) {
    const ProcessInfo& process = p.second;

    if (target_pid != 0 && process.pid != target_pid)
      continue;

    GProfileBuilder builder(locations, &interner);
    builder.WriteSampleTypes({{"samples", "count"}});

    std::string query = "select callsite_id from perf_sample where utid in (" +
                        AsCsvString(process.utids) +
                        ") and callsite_id is not null order by ts asc;";

    protozero::PackedVarInt single_count_value;
    single_count_value.Append(1);

    Iterator it = tp->ExecuteQuery(query);
    while (it.Next()) {
      int64_t callsite_id = static_cast<int64_t>(it.Get(0).AsLong());
      builder.AddSample(single_count_value, callsite_id);
    }
    if (!it.Status().ok()) {
      PERFETTO_DFATAL_OR_ELOG("Failed to iterate over samples: %s",
                              it.Status().c_message());
      return false;
    }

    std::string profile_proto = builder.CompleteProfile(tp);
    output->emplace_back(SerializedProfile{
        ProfileType::kPerfProfile, process.pid, std::move(profile_proto), ""});
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1196">

---

Back in <SwmToken path="src/traceconv/pprof_builder.cc" pos="1182:2:2" line-data="bool TraceToPprof(trace_processor::TraceProcessor* tp,">`TraceToPprof`</SwmToken>, after handling perf profiles, we check if the mode is for Java heap profiles and call the Java-specific <SwmToken path="src/traceconv/pprof_builder.cc" pos="1197:5:5" line-data="      return java_heap_profile::TraceToHeapPprof(tp, output, pid, timestamps);">`TraceToHeapPprof`</SwmToken>. This lets us process Java heap graphs separately from native heap or perf data.

```c++
    case (ConversionMode::kJavaHeapProfile):
      return java_heap_profile::TraceToHeapPprof(tp, output, pid, timestamps);
  }
```

---

</SwmSnippet>

## Processing Java Heap Graphs

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Start trace to heap profile conversion"]
    click node1 openCode "src/traceconv/pprof_builder.cc:995:1001"
    node1 --> node2["For each heap graph in trace"]
    click node2 openCode "src/traceconv/pprof_builder.cc:1002:1044"
    subgraph loop1["For each heap graph"]
      node2 --> node3{"Does heap graph match target process id and timestamp?"}
      click node3 openCode "src/traceconv/pprof_builder.cc:1011:1016"
      node3 -->|"Yes"| node4["Convert heap graph to Java heap profile and add to output"]
      click node4 openCode "src/traceconv/pprof_builder.cc:1017:1043"
      node3 -->|"No"| node2
      node4 --> node2
    end
    node2 --> node5{"Was trace data processed successfully?"}
    click node5 openCode "src/traceconv/pprof_builder.cc:1046:1049"
    node5 -->|"Yes"| node6["Return success"]
    click node6 openCode "src/traceconv/pprof_builder.cc:1052:1053"
    node5 -->|"No"| node7["Return failure"]
    click node7 openCode "src/traceconv/pprof_builder.cc:1049:1050"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Start trace to heap profile conversion"]
%%     click node1 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:995:1001"
%%     node1 --> node2["For each heap graph in trace"]
%%     click node2 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1002:1044"
%%     subgraph loop1["For each heap graph"]
%%       node2 --> node3{"Does heap graph match target process id and timestamp?"}
%%       click node3 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1011:1016"
%%       node3 -->|"Yes"| node4["Convert heap graph to Java heap profile and add to output"]
%%       click node4 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1017:1043"
%%       node3 -->|"No"| node2
%%       node4 --> node2
%%     end
%%     node2 --> node5{"Was trace data processed successfully?"}
%%     click node5 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1046:1049"
%%     node5 -->|"Yes"| node6["Return success"]
%%     click node6 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1052:1053"
%%     node5 -->|"No"| node7["Return failure"]
%%     click node7 openCode "<SwmPath>[src/traceconv/pprof_builder.cc](src/traceconv/pprof_builder.cc)</SwmPath>:1049:1050"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="995">

---

In <SwmToken path="src/traceconv/pprof_builder.cc" pos="995:2:2" line-data="bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,">`TraceToHeapPprof`</SwmToken> (Java version), we query for unique heap graph samples and process each one, filtering by PID and timestamp if needed. For each valid sample, we set up Java allocation views, preprocess locations, and serialize the heap graph into a profile.

```c++
bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,
                      std::vector<SerializedProfile>* output,
                      uint64_t target_pid,
                      const std::vector<uint64_t>& target_timestamps) {
  trace_processor::StringPool interner;

  // Find all heap graphs available in the trace and iterate over them
  Iterator it = tp->ExecuteQuery(
      "select distinct hgo.graph_sample_ts, hgo.upid, p.pid from "
      "heap_graph_object hgo join process p using (upid)");

  while (it.Next()) {
    uint64_t ts = static_cast<uint64_t>(it.Get(0).AsLong());
    uint64_t upid = static_cast<uint64_t>(it.Get(1).AsLong());
    uint64_t profile_pid = static_cast<uint64_t>(it.Get(2).AsLong());

    if ((target_pid > 0 && profile_pid != target_pid) ||
        (!target_timestamps.empty() &&
         std::find(target_timestamps.begin(), target_timestamps.end(), ts) ==
             target_timestamps.end())) {
      continue;
    }

    // flamegraph id -> view values
    std::unordered_map<int64_t, std::vector<int64_t>> view_values;

    std::vector<View> views;
    views.assign(std::begin(kJavaAllocationViews),
                 std::end(kJavaAllocationViews));

    LocationTracker locations = PreprocessLocationsForJavaHeap(
        tp, &interner, views, view_values, upid, ts);

    GProfileBuilder builder(locations, &interner);

    std::vector<std::pair<std::string, std::string>> sample_types;
    for (const auto& view : views) {
      sample_types.emplace_back(view.type, view.unit);
    }
    builder.WriteSampleTypes(sample_types);

    std::string profile_proto;
    if (WriteAllocations(&builder, view_values)) {
      profile_proto = builder.CompleteProfile(tp, /*write_mappings=*/false);
    }

    output->emplace_back(SerializedProfile{ProfileType::kJavaHeapProfile,
                                           profile_pid,
                                           std::move(profile_proto), ""});
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1046">

---

<SwmToken path="src/traceconv/pprof_builder.cc" pos="763:4:4" line-data="static bool TraceToHeapPprof(trace_processor::TraceProcessor* tp,">`TraceToHeapPprof`</SwmToken> returns true if the SQL iterator was valid, otherwise false.

```c++
  if (!it.Status().ok()) {
    PERFETTO_DFATAL_OR_ELOG("Invalid iterator: %s",
                            it.Status().message().c_str());
    return false;
  }

  return true;
}
```

---

</SwmSnippet>

## Handling Unknown Conversion Modes

<SwmSnippet path="/src/traceconv/pprof_builder.cc" line="1199">

---

Back in <SwmToken path="src/traceconv/pprof_builder.cc" pos="1182:2:2" line-data="bool TraceToPprof(trace_processor::TraceProcessor* tp,">`TraceToPprof`</SwmToken>, if the mode is unknown, we abort with a fatal error.

```c++
  PERFETTO_FATAL("unknown conversion option");  // for gcc
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
