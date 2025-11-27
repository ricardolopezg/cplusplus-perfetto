---
title: Storing and Updating Metadata Entries
---
This document describes how metadata entries are stored and updated. When a metadata key and value are provided, the system checks for special cases such as trace UUIDs for crash diagnostics, ensures each metadata entry is unique, and either updates an existing entry or adds a new one.

```mermaid
flowchart TD
  node1["Storing and Updating Metadata Entries
(Storing and Updating Metadata Entries)"]:::HeadingStyle
  click node1 goToHeading "Storing and Updating Metadata Entries"
  node1 --> node2{"Is key trace_uuid and value a string?
(Storing and Updating Metadata Entries)"}:::HeadingStyle
  click node2 goToHeading "Storing and Updating Metadata Entries"
  node2 --> node3["Checking for Existing Metadata"]:::HeadingStyle
  click node3 goToHeading "Checking for Existing Metadata"
  node3 -->|"Entry exists"| node5["Finalizing Metadata Storage"]:::HeadingStyle
  click node5 goToHeading "Finalizing Metadata Storage"
  node3 -->|"Entry does not exist"| node4["Inserting into the Metadata Table"]:::HeadingStyle
  click node4 goToHeading "Inserting into the Metadata Table"
  node4 --> node5
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% flowchart TD
%%   node1["Storing and Updating Metadata Entries
%% (Storing and Updating Metadata Entries)"]:::HeadingStyle
%%   click node1 goToHeading "Storing and Updating Metadata Entries"
%%   node1 --> node2{"Is key <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="50:7:7" line-data="  // When the trace_uuid is set, store a copy in a crash key, so in case of">`trace_uuid`</SwmToken> and value a string?
%% (Storing and Updating Metadata Entries)"}:::HeadingStyle
%%   click node2 goToHeading "Storing and Updating Metadata Entries"
%%   node2 --> node3["Checking for Existing Metadata"]:::HeadingStyle
%%   click node3 goToHeading "Checking for Existing Metadata"
%%   node3 -->|"Entry exists"| node5["Finalizing Metadata Storage"]:::HeadingStyle
%%   click node5 goToHeading "Finalizing Metadata Storage"
%%   node3 -->|"Entry does not exist"| node4["Inserting into the Metadata Table"]:::HeadingStyle
%%   click node4 goToHeading "Inserting into the Metadata Table"
%%   node4 --> node5
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

(Note - these are only some of the entry points of this flow)

```mermaid
graph TD;
      7947eb6e403ecf52613423a7de4d6d146c0dc02dc06452322016bc2f8609b290(src/…/proto/track_event_module.cc::TokenizePacket) --> c005dbe7b73e3d60ee01d5b7dcb5b2db7d3b742bd9395d166fc5cd4a682245e7(src/…/proto/track_event_tokenizer.cc::TokenizeRangeOfInterestPacket)

c005dbe7b73e3d60ee01d5b7dcb5b2db7d3b742bd9395d166fc5cd4a682245e7(src/…/proto/track_event_tokenizer.cc::TokenizeRangeOfInterestPacket) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(src/…/common/metadata_tracker.cc::SetMetadata):::mainFlowStyle

f81d89370499bccf9596509449645a0140b685c91273870a0813e517eb1a877f(src/…/perf_text/perf_text_trace_tokenizer.cc::Parse) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(src/…/util/clock_synchronizer.h::SetTraceTimeClock)

9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(src/…/util/clock_synchronizer.h::SetTraceTimeClock) --> eaebd08511eab323fa91bbc495785ef065c357785b00c0fe69264cb0f49efaec(src/…/common/clock_tracker.cc::OnSetTraceTimeClock)

eaebd08511eab323fa91bbc495785ef065c357785b00c0fe69264cb0f49efaec(src/…/common/clock_tracker.cc::OnSetTraceTimeClock) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(src/…/common/metadata_tracker.cc::SetMetadata):::mainFlowStyle

5bdd1128dacdfb278b4e7e1949146dfb801c5179b86619019e9a0a317b33a9f0(src/…/perf/perf_data_tokenizer.cc::ParseFeature) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(src/…/common/metadata_tracker.cc::SetMetadata):::mainFlowStyle

8ffc7776b1d0b388f0bf8014f0d3d0dd714948cb0c8fb3e013efe0161f9d6f70(src/…/proto/proto_trace_reader.cc::ParseClockSnapshot) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(src/…/util/clock_synchronizer.h::SetTraceTimeClock)

b911609543ff536610e21fa09aa24ea3c7d47a091a57ca59d2a30968c04388d1(src/…/gecko/gecko_trace_tokenizer.cc::NotifyEndOfFile) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(src/…/util/clock_synchronizer.h::SetTraceTimeClock)


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9

%% Swimm:
%% graph TD;
%%       7947eb6e403ecf52613423a7de4d6d146c0dc02dc06452322016bc2f8609b290(<SwmPath>[src/…/proto/track_event_module.cc](src/trace_processor/importers/proto/track_event_module.cc)</SwmPath>::TokenizePacket) --> c005dbe7b73e3d60ee01d5b7dcb5b2db7d3b742bd9395d166fc5cd4a682245e7(<SwmPath>[src/…/proto/track_event_tokenizer.cc](src/trace_processor/importers/proto/track_event_tokenizer.cc)</SwmPath>::TokenizeRangeOfInterestPacket)
%% 
%% c005dbe7b73e3d60ee01d5b7dcb5b2db7d3b742bd9395d166fc5cd4a682245e7(<SwmPath>[src/…/proto/track_event_tokenizer.cc](src/trace_processor/importers/proto/track_event_tokenizer.cc)</SwmPath>::TokenizeRangeOfInterestPacket) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>):::mainFlowStyle
%% 
%% f81d89370499bccf9596509449645a0140b685c91273870a0813e517eb1a877f(<SwmPath>[src/…/perf_text/perf_text_trace_tokenizer.cc](src/trace_processor/importers/perf_text/perf_text_trace_tokenizer.cc)</SwmPath>::Parse) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(<SwmPath>[src/…/util/clock_synchronizer.h](src/trace_processor/util/clock_synchronizer.h)</SwmPath>::SetTraceTimeClock)
%% 
%% 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(<SwmPath>[src/…/util/clock_synchronizer.h](src/trace_processor/util/clock_synchronizer.h)</SwmPath>::SetTraceTimeClock) --> eaebd08511eab323fa91bbc495785ef065c357785b00c0fe69264cb0f49efaec(<SwmPath>[src/…/common/clock_tracker.cc](src/trace_processor/importers/common/clock_tracker.cc)</SwmPath>::OnSetTraceTimeClock)
%% 
%% eaebd08511eab323fa91bbc495785ef065c357785b00c0fe69264cb0f49efaec(<SwmPath>[src/…/common/clock_tracker.cc](src/trace_processor/importers/common/clock_tracker.cc)</SwmPath>::OnSetTraceTimeClock) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>):::mainFlowStyle
%% 
%% 5bdd1128dacdfb278b4e7e1949146dfb801c5179b86619019e9a0a317b33a9f0(<SwmPath>[src/…/perf/perf_data_tokenizer.cc](src/trace_processor/importers/perf/perf_data_tokenizer.cc)</SwmPath>::ParseFeature) --> a9f1359fc7a4d614b3e7371ef8c111c32e830d1031140d47e38aabbd2f1350a4(<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>):::mainFlowStyle
%% 
%% 8ffc7776b1d0b388f0bf8014f0d3d0dd714948cb0c8fb3e013efe0161f9d6f70(<SwmPath>[src/…/proto/proto_trace_reader.cc](src/trace_processor/importers/proto/proto_trace_reader.cc)</SwmPath>::ParseClockSnapshot) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(<SwmPath>[src/…/util/clock_synchronizer.h](src/trace_processor/util/clock_synchronizer.h)</SwmPath>::SetTraceTimeClock)
%% 
%% b911609543ff536610e21fa09aa24ea3c7d47a091a57ca59d2a30968c04388d1(<SwmPath>[src/…/gecko/gecko_trace_tokenizer.cc](src/trace_processor/importers/gecko/gecko_trace_tokenizer.cc)</SwmPath>::NotifyEndOfFile) --> 9efa26c67305ed95772fd895ede0f410a7f49ac10f1bef5dad33ac4f2729c5c4(<SwmPath>[src/…/util/clock_synchronizer.h](src/trace_processor/util/clock_synchronizer.h)</SwmPath>::SetTraceTimeClock)
%% 
%% 
%% classDef mainFlowStyle color:#000000,fill:#7CB9F4
%% classDef rootsStyle color:#000000,fill:#00FFF4
%% classDef Style1 color:#000000,fill:#00FFAA
%% classDef Style2 color:#000000,fill:#FFFF00
%% classDef Style3 color:#000000,fill:#AA7CB9
```

# Storing and Updating Metadata Entries

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Receive metadata key and value"] --> node2{"Is key trace_uuid and value a string?"}
    click node1 openCode "src/trace_processor/importers/common/metadata_tracker.cc:46:49"
    node2 -->|"Yes"| node3["Store trace UUID for crash diagnostics"]
    click node2 openCode "src/trace_processor/importers/common/metadata_tracker.cc:52:55"
    node2 -->|"No"| node4
    node3 --> node4
    
    subgraph loop1["For each metadata entry with this name"]
      node4 --> node5{"Does entry exist?"}
      
      node5 -->|"Yes"| node6["Checking for Existing Metadata"]
      
      node5 -->|"No"| node7["Finalizing Metadata Storage"]
      
    end
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node5 goToHeading "Checking for Existing Metadata"
node5:::HeadingStyle
click node6 goToHeading "Checking for Existing Metadata"
node6:::HeadingStyle
click node7 goToHeading "Finalizing Metadata Storage"
node7:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Receive metadata key and value"] --> node2{"Is key <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="50:7:7" line-data="  // When the trace_uuid is set, store a copy in a crash key, so in case of">`trace_uuid`</SwmToken> and value a string?"}
%%     click node1 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:46:49"
%%     node2 -->|"Yes"| node3["Store trace UUID for crash diagnostics"]
%%     click node2 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:52:55"
%%     node2 -->|"No"| node4
%%     node3 --> node4
%%     
%%     subgraph loop1["For each metadata entry with this name"]
%%       node4 --> node5{"Does entry exist?"}
%%       
%%       node5 -->|"Yes"| node6["Checking for Existing Metadata"]
%%       
%%       node5 -->|"No"| node7["Finalizing Metadata Storage"]
%%       
%%     end
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node5 goToHeading "Checking for Existing Metadata"
%% node5:::HeadingStyle
%% click node6 goToHeading "Checking for Existing Metadata"
%% node6:::HeadingStyle
%% click node7 goToHeading "Finalizing Metadata Storage"
%% node7:::HeadingStyle
```

<SwmSnippet path="/src/trace_processor/importers/common/metadata_tracker.cc" line="46">

---

In <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>, we check for the <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="50:7:7" line-data="  // When the trace_uuid is set, store a copy in a crash key, so in case of">`trace_uuid`</SwmToken> key to store its value for crash diagnostics, then move on to fetch the string ID for the key name from the string pool.

```c++
MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {
  PERFETTO_DCHECK(metadata::kKeyTypes[key] == metadata::KeyType::kSingle);
  PERFETTO_DCHECK(value.type == metadata::kValueTypes[key]);

  // When the trace_uuid is set, store a copy in a crash key, so in case of
  // a crash in the pipelines we can tell which trace caused the crash.
  if (key == metadata::trace_uuid && value.type == Variadic::kString) {
    auto uuid_string_view = storage_->GetString(value.string_value);
    g_crash_key_uuid.Set(uuid_string_view);
  }

  auto& metadata_table = *storage_->mutable_metadata_table();
  auto key_idx = static_cast<uint32_t>(key);
  auto name_id = storage_->string_pool().GetId(metadata::kNames[key_idx]);
```

---

</SwmSnippet>

## Resolving String Identifiers

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Receive input string"]
  click node1 openCode "src/trace_processor/containers/string_pool.h:225:229"
  node1 --> node2{"Is input string empty?"}
  click node2 openCode "src/trace_processor/containers/string_pool.h:226:228"
  node2 -->|"Yes"| node3["Return null identifier"]
  click node3 openCode "src/trace_processor/containers/string_pool.h:227:228"
  node2 -->|"No"| node4["File and Name Matching"]
  
  node4 --> node5{"Identifier found?"}
  click node5 openCode "src/trace_processor/containers/string_pool.h:232:235"
  node5 -->|"Yes"| node6["Return identifier"]
  click node6 openCode "src/trace_processor/containers/string_pool.h:234:235"
  node5 -->|"No"| node3
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node4 goToHeading "Hashing String Data"
node4:::HeadingStyle
click node4 goToHeading "File and Name Matching"
node4:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Receive input string"]
%%   click node1 openCode "<SwmPath>[src/…/containers/string_pool.h](src/trace_processor/containers/string_pool.h)</SwmPath>:225:229"
%%   node1 --> node2{"Is input string empty?"}
%%   click node2 openCode "<SwmPath>[src/…/containers/string_pool.h](src/trace_processor/containers/string_pool.h)</SwmPath>:226:228"
%%   node2 -->|"Yes"| node3["Return null identifier"]
%%   click node3 openCode "<SwmPath>[src/…/containers/string_pool.h](src/trace_processor/containers/string_pool.h)</SwmPath>:227:228"
%%   node2 -->|"No"| node4["File and Name Matching"]
%%   
%%   node4 --> node5{"Identifier found?"}
%%   click node5 openCode "<SwmPath>[src/…/containers/string_pool.h](src/trace_processor/containers/string_pool.h)</SwmPath>:232:235"
%%   node5 -->|"Yes"| node6["Return identifier"]
%%   click node6 openCode "<SwmPath>[src/…/containers/string_pool.h](src/trace_processor/containers/string_pool.h)</SwmPath>:234:235"
%%   node5 -->|"No"| node3
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node4 goToHeading "Hashing String Data"
%% node4:::HeadingStyle
%% click node4 goToHeading "File and Name Matching"
%% node4:::HeadingStyle
```

<SwmSnippet path="/src/trace_processor/containers/string_pool.h" line="225">

---

In <SwmToken path="src/trace_processor/containers/string_pool.h" pos="225:8:8" line-data="  std::optional&lt;Id&gt; GetId(base::StringView str) const {">`GetId`</SwmToken>, we check if the string is null and return a null ID if so. Otherwise, we hash the string using <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="207:22:22" line-data="// Helper to check if a type has a built-in MurmurHash implementation.">`MurmurHash`</SwmToken> to get a value for fast lookup in the string pool. Next, we need to call the hash function to get this value.

```c
  std::optional<Id> GetId(base::StringView str) const {
    if (str.data() == nullptr) {
      return Id::Null();
    }
    auto hash = base::MurmurHashValue(str);
```

---

</SwmSnippet>

### Hashing String Data

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="327">

---

<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="327:2:2" line-data="uint64_t MurmurHashValue(const T&amp; value) {">`MurmurHashValue`</SwmToken> picks the right hashing logic based on the type of the input. If there's a <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="171:18:20" line-data="// Computes a 64-bit hash for a single built-in value without any combination.">`built-in`</SwmToken> hash, it uses that; otherwise, it falls back to combining the value. Next, we need to handle the actual hashing for the specific type, which could involve normalization for floats or direct hashing for strings.

```c
uint64_t MurmurHashValue(const T& value) {
  if constexpr (murmur_internal::HasMurmurHashBuiltinValue<T>()) {
    return murmur_internal::MurmurHashBuiltinValue(value);
  } else {
    return MurmurHashCombine(value);
  }
}
```

---

</SwmSnippet>

### Type-Specific Hash Normalization

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Receive input value"] --> node2{"What is the input type?"}
  click node1 openCode "include/perfetto/ext/base/murmur_hash.h:179:205"
  node2 -->|"Enum"| node3["Generate hash for enum"]
  click node3 openCode "include/perfetto/ext/base/murmur_hash.h:180:183"
  node2 -->|"Integer"| node4["Generate hash for integer"]
  click node4 openCode "include/perfetto/ext/base/murmur_hash.h:184:184"
  node2 -->|"Double"| node5["Normalize value if zero or NaN, then generate hash"]
  click node5 openCode "include/perfetto/ext/base/murmur_hash.h:185:188"
  node2 -->|"Float"| node6["Normalize value if zero or NaN, then generate hash"]
  click node6 openCode "include/perfetto/ext/base/murmur_hash.h:189:190"
  node2 -->|"String or String-like"| node7["Generate hash for string"]
  click node7 openCode "include/perfetto/ext/base/murmur_hash.h:191:197"
  node2 -->|"Pointer"| node8["Generate hash for pointer"]
  click node8 openCode "include/perfetto/ext/base/murmur_hash.h:198:200"
  node2 -->|"Unsupported type"| node9["Return invalid result"]
  click node9 openCode "include/perfetto/ext/base/murmur_hash.h:201:204"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Receive input value"] --> node2{"What is the input type?"}
%%   click node1 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:179:205"
%%   node2 -->|"Enum"| node3["Generate hash for enum"]
%%   click node3 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:180:183"
%%   node2 -->|"Integer"| node4["Generate hash for integer"]
%%   click node4 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:184:184"
%%   node2 -->|"Double"| node5["Normalize value if zero or <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="162:7:7" line-data="    // Turn arbtirary NaN representations to a consistent NaN repr.">`NaN`</SwmToken>, then generate hash"]
%%   click node5 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:185:188"
%%   node2 -->|"Float"| node6["Normalize value if zero or <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="162:7:7" line-data="    // Turn arbtirary NaN representations to a consistent NaN repr.">`NaN`</SwmToken>, then generate hash"]
%%   click node6 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:189:190"
%%   node2 -->|"String or String-like"| node7["Generate hash for string"]
%%   click node7 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:191:197"
%%   node2 -->|"Pointer"| node8["Generate hash for pointer"]
%%   click node8 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:198:200"
%%   node2 -->|"Unsupported type"| node9["Return invalid result"]
%%   click node9 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:201:204"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="179">

---

<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="179:2:2" line-data="auto MurmurHashBuiltinValue(const T&amp; value) {">`MurmurHashBuiltinValue`</SwmToken> normalizes and hashes values based on their type, and for floats, we need to normalize them before hashing.

```c
auto MurmurHashBuiltinValue(const T& value) {
  if constexpr (std::is_enum_v<T>) {
    return murmur_internal::MurmurHashMix(
        static_cast<uint64_t>(static_cast<std::underlying_type_t<T>>(value)));
  } else if constexpr (std::is_integral_v<T>) {
    return murmur_internal::MurmurHashMix(static_cast<uint64_t>(value));
  } else if constexpr (std::is_same_v<T, double>) {
    return murmur_internal::MurmurHashMix(
        murmur_internal::NormalizeFloatToInt<double, uint64_t>(value));
  } else if constexpr (std::is_same_v<T, float>) {
    return murmur_internal::MurmurHashMix(
        murmur_internal::NormalizeFloatToInt<float, uint32_t>(value));
```

---

</SwmSnippet>

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="153">

---

<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="153:2:2" line-data="Int NormalizeFloatToInt(Float value) {">`NormalizeFloatToInt`</SwmToken> normalizes special float values and copies their bit pattern into an integer for consistent hashing.

```c
Int NormalizeFloatToInt(Float value) {
  static_assert(std::is_floating_point_v<Float>);
  static_assert(std::is_integral_v<Int>);

  // Normalize floating point representations which can vary.
  if (PERFETTO_UNLIKELY(value == 0.0)) {
    // Turn negative zero into positive zero
    value = 0.0;
  } else if (PERFETTO_UNLIKELY(std::isnan(value))) {
    // Turn arbtirary NaN representations to a consistent NaN repr.
    value = std::numeric_limits<Float>::quiet_NaN();
  }
  Int res;
  static_assert(sizeof(Float) == sizeof(Int));
  memcpy(&res, &value, sizeof(Float));
  return res;
}
```

---

</SwmSnippet>

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="191">

---

Back in <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="179:2:2" line-data="auto MurmurHashBuiltinValue(const T&amp; value) {">`MurmurHashBuiltinValue`</SwmToken>, after handling floats, we handle strings by hashing their bytes and pointers by hashing their address. If the type isn't supported, we return an invalid marker.

```c
  } else if constexpr (std::is_same_v<T, std::string> ||
                       std::is_same_v<T, std::string_view> ||
                       std::is_same_v<T, base::StringView>) {
    return murmur_internal::MurmurHashBytes(value.data(), value.size());
  } else if constexpr (std::is_same_v<T, const char*>) {
    std::string_view view(value);
    return murmur_internal::MurmurHashBytes(view.data(), view.size());
  } else if constexpr (std::is_pointer_v<T>) {
    return murmur_internal::MurmurHashMix(
        static_cast<uint64_t>(reinterpret_cast<uintptr_t>(value)));
  } else {
    struct InvalidBuiltin {};
    return InvalidBuiltin{};
  }
}
```

---

</SwmSnippet>

### String Pool Lookup by Hash

<SwmSnippet path="/src/trace_processor/containers/string_pool.h" line="230">

---

Back in <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="59:13:13" line-data="  auto name_id = storage_-&gt;string_pool().GetId(metadata::kNames[key_idx]);">`GetId`</SwmToken>, after hashing, we lock the string pool if needed and look up the hash in the string index. If the string isn't found, we need to check for its existence in another structure, which is why we call the next function.

```c
    MaybeLockGuard guard{mutex_, should_acquire_mutex_};
    Id* id = string_index_.Find(hash);
```

---

</SwmSnippet>

### File and Name Matching

<SwmSnippet path="/src/trace_processor/util/zip_reader.cc" line="328">

---

<SwmToken path="src/trace_processor/util/zip_reader.cc" pos="328:3:5" line-data="ZipFile* ZipReader::Find(const std::string&amp; path) {">`ZipReader::Find`</SwmToken> loops through all files and compares each file's name to the target path. If it matches, we return the file. To get the name, we call the name() function, which is implemented elsewhere.

```c++
ZipFile* ZipReader::Find(const std::string& path) {
  for (ZipFile& zf : files_) {
    if (zf.name() == path)
      return &zf;
  }
  return nullptr;
}
```

---

</SwmSnippet>

<SwmSnippet path="/src/trace_processor/util/sql_argument.h" line="59">

---

<SwmToken path="src/trace_processor/util/sql_argument.h" pos="59:3:5" line-data="  NullTermStringView name() const {">`name()`</SwmToken> returns a substring view of <SwmToken path="src/trace_processor/util/sql_argument.h" pos="60:5:5" line-data="    return NullTermStringView(dollar_name_.c_str() + 1,">`dollar_name_`</SwmToken> starting from the second character, skipping the first (probably a special marker). There's no check for empty strings, so <SwmToken path="src/trace_processor/util/sql_argument.h" pos="60:5:5" line-data="    return NullTermStringView(dollar_name_.c_str() + 1,">`dollar_name_`</SwmToken> must always have at least one character.

```c
  NullTermStringView name() const {
    return NullTermStringView(dollar_name_.c_str() + 1,
                              dollar_name_.size() - 1);
  }
```

---

</SwmSnippet>

### Returning String <SwmToken path="src/trace_processor/containers/string_pool.h" pos="48:5:5" line-data="  // StringPool IDs are 32-bit. If the MSB is 1, the remaining bits of the ID">`IDs`</SwmToken>

<SwmSnippet path="/src/trace_processor/containers/string_pool.h" line="232">

---

Back in <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="59:13:13" line-data="  auto name_id = storage_-&gt;string_pool().GetId(metadata::kNames[key_idx]);">`GetId`</SwmToken>, if we find the string's ID in the index, we double-check that the ID maps to the right string, then return it. If not found, we return std::nullopt to signal the string isn't in the pool.

```c
    if (id) {
      PERFETTO_DCHECK(Get(*id) == str);
      return *id;
    }
    return std::nullopt;
  }
```

---

</SwmSnippet>

## Checking for Existing Metadata

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1{"Is metadata name provided?"}
    click node1 openCode "src/trace_processor/importers/common/metadata_tracker.cc:60:66"
    node1 -->|"Yes"| node2["Iterate over metadata entries"]
    click node2 openCode "src/trace_processor/importers/common/metadata_tracker.cc:61:66"
    subgraph loop1["For each metadata entry"]
      node2 --> node3{"Does entry name match metadata name?"}
      click node3 openCode "src/trace_processor/importers/common/metadata_tracker.cc:62:63"
      node3 -->|"Yes"| node4["Update metadata value"]
      click node4 openCode "src/trace_processor/importers/common/metadata_tracker.cc:63:64"
      node4 --> node5["Return entry"]
      click node5 openCode "src/trace_processor/importers/common/metadata_tracker.cc:64:65"
      node3 -->|"No match after all entries"| node6["Create new metadata entry"]
      click node6 openCode "src/trace_processor/importers/common/metadata_tracker.cc:69:73"
      node6 --> node5
    end
    node1 -->|"No"| node6

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1{"Is metadata name provided?"}
%%     click node1 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:60:66"
%%     node1 -->|"Yes"| node2["Iterate over metadata entries"]
%%     click node2 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:61:66"
%%     subgraph loop1["For each metadata entry"]
%%       node2 --> node3{"Does entry name match metadata name?"}
%%       click node3 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:62:63"
%%       node3 -->|"Yes"| node4["Update metadata value"]
%%       click node4 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:63:64"
%%       node4 --> node5["Return entry"]
%%       click node5 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:64:65"
%%       node3 -->|"No match after all entries"| node6["Create new metadata entry"]
%%       click node6 openCode "<SwmPath>[src/…/common/metadata_tracker.cc](src/trace_processor/importers/common/metadata_tracker.cc)</SwmPath>:69:73"
%%       node6 --> node5
%%     end
%%     node1 -->|"No"| node6
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/trace_processor/importers/common/metadata_tracker.cc" line="60">

---

Back in <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>, after getting the string ID, we scan the metadata table for an existing entry with the same name. If found, we update its value and return its ID, preventing duplicates.

```c++
  if (name_id) {
    for (auto it = metadata_table.IterateRows(); it; ++it) {
      if (it.name() == *name_id) {
        WriteValue(it.row_number().row_number(), value);
        return it.id();
      }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/trace_processor/importers/common/metadata_tracker.cc" line="69">

---

If no entry exists, we create a new row and insert it into the metadata table's tree.

```c++
  tables::MetadataTable::Row row;
  row.name = key_ids_[key_idx];
  row.key_type = key_type_ids_[static_cast<size_t>(metadata::KeyType::kSingle)];

  auto id_and_row = metadata_table.Insert(row);
```

---

</SwmSnippet>

## Inserting into the Metadata Table

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Start: Want to insert entry into tree"]
  click node1 openCode "src/base/intrusive_tree.h:141:149"
  subgraph loop1["While searching for position in tree"]
    node1 --> node2{"Is key already present?"}
    click node2 openCode "src/base/intrusive_tree.h:149:157"
    node2 -->|"Yes"| node3["Return: Entry already exists, do not insert"]
    click node3 openCode "src/base/intrusive_tree.h:157:158"
    node2 -->|"No"| node4{"Go left or right?"}
    click node4 openCode "src/base/intrusive_tree.h:151:156"
    node4 -->|"Left"| node2
    node4 -->|"Right"| node2
    node4 -->|"Found position"| node5["Insert entry into tree"]
    click node5 openCode "src/base/intrusive_tree.h:160:173"
  end
  node5 --> node6["Update tree balance"]
  click node6 openCode "src/base/intrusive_tree.h:174:175"
  node6 --> node7["Return: Entry inserted"]
  click node7 openCode "src/base/intrusive_tree.h:176:177"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Start: Want to insert entry into tree"]
%%   click node1 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:141:149"
%%   subgraph loop1["While searching for position in tree"]
%%     node1 --> node2{"Is key already present?"}
%%     click node2 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:149:157"
%%     node2 -->|"Yes"| node3["Return: Entry already exists, do not insert"]
%%     click node3 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:157:158"
%%     node2 -->|"No"| node4{"Go left or right?"}
%%     click node4 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:151:156"
%%     node4 -->|"Left"| node2
%%     node4 -->|"Right"| node2
%%     node4 -->|"Found position"| node5["Insert entry into tree"]
%%     click node5 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:160:173"
%%   end
%%   node5 --> node6["Update tree balance"]
%%   click node6 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:174:175"
%%   node6 --> node7["Return: Entry inserted"]
%%   click node7 openCode "<SwmPath>[src/base/intrusive_tree.h](src/base/intrusive_tree.h)</SwmPath>:176:177"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/base/intrusive_tree.h" line="141">

---

In <SwmToken path="src/base/intrusive_tree.h" pos="141:11:11" line-data="  std::pair&lt;Iterator, bool&gt; Insert(T&amp; entry) {">`Insert`</SwmToken>, we walk the red-black tree to find where to put the new entry. If the key exists, we return the existing node. Otherwise, we link the new node, color it red, and rebalance the tree. The nodeof(&entry) trick is used to get the internal tree node from the entry struct.

```c
  std::pair<Iterator, bool> Insert(T& entry) {
    // The insertion preamble is inlined because it's few instructions and
    // out-lining it would require std::function indirections for getting the
    // key and the comparator.
    int comp = 0;
    internal::RBNode* tmp = root_;
    internal::RBNode* parent = nullptr;
    internal::RBNode* const entry_node = nodeof(&entry);
    while (tmp) {
      parent = tmp;
      comp = key_compare(entry_node, parent);
      if (comp < 0) {
        tmp = tmp->left;
      } else if (comp > 0) {
        tmp = tmp->right;
      } else {
        return {Iterator(tmp), false};  // The key exists already.
      }
    }  // while(tmp)
```

---

</SwmSnippet>

<SwmSnippet path="/src/base/intrusive_tree.h" line="159">

---

After inserting, we return an iterator to the node and a flag showing if it was a new insert. The tree is balanced as needed, so lookups stay fast.

```c
    }  // while(tmp)
    entry_node->left = entry_node->right = nullptr;
    entry_node->parent = parent;
    entry_node->color = internal::RBColor::RED;
    if (parent) {
      if (comp < 0) {
        PERFETTO_DCHECK(parent->left == nullptr);
        parent->left = entry_node;
      } else {
        PERFETTO_DCHECK(parent->right == nullptr);
        parent->right = entry_node;
      }
    } else {
      root_ = entry_node;
    }
    internal::RBInsertColor(&root_, entry_node);
    ++size_;
    return {Iterator(entry_node), true};
  }
```

---

</SwmSnippet>

## Finalizing Metadata Storage

<SwmSnippet path="/src/trace_processor/importers/common/metadata_tracker.cc" line="74">

---

Back in <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:4:4" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`SetMetadata`</SwmToken>, after inserting the new row, we write the value to it and return the new <SwmToken path="src/trace_processor/importers/common/metadata_tracker.cc" pos="46:0:0" line-data="MetadataId MetadataTracker::SetMetadata(metadata::KeyId key, Variadic value) {">`MetadataId`</SwmToken>. This wraps up the process of adding new metadata.

```c++
  WriteValue(id_and_row.row, value);
  return id_and_row.id;
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
