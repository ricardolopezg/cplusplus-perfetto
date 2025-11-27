---
title: Filtering Protocol Buffer Messages
---
This document describes the flow for processing protocol buffer messages and filter configurations to produce filtered outputs or filter bytecode. Users specify their desired operation through command-line arguments, enabling flexible filtering using rules from files, configurations, or schemas. The flow supports deduplication and can output usage statistics as needed.

# Entry Point and Delegation

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="391">

---

Main just hands off execution to perfetto::<SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="392:5:5" line-data="  return perfetto::proto_filter::Main(argc, argv);">`proto_filter`</SwmToken>::Main, passing along the command-line arguments. This keeps the entry point clean and puts all the logic in Main, which is where everything actually happens.

```c++
int main(int argc, char** argv) {
  return perfetto::proto_filter::Main(argc, argv);
}
```

---

</SwmSnippet>

# Argument Parsing and Setup

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    subgraph loop1["Parse command-line options"]
        node1["Start: Parse command-line options"]
        click node1 openCode "src/tools/proto_filter/proto_filter.cc:135:214"
        node1 --> node2{"Help or version requested?"}
        click node2 openCode "src/tools/proto_filter/proto_filter.cc:142:145"
        node2 -->|"Yes"| node3["Print help/version and exit"]
        click node3 openCode "src/tools/proto_filter/proto_filter.cc:208:210"
        node2 -->|"No"| node4{"Are required inputs present?"}
        click node4 openCode "src/tools/proto_filter/proto_filter.cc:216:219"
        node4 -->|"No"| node5["Print usage and exit with error"]
        click node5 openCode "src/tools/proto_filter/proto_filter.cc:217:218"
        node4 -->|"Yes"| node6["Continue processing"]
    end
    node3 --> node18["Exit"]
    node5 --> node18
    node6 --> node7{"Mutually exclusive options provided?"}
    click node7 openCode "src/tools/proto_filter/proto_filter.cc:221:224"
    node7 -->|"Yes"| node5
    node7 -->|"No"| node8{"Which filter bytecode source?"}
    click node8 openCode "src/tools/proto_filter/proto_filter.cc:251:294"
    node8 -->|"filter_in provided"| node9["Load filter bytecode from file"]
    click node9 openCode "src/tools/proto_filter/proto_filter.cc:251:258"
    node8 -->|"config_in provided"| node10["Load filter bytecode and rules from config"]
    click node10 openCode "src/tools/proto_filter/proto_filter.cc:258:285"
    node8 -->|"schema_in provided"| node11["Generate filter bytecode from schema"]
    click node11 openCode "src/tools/proto_filter/proto_filter.cc:290:294"
    node9 --> node12{"Deduplication requested?"}
    node10 --> subgraph loop2["For each string filter rule in config"]
        node10a["Add string filter rule"]
        click node10a openCode "src/tools/proto_filter/proto_filter.cc:277:285"
    end
    node10 --> node12
    node11 --> node12
    node12 -->|"Yes"| node13["Apply deduplication"]
    click node13 openCode "src/tools/proto_filter/proto_filter.cc:244:245"
    node12 -->|"No"| node14["Continue"]
    node13 --> node14
    node14 --> node15{"Apply filter to input message?"}
    click node15 openCode "src/tools/proto_filter/proto_filter.cc:345:354"
    node15 -->|"Yes"| node16["Apply filter and write filtered message"]
    click node16 openCode "src/tools/proto_filter/proto_filter.cc:345:362"
    node15 -->|"No"| node17["Write filter bytecode outputs"]
    click node17 openCode "src/tools/proto_filter/proto_filter.cc:305:341"
    node16 --> subgraph loop3["For each field in usage map"]
        node16a["Print field usage statistics"]
        click node16a openCode "src/tools/proto_filter/proto_filter.cc:366:372"
    end
    node16 --> node19{"Show dedupe warning?"}
    node17 --> node19
    node19 -->|"Yes"| node20["Show dedupe warning"]
    click node20 openCode "src/tools/proto_filter/proto_filter.cc:379:382"
    node19 -->|"No"| node18["Exit"]
    node20 --> node18["Exit"]
    click node18 openCode "src/tools/proto_filter/proto_filter.cc:384:385"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     subgraph loop1["Parse command-line options"]
%%         node1["Start: Parse command-line options"]
%%         click node1 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:135:214"
%%         node1 --> node2{"Help or version requested?"}
%%         click node2 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:142:145"
%%         node2 -->|"Yes"| node3["Print help/version and exit"]
%%         click node3 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:208:210"
%%         node2 -->|"No"| node4{"Are required inputs present?"}
%%         click node4 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:216:219"
%%         node4 -->|"No"| node5["Print usage and exit with error"]
%%         click node5 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:217:218"
%%         node4 -->|"Yes"| node6["Continue processing"]
%%     end
%%     node3 --> node18["Exit"]
%%     node5 --> node18
%%     node6 --> node7{"Mutually exclusive options provided?"}
%%     click node7 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:221:224"
%%     node7 -->|"Yes"| node5
%%     node7 -->|"No"| node8{"Which filter bytecode source?"}
%%     click node8 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:251:294"
%%     node8 -->|"<SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="115:3:3" line-data="      {&quot;filter_in&quot;, required_argument, nullptr, &#39;f&#39;},">`filter_in`</SwmToken> provided"| node9["Load filter bytecode from file"]
%%     click node9 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:251:258"
%%     node8 -->|"<SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="114:3:3" line-data="      {&quot;config_in&quot;, required_argument, nullptr, &#39;c&#39;},">`config_in`</SwmToken> provided"| node10["Load filter bytecode and rules from config"]
%%     click node10 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:258:285"
%%     node8 -->|"<SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="110:3:3" line-data="      {&quot;schema_in&quot;, required_argument, nullptr, &#39;s&#39;},">`schema_in`</SwmToken> provided"| node11["Generate filter bytecode from schema"]
%%     click node11 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:290:294"
%%     node9 --> node12{"Deduplication requested?"}
%%     node10 --> subgraph loop2["For each string filter rule in config"]
%%         node10a["Add string filter rule"]
%%         click node10a openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:277:285"
%%     end
%%     node10 --> node12
%%     node11 --> node12
%%     node12 -->|"Yes"| node13["Apply deduplication"]
%%     click node13 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:244:245"
%%     node12 -->|"No"| node14["Continue"]
%%     node13 --> node14
%%     node14 --> node15{"Apply filter to input message?"}
%%     click node15 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:345:354"
%%     node15 -->|"Yes"| node16["Apply filter and write filtered message"]
%%     click node16 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:345:362"
%%     node15 -->|"No"| node17["Write filter bytecode outputs"]
%%     click node17 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:305:341"
%%     node16 --> subgraph loop3["For each field in usage map"]
%%         node16a["Print field usage statistics"]
%%         click node16a openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:366:372"
%%     end
%%     node16 --> node19{"Show dedupe warning?"}
%%     node17 --> node19
%%     node19 -->|"Yes"| node20["Show dedupe warning"]
%%     click node20 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:379:382"
%%     node19 -->|"No"| node18["Exit"]
%%     node20 --> node18["Exit"]
%%     click node18 openCode "<SwmPath>[src/…/proto_filter/proto_filter.cc](src/tools/proto_filter/proto_filter.cc)</SwmPath>:384:385"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="104">

---

In Main, we parse all the command-line options and set up variables for input/output files, schema, config, filter options, and flags. This sets up everything needed for the rest of the filtering logic.

```c++
int Main(int argc, char** argv) {
  static const option long_options[] = {
      {"help", no_argument, nullptr, 'h'},
      {"version", no_argument, nullptr, 'v'},
      {"dedupe", no_argument, nullptr, 'd'},
      {"proto_path", required_argument, nullptr, 'I'},
      {"schema_in", required_argument, nullptr, 's'},
      {"root_message", required_argument, nullptr, 'r'},
      {"msg_in", required_argument, nullptr, 'i'},
      {"msg_out", required_argument, nullptr, 'o'},
      {"config_in", required_argument, nullptr, 'c'},
      {"filter_in", required_argument, nullptr, 'f'},
      {"filter_out", required_argument, nullptr, 'F'},
      {"filter_oct_out", required_argument, nullptr, 'T'},
      {"passthrough", required_argument, nullptr, 'x'},
      {"filter_string", required_argument, nullptr, 'g'},
      {nullptr, 0, nullptr, 0}};

  std::string msg_in;
  std::string msg_out;
  std::string config_in;
  std::string filter_in;
  std::string schema_in;
  std::string filter_out;
  std::string filter_oct_out;
  std::string proto_path;
  std::string root_message_arg;
  std::set<std::string> passthrough_fields;
  std::set<std::string> filter_string_fields;
  bool dedupe = false;

  for (;;) {
    int option = getopt_long(
        argc, argv, "hvdI:s:r:i:o:f:F:T:x:g:c:", long_options, nullptr);

    if (option == -1)
      break;  // EOF.

    if (option == 'v') {
      printf("%s\n", base::GetVersionString());
      exit(0);
    }

    if (option == 'd') {
      dedupe = true;
      continue;
    }

    if (option == 'I') {
      proto_path = optarg;
      continue;
    }

    if (option == 's') {
      schema_in = optarg;
      continue;
    }

    if (option == 'c') {
      config_in = optarg;
      continue;
    }

    if (option == 'r') {
      root_message_arg = optarg;
      continue;
    }

    if (option == 'i') {
      msg_in = optarg;
      continue;
    }

    if (option == 'o') {
      msg_out = optarg;
      continue;
    }

    if (option == 'f') {
      filter_in = optarg;
      continue;
    }

    if (option == 'F') {
      filter_out = optarg;
      continue;
    }

    if (option == 'T') {
      filter_oct_out = optarg;
      continue;
    }

    if (option == 'x') {
      passthrough_fields.insert(optarg);
      continue;
    }

    if (option == 'g') {
      filter_string_fields.insert(optarg);
      continue;
    }

    if (option == 'h') {
      fprintf(stdout, kUsage);
      exit(0);
    }

    fprintf(stderr, kUsage);
    exit(1);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="216">

---

After parsing arguments, we check for required inputs and error out if they're missing or conflicting. Then, if a message or schema is provided, we load them. <SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="235:3:3" line-data="  protozero::FilterUtil filter;">`FilterUtil`</SwmToken> handles schema loading and deduplication, while <SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="248:3:3" line-data="  protozero::MessageFilter msg_filter;">`MessageFilter`</SwmToken> is set up for filtering. We also load filter bytecode from a file, config, or generate it from the schema as needed.

```c++
  if (msg_in.empty() && filter_in.empty() && schema_in.empty()) {
    fprintf(stderr, kUsage);
    return 1;
  }

  if (!filter_in.empty() && !config_in.empty()) {
    fprintf(stderr, kUsage);
    return 1;
  }

  std::string msg_in_data;
  if (!msg_in.empty()) {
    PERFETTO_LOG("Loading proto-encoded message from %s", msg_in.c_str());
    if (!base::ReadFile(msg_in, &msg_in_data)) {
      PERFETTO_ELOG("Could not open message file %s", msg_in.c_str());
      return 1;
    }
  }

  protozero::FilterUtil filter;
  if (!schema_in.empty()) {
    PERFETTO_LOG("Loading proto schema from %s", schema_in.c_str());
    if (!filter.LoadMessageDefinition(schema_in, root_message_arg, proto_path,
                                      passthrough_fields,
                                      filter_string_fields)) {
      PERFETTO_ELOG("Failed to parse proto schema from %s", schema_in.c_str());
      return 1;
    }
    if (dedupe)
      filter.Dedupe();
  }

  protozero::MessageFilter msg_filter;
  std::string filter_data;
  std::string filter_data_src;
  if (!filter_in.empty()) {
    PERFETTO_LOG("Loading filter bytecode from %s", filter_in.c_str());
    if (!base::ReadFile(filter_in, &filter_data)) {
      PERFETTO_ELOG("Could not open filter file %s", filter_in.c_str());
      return 1;
    }
    filter_data_src = filter_in;
  } else if (!config_in.empty()) {
    PERFETTO_LOG("Loading filter bytecode and rules from %s",
                 config_in.c_str());
    std::string config_data;
    if (!base::ReadFile(config_in, &config_data)) {
      PERFETTO_ELOG("Could not open config file %s", config_in.c_str());
      return 1;
    }
    auto res = TraceConfigTxtToPb(config_data, config_in);
    if (!res.ok()) {
      fprintf(stderr, "%s\n", res.status().c_message());
      return 1;
    }

    std::vector<uint8_t>& config_bytes = res.value();
    protos::gen::TraceConfig config;
    config.ParseFromArray(config_bytes.data(), config_bytes.size());

    const auto& trace_filter = config.trace_filter();
    for (const auto& rule : trace_filter.string_filter_chain().rules()) {
      auto opt_policy = ConvertPolicy(rule.policy());
      if (!opt_policy) {
        PERFETTO_ELOG("Unknown string filter policy %d", rule.policy());
        return 1;
      }
      msg_filter.string_filter().AddRule(*opt_policy, rule.regex_pattern(),
                                         rule.atrace_payload_starts_with());
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="286">

---

Here we decide where to get the filter bytecode: from a file, from a config (parsing rules and extracting bytecode), or by generating it from the schema. Once we have the bytecode, we load it into the <SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="248:3:3" line-data="  protozero::MessageFilter msg_filter;">`MessageFilter`</SwmToken> for later use.

```c++
    filter_data = trace_filter.bytecode_v2().empty()
                      ? trace_filter.bytecode()
                      : trace_filter.bytecode_v2();
    filter_data_src = config_in;
  } else if (!schema_in.empty()) {
    PERFETTO_LOG("Generating filter bytecode from %s", schema_in.c_str());
    filter_data = filter.GenerateFilterBytecode();
    filter_data_src = schema_in;
  }

  if (!filter_data.empty()) {
    const uint8_t* data = reinterpret_cast<const uint8_t*>(filter_data.data());
    if (!msg_filter.LoadFilterBytecode(data, filter_data.size())) {
      PERFETTO_ELOG("Failed to parse filter bytecode from %s",
                    filter_data_src.c_str());
      return 1;
    }
  }

  // Write the filter bytecode in output.
  if (!filter_out.empty()) {
    auto fd = base::OpenFile(filter_out, O_WRONLY | O_TRUNC | O_CREAT, 0644);
    if (!fd) {
      PERFETTO_ELOG("Could not open filter out path %s", filter_out.c_str());
      return 1;
    }
    PERFETTO_LOG("Writing filter bytecode (%zu bytes) into %s",
                 filter_data.size(), filter_out.c_str());
    base::WriteAll(*fd, filter_data.data(), filter_data.size());
  }

  if (!filter_oct_out.empty()) {
    auto fd =
        base::OpenFile(filter_oct_out, O_WRONLY | O_TRUNC | O_CREAT, 0644);
    if (!fd) {
      PERFETTO_ELOG("Could not open filter out path %s",
                    filter_oct_out.c_str());
      return 1;
    }
    std::string oct_str;
    oct_str.reserve(filter_data.size() * 4 + 64);
    oct_str.append("trace_filter {\n  bytecode: \"");
    for (char c : filter_data) {
      uint8_t octet = static_cast<uint8_t>(c);
      char buf[5]{'\\', '0', '0', '0', 0};
      for (uint8_t i = 0; i < 3; ++i) {
        buf[3 - i] = static_cast<char>('0' + static_cast<uint8_t>(octet) % 8);
        octet /= 8;
      }
      oct_str.append(buf);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="337">

---

If requested, we write the filter bytecode to an output file (raw or octal-encoded). Then, if there's an input message, we apply the filter using <SwmToken path="src/tools/proto_filter/proto_filter.cc" pos="248:3:3" line-data="  protozero::MessageFilter msg_filter;">`MessageFilter`</SwmToken> and collect the filtered result. This is where the actual filtering happens.

```c++
    oct_str.append("\"\n}\n");
    PERFETTO_LOG("Writing filter bytecode (%zu bytes) into %s", oct_str.size(),
                 filter_oct_out.c_str());
    base::WriteAll(*fd, oct_str.data(), oct_str.size());
  }

  // Apply the filter to the input message (if any).
  std::vector<uint8_t> msg_filtered_data;
  if (!msg_in.empty()) {
    PERFETTO_LOG("Applying filter %s to proto message %s",
                 filter_data_src.c_str(), msg_in.c_str());
    msg_filter.enable_field_usage_tracking(true);
    auto res = msg_filter.FilterMessage(msg_in_data.data(), msg_in_data.size());
    if (res.error)
      PERFETTO_FATAL("Filtering failed");
    msg_filtered_data.insert(msg_filtered_data.end(), res.data.get(),
                             res.data.get() + res.size);
  }

  // Write out the filtered message.
  if (!msg_out.empty()) {
    PERFETTO_LOG("Writing filtered proto bytes (%zu bytes) into %s",
                 msg_filtered_data.size(), msg_out.c_str());
    auto fd = base::OpenFile(msg_out, O_WRONLY | O_TRUNC | O_CREAT, 0644);
    base::WriteAll(*fd, msg_filtered_data.data(), msg_filtered_data.size());
  }

  if (!msg_in.empty()) {
    const auto& field_usage_map = msg_filter.field_usage();
    for (const auto& it : field_usage_map) {
      const std::string& field_path_varint = it.first;
      int32_t num_occurrences = it.second;
      std::string path_str = filter.LookupField(field_path_varint);
      printf("%-100s %s %d\n", path_str.c_str(),
             num_occurrences < 0 ? "DROP" : "PASS", std::abs(num_occurrences));
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/tools/proto_filter/proto_filter.cc" line="373">

---

Finally, we print field usage stats if a message was filtered, or print the filter as text if only a schema was provided. If filter bytecode was generated without dedupe, we warn the user. Main returns 0 for success.

```c++
  } else if (!schema_in.empty()) {
    filter.PrintAsText(!filter_data.empty() ? std::make_optional(filter_data)
                                            : std::nullopt);
  }

  if ((!filter_out.empty() || !filter_oct_out.empty()) && !dedupe) {
    PERFETTO_ELOG(
        "Warning: looks like you are generating a filter without --dedupe. For "
        "production use cases, --dedupe can make the output bytecode "
        "significantly smaller.");
  }
  return 0;
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
