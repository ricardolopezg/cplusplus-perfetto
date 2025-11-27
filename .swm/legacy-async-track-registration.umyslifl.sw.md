---
title: Legacy Async Track Registration
---
This document describes how legacy asynchronous tracks are registered and uniquely identified for trace analysis. The flow supports both process-scoped and global async tracks, handling different event types. Upon receiving a registration request, the system determines the track's scope, adapts the track name if needed, constructs a blueprint, and generates a unique identifier for trace analysis.

```mermaid
flowchart TD
  node1["Async Track Interning Entry Point"]:::HeadingStyle
  click node1 goToHeading "Async Track Interning Entry Point"
  node1 --> node2{"Process-scoped or global?"}
  node2 -->|"Process-scoped"| node3["Process-Scoped Track Interning"]:::HeadingStyle
  click node3 goToHeading "Process-Scoped Track Interning"
  node2 -->|"Global"| node4["Global Track Interning"]:::HeadingStyle
  click node4 goToHeading "Global Track Interning"
  node3 --> node5["Global Track Interning Finalization"]:::HeadingStyle
  click node5 goToHeading "Global Track Interning Finalization"
  node4 --> node5
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

# Where is this flow used?

This flow is used multiple times in the codebase as represented in the following diagram:

```mermaid
graph TD;
      8e22a1ebd7c49013e82f59a480bf4f2340b475310ef2cfc4fe29c22e150347fd(src/…/fuchsia/fuchsia_trace_parser.cc::Parse) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(src/…/common/track_compressor.cc::InternLegacyAsyncTrack):::mainFlowStyle

5e78994235142c4444fdedd929ed00023c9ef4c7e9132e37d45546c09e77277b(src/…/json/json_trace_parser.cc::ParseJsonPacket) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(src/…/common/track_compressor.cc::InternLegacyAsyncTrack):::mainFlowStyle

c61ea289f838ed148142490cd7253014813015e78ccce14d7ccba546b9986818(src/…/proto/track_event_event_importer.h::ParseTrackAssociationInternal) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(src/…/common/track_compressor.cc::InternLegacyAsyncTrack):::mainFlowStyle


classDef mainFlowStyle color:#000000,fill:#7CB9F4
classDef rootsStyle color:#000000,fill:#00FFF4
classDef Style1 color:#000000,fill:#00FFAA
classDef Style2 color:#000000,fill:#FFFF00
classDef Style3 color:#000000,fill:#AA7CB9

%% Swimm:
%% graph TD;
%%       8e22a1ebd7c49013e82f59a480bf4f2340b475310ef2cfc4fe29c22e150347fd(<SwmPath>[src/…/fuchsia/fuchsia_trace_parser.cc](src/trace_processor/importers/fuchsia/fuchsia_trace_parser.cc)</SwmPath>::Parse) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken>):::mainFlowStyle
%% 
%% 5e78994235142c4444fdedd929ed00023c9ef4c7e9132e37d45546c09e77277b(<SwmPath>[src/…/json/json_trace_parser.cc](src/trace_processor/importers/json/json_trace_parser.cc)</SwmPath>::ParseJsonPacket) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken>):::mainFlowStyle
%% 
%% c61ea289f838ed148142490cd7253014813015e78ccce14d7ccba546b9986818(<SwmPath>[src/…/proto/track_event_event_importer.h](src/trace_processor/importers/proto/track_event_event_importer.h)</SwmPath>::ParseTrackAssociationInternal) --> d0ee9a83d1815d621820739ff846a021c63f53871b9fe901e9d7da21ec370004(<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>::<SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken>):::mainFlowStyle
%% 
%% 
%% classDef mainFlowStyle color:#000000,fill:#7CB9F4
%% classDef rootsStyle color:#000000,fill:#00FFF4
%% classDef Style1 color:#000000,fill:#00FFAA
%% classDef Style2 color:#000000,fill:#FFFF00
%% classDef Style3 color:#000000,fill:#AA7CB9
```

# Async Track Interning Entry Point

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Receive async track registration request"]
  click node1 openCode "src/trace_processor/importers/common/track_compressor.cc:137:142"
  node1 --> node2{"Is trace process-scoped?"}
  click node2 openCode "src/trace_processor/importers/common/track_compressor.cc:150:151"
  node2 -->|"Yes"| node3{"Async slice type?"}
  
  node2 -->|"No"| node4{"Async slice type?"}
  
  node3 -->|"Begin"| node5["Process-Scoped Track Interning"]
  
  node3 -->|"End"| node6["Process-Scoped Track Interning"]
  
  node3 -->|"Instant"| node7["Process-Scoped Track Interning"]
  
  node4 -->|"Begin"| node8["Global Track Interning Finalization"]
  
  node4 -->|"End"| node6
  node4 -->|"Instant"| node7

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
click node3 goToHeading "Process-Scoped Track Interning"
node3:::HeadingStyle
click node4 goToHeading "Global Track Interning Finalization"
node4:::HeadingStyle
click node5 goToHeading "Process-Scoped Track Interning"
node5:::HeadingStyle
click node6 goToHeading "Process-Scoped Track Interning"
node6:::HeadingStyle
click node7 goToHeading "Process-Scoped Track Interning"
node7:::HeadingStyle
click node8 goToHeading "Global Track Interning Finalization"
node8:::HeadingStyle

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Receive async track registration request"]
%%   click node1 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:137:142"
%%   node1 --> node2{"Is trace process-scoped?"}
%%   click node2 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:150:151"
%%   node2 -->|"Yes"| node3{"Async slice type?"}
%%   
%%   node2 -->|"No"| node4{"Async slice type?"}
%%   
%%   node3 -->|"Begin"| node5["Process-Scoped Track Interning"]
%%   
%%   node3 -->|"End"| node6["Process-Scoped Track Interning"]
%%   
%%   node3 -->|"Instant"| node7["Process-Scoped Track Interning"]
%%   
%%   node4 -->|"Begin"| node8["Global Track Interning Finalization"]
%%   
%%   node4 -->|"End"| node6
%%   node4 -->|"Instant"| node7
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
%% click node3 goToHeading "Process-Scoped Track Interning"
%% node3:::HeadingStyle
%% click node4 goToHeading "Global Track Interning Finalization"
%% node4:::HeadingStyle
%% click node5 goToHeading "Process-Scoped Track Interning"
%% node5:::HeadingStyle
%% click node6 goToHeading "Process-Scoped Track Interning"
%% node6:::HeadingStyle
%% click node7 goToHeading "Process-Scoped Track Interning"
%% node7:::HeadingStyle
%% click node8 goToHeading "Global Track Interning Finalization"
%% node8:::HeadingStyle
```

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.cc" line="137">

---

We start by prepping metadata for the track and, if we're dealing with process-scoped tracks, we translate the name to make sure it's unique per process before moving on.

```c++
TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,
                                                uint32_t upid,
                                                int64_t trace_id,
                                                bool trace_id_is_process_scoped,
                                                StringId source_scope,
                                                AsyncSliceType slice_type) {
  auto args_fn = [&](ArgsTracker::BoundInserter& inserter) {
    inserter.AddArg(source_key_, Variadic::String(chrome_source_))
        .AddArg(trace_id_is_process_scoped_key_,
                Variadic::Boolean(trace_id_is_process_scoped))
        .AddArg(upid_, Variadic::UnsignedInteger(upid))
        .AddArg(source_scope_key_, Variadic::String(source_scope));
  };
  if (trace_id_is_process_scoped) {
    const StringId name =
        context_->process_track_translation_table->TranslateName(raw_name);
```

---

</SwmSnippet>

## Track Name Translation

<SwmSnippet path="/src/trace_processor/importers/common/process_track_translation_table.h" line="36">

---

<SwmToken path="src/trace_processor/importers/common/process_track_translation_table.h" pos="36:3:3" line-data="  StringId TranslateName(StringId raw_name) const {">`TranslateName`</SwmToken> checks if the <SwmToken path="src/trace_processor/importers/common/process_track_translation_table.h" pos="36:7:7" line-data="  StringId TranslateName(StringId raw_name) const {">`raw_name`</SwmToken> has a mapped (deobfuscated) version in the table. If it does, we use that; if not, we stick with the original. Next, we might need to look up address ranges to resolve further context for the track.

```c
  StringId TranslateName(StringId raw_name) const {
    const auto* mapped_name = raw_to_deobfuscated_name_.Find(raw_name);
    return mapped_name ? *mapped_name : raw_name;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/trace_processor/importers/common/address_range.h" line="255">

---

<SwmToken path="src/trace_processor/importers/common/address_range.h" pos="255:3:3" line-data="  iterator Find(uint64_t address) {">`Find`</SwmToken> uses <SwmToken path="src/trace_processor/importers/common/address_range.h" pos="256:9:9" line-data="    auto it = ranges_.upper_bound(address);">`upper_bound`</SwmToken> to locate the first range with a start greater than the address, then checks if the address is inside that range. This is a fast way to see if an address matches any known range.

```c
  iterator Find(uint64_t address) {
    auto it = ranges_.upper_bound(address);
    if (it != ranges_.end() && address >= it->first.start()) {
      return it;
    }
    return end();
  }
```

---

</SwmSnippet>

## Process-Scoped Track Interning

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Receive async operation details (name, upid, trace_id)"] --> node2{"Type of async slice?"}
    click node1 openCode "src/trace_processor/importers/common/track_compressor.cc:153:160"
    node2 -->|"Begin"| node3["Record operation start in trace"]
    click node2 openCode "src/trace_processor/importers/common/track_compressor.cc:161:180"
    node2 -->|"End"| node4["Record operation end in trace"]
    node2 -->|"Instant"| node5["Record both start and end as instant in trace"]
    node5 --> node7{"Are start and end trace entries equal?"}
    node7 -->|"Yes"| node6["Return trace entry"]
    node3 --> node6["Trace entry created"]
    node4 --> node6
    node7 -->|"No"| node8["Report inconsistency"]
    click node3 openCode "src/trace_processor/importers/common/track_compressor.cc:163:165"
    click node4 openCode "src/trace_processor/importers/common/track_compressor.cc:167:169"
    click node5 openCode "src/trace_processor/importers/common/track_compressor.cc:171:176"
    click node7 openCode "src/trace_processor/importers/common/track_compressor.cc:177:178"
    click node6 openCode "src/trace_processor/importers/common/track_compressor.cc:178:179"
    click node8 openCode "src/trace_processor/importers/common/track_compressor.cc:181:181"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%     node1["Receive async operation details (name, upid, <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="139:3:3" line-data="                                                int64_t trace_id,">`trace_id`</SwmToken>)"] --> node2{"Type of async slice?"}
%%     click node1 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:153:160"
%%     node2 -->|"Begin"| node3["Record operation start in trace"]
%%     click node2 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:161:180"
%%     node2 -->|"End"| node4["Record operation end in trace"]
%%     node2 -->|"Instant"| node5["Record both start and end as instant in trace"]
%%     node5 --> node7{"Are start and end trace entries equal?"}
%%     node7 -->|"Yes"| node6["Return trace entry"]
%%     node3 --> node6["Trace entry created"]
%%     node4 --> node6
%%     node7 -->|"No"| node8["Report inconsistency"]
%%     click node3 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:163:165"
%%     click node4 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:167:169"
%%     click node5 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:171:176"
%%     click node7 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:177:178"
%%     click node6 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:178:179"
%%     click node8 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:181:181"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.cc" line="153">

---

Back in <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken> after translating the name, we set up the blueprint for process-scoped tracks, hash upid and <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="160:8:8" line-data="        base::MurmurHashCombine(upid, trace_id), name);">`trace_id`</SwmToken>, and insert into the map. Depending on <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="161:4:4" line-data="    switch (slice_type) {">`slice_type`</SwmToken>, we call <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="163:3:3" line-data="        return InternBegin(kBlueprint,">`InternBegin`</SwmToken>, <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="167:3:3" line-data="        return InternEnd(kBlueprint,">`InternEnd`</SwmToken>, or both for instant slices. Next, we need to build the blueprint using <SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath> to define the track's structure.

```c++
    static constexpr auto kBlueprint = TrackCompressor::SliceBlueprint(
        "legacy_async_process_slice",
        tracks::DimensionBlueprints(tracks::kProcessDimensionBlueprint,
                                    tracks::StringIdDimensionBlueprint("scope"),
                                    tracks::StringIdDimensionBlueprint("name")),
        tracks::DynamicNameBlueprint());
    auto [it, inserted] = async_tracks_to_root_string_id_.Insert(
        base::MurmurHashCombine(upid, trace_id), name);
    switch (slice_type) {
      case AsyncSliceType::kBegin:
        return InternBegin(kBlueprint,
                           tracks::Dimensions(upid, source_scope, *it),
                           trace_id, tracks::DynamicName(name), args_fn);
      case AsyncSliceType::kEnd:
        return InternEnd(kBlueprint,
                         tracks::Dimensions(upid, source_scope, *it), trace_id,
                         tracks::DynamicName(name), args_fn);
      case AsyncSliceType::kInstant: {
        TrackId begin =
            InternBegin(kBlueprint, tracks::Dimensions(upid, source_scope, *it),
                        trace_id, tracks::DynamicName(name), args_fn);
        TrackId end =
            InternEnd(kBlueprint, tracks::Dimensions(upid, source_scope, *it),
                      trace_id, tracks::DynamicName(name), args_fn);
        PERFETTO_DCHECK(begin == end);
        return begin;
      }
    }
    PERFETTO_FATAL("For GCC");
  }
  static constexpr auto kBlueprint = TrackCompressor::SliceBlueprint(
      "legacy_async_global_slice",
      tracks::DimensionBlueprints(tracks::StringIdDimensionBlueprint("scope"),
                                  tracks::StringIdDimensionBlueprint("name")),
      tracks::DynamicNameBlueprint());
```

---

</SwmSnippet>

## Track Blueprint Construction

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Create base slice blueprint with type, dimensions, and name"]
  click node1 openCode "src/trace_processor/importers/common/track_compressor.h:181:182"
  node1 --> node2["Add compressor index dimension to blueprint"]
  click node2 openCode "src/trace_processor/importers/common/track_compressor.h:187:189"
  node2 --> node3{"Is custom naming logic present?"}
  click node3 openCode "src/trace_processor/importers/common/track_compressor.h:191:192"
  node3 -->|"Yes"| node4["Adapt naming logic using MakeNameFn"]
  click node4 openCode "src/trace_processor/importers/common/track_compressor.h:194:195"
  node3 -->|"No"| node5["Keep default naming logic"]
  click node5 openCode "src/trace_processor/importers/common/track_compressor.h:211:224"
  node4 --> node6["Return completed slice blueprint (with compressor index)"]
  click node6 openCode "src/trace_processor/importers/common/track_compressor.h:196:210"
  node5 --> node6
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Create base slice blueprint with type, dimensions, and name"]
%%   click node1 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:181:182"
%%   node1 --> node2["Add compressor index dimension to blueprint"]
%%   click node2 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:187:189"
%%   node2 --> node3{"Is custom naming logic present?"}
%%   click node3 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:191:192"
%%   node3 -->|"Yes"| node4["Adapt naming logic using <SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="195:1:1" line-data="                MakeNameFn&lt;F, decltype(x)...&gt;(blueprint.name_blueprint.fn);">`MakeNameFn`</SwmToken>"]
%%   click node4 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:194:195"
%%   node3 -->|"No"| node5["Keep default naming logic"]
%%   click node5 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:211:224"
%%   node4 --> node6["Return completed slice blueprint (with compressor index)"]
%%   click node6 openCode "<SwmPath>[src/…/common/track_compressor.h](src/trace_processor/importers/common/track_compressor.h)</SwmPath>:196:210"
%%   node5 --> node6
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.h" line="177">

---

<SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="177:7:7" line-data="  static constexpr auto SliceBlueprint(">`SliceBlueprint`</SwmToken> builds the blueprint for a track, adding a <SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="189:6:6" line-data="              tracks::UintDimensionBlueprint(&quot;track_compressor_idx&quot;);">`track_compressor_idx`</SwmToken> dimension for uniqueness. If the name blueprint is a function, it wraps it for compatibility. Next, <SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="195:1:1" line-data="                MakeNameFn&lt;F, decltype(x)...&gt;(blueprint.name_blueprint.fn);">`MakeNameFn`</SwmToken> adapts the name function for the blueprint.

```c
  static constexpr auto SliceBlueprint(
      const char type[],
      tracks::DimensionBlueprintsT<D...> dimensions = {},
      NB name = NB{}) {
    auto blueprint = tracks::SliceBlueprint(type, dimensions, name);
    using BT = decltype(blueprint);
    constexpr auto kCompressorIdxDimensionIndex =
        std::tuple_size_v<typename BT::dimension_blueprints_t>;
    return std::apply(
        [&](auto... x) {
          auto blueprints = blueprint.dimension_blueprints;
          blueprints[kCompressorIdxDimensionIndex] =
              tracks::UintDimensionBlueprint("track_compressor_idx");

          if constexpr (std::is_base_of_v<tracks::NameBlueprintT::FnBase,
                                          typename BT::name_blueprint_t>) {
            using F = decltype(blueprint.name_blueprint.fn);
            auto fn =
                MakeNameFn<F, decltype(x)...>(blueprint.name_blueprint.fn);
            return tracks::BlueprintT<
                decltype(fn), typename BT::unit_blueprint_t,
                typename BT::description_blueprint_t, decltype(x)...,
                tracks::DimensionBlueprintT<uint32_t>>{
                {
                    blueprint.event_type,
                    blueprint.type,
                    blueprint.hasher,
                    blueprints,
                },
                fn,
                blueprint.unit_blueprint,
                blueprint.description_blueprint,
            };
          } else {
            return tracks::BlueprintT<
                typename BT::name_blueprint_t, typename BT::unit_blueprint_t,
                typename BT::description_blueprint_t, decltype(x)...,
                tracks::DimensionBlueprintT<uint32_t>>{
                {
                    blueprint.event_type,
                    blueprint.type,
                    blueprint.hasher,
                    blueprints,
                },
                blueprint.name_blueprint,
                blueprint.unit_blueprint,
                blueprint.description_blueprint,
            };
          }
        },
        typename BT::dimension_blueprints_t());
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.h" line="374">

---

<SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="374:7:7" line-data="  static constexpr auto MakeNameFn(F fn) {">`MakeNameFn`</SwmToken> wraps the name function in a lambda that matches the blueprint's expected signature by adding an unused <SwmToken path="src/trace_processor/importers/common/track_compressor.h" pos="375:21:21" line-data="    auto f = [fn](typename T::type... y, uint32_t) { return fn(y...); };">`uint32_t`</SwmToken> parameter. This keeps things compatible with the blueprint logic.

```c
  static constexpr auto MakeNameFn(F fn) {
    auto f = [fn](typename T::type... y, uint32_t) { return fn(y...); };
    return tracks::NameBlueprintT::Fn<decltype(f)>{{}, f};
  }
```

---

</SwmSnippet>

## Global Track Interning

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.cc" line="188">

---

After building the blueprint for global tracks in <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken>, we hash the <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="189:5:5" line-data="      base::MurmurHashValue(trace_id), raw_name);">`trace_id`</SwmToken> using <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="207:22:22" line-data="// Helper to check if a type has a built-in MurmurHash implementation.">`MurmurHash`</SwmToken> and insert it with <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="189:9:9" line-data="      base::MurmurHashValue(trace_id), raw_name);">`raw_name`</SwmToken> into the map. Next, we call <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="207:22:22" line-data="// Helper to check if a type has a built-in MurmurHash implementation.">`MurmurHash`</SwmToken> to get a unique track ID.

```c++
  auto [it, inserted] = async_tracks_to_root_string_id_.Insert(
      base::MurmurHashValue(trace_id), raw_name);
```

---

</SwmSnippet>

## Track ID Hashing

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Receive input value"] --> node2{"Does input type have built-in hash?"}
  click node1 openCode "include/perfetto/ext/base/murmur_hash.h:327:328"
  node2 -->|"Yes"| node3["Generate unique hash using built-in method"]
  click node2 openCode "include/perfetto/ext/base/murmur_hash.h:328:329"
  click node3 openCode "include/perfetto/ext/base/murmur_hash.h:329:330"
  node2 -->|"No"| node4["Generate unique hash using generic method"]
  click node4 openCode "include/perfetto/ext/base/murmur_hash.h:331:332"
  node3 --> node5["Return hash value"]
  node4 --> node5
  click node5 openCode "include/perfetto/ext/base/murmur_hash.h:329:332"

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Receive input value"] --> node2{"Does input type have <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="171:18:20" line-data="// Computes a 64-bit hash for a single built-in value without any combination.">`built-in`</SwmToken> hash?"}
%%   click node1 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:327:328"
%%   node2 -->|"Yes"| node3["Generate unique hash using <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="171:18:20" line-data="// Computes a 64-bit hash for a single built-in value without any combination.">`built-in`</SwmToken> method"]
%%   click node2 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:328:329"
%%   click node3 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:329:330"
%%   node2 -->|"No"| node4["Generate unique hash using generic method"]
%%   click node4 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:331:332"
%%   node3 --> node5["Return hash value"]
%%   node4 --> node5
%%   click node5 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:329:332"
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="327">

---

<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="327:2:2" line-data="uint64_t MurmurHashValue(const T&amp; value) {">`MurmurHashValue`</SwmToken> picks the hashing method based on the type—builtin if available, otherwise it combines the value. Next, we handle special cases like floats and enums in the hashing logic.

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

## Type-Specific Hashing

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Determine value type for hashing"] --> node2{"Is value an enum?"}
  click node1 openCode "include/perfetto/ext/base/murmur_hash.h:179:205"
  node2 -->|"Yes"| node3["Hash as integer (enum underlying value)"]
  click node2 openCode "include/perfetto/ext/base/murmur_hash.h:180:183"
  click node3 openCode "include/perfetto/ext/base/murmur_hash.h:182:182"
  node2 -->|"No"| node4{"Is value an integer?"}
  click node4 openCode "include/perfetto/ext/base/murmur_hash.h:183:184"
  node4 -->|"Yes"| node5["Hash as integer"]
  click node5 openCode "include/perfetto/ext/base/murmur_hash.h:184:184"
  node4 -->|"No"| node6{"Is value a double?"}
  click node6 openCode "include/perfetto/ext/base/murmur_hash.h:185:187"
  node6 -->|"Yes"| node7["Normalize double for consistent hashing, then hash"]
  click node7 openCode "include/perfetto/ext/base/murmur_hash.h:186:187"
  node6 -->|"No"| node8{"Is value a float?"}
  click node8 openCode "include/perfetto/ext/base/murmur_hash.h:188:190"
  node8 -->|"Yes"| node9["Normalize float for consistent hashing, then hash"]
  click node9 openCode "include/perfetto/ext/base/murmur_hash.h:189:190"
  node8 -->|"No"| node10{"Is value a string, string_view, or base::StringView?"}
  click node10 openCode "include/perfetto/ext/base/murmur_hash.h:191:194"
  node10 -->|"Yes"| node11["Hash as string bytes"]
  click node11 openCode "include/perfetto/ext/base/murmur_hash.h:194:194"
  node10 -->|"No"| node12{"Is value a const char*?"}
  click node12 openCode "include/perfetto/ext/base/murmur_hash.h:195:197"
  node12 -->|"Yes"| node13["Hash as string bytes"]
  click node13 openCode "include/perfetto/ext/base/murmur_hash.h:197:197"
  node12 -->|"No"| node14{"Is value a pointer?"}
  click node14 openCode "include/perfetto/ext/base/murmur_hash.h:198:200"
  node14 -->|"Yes"| node15["Hash as pointer address"]
  click node15 openCode "include/perfetto/ext/base/murmur_hash.h:199:200"
  node14 -->|"No"| node16["Return invalid result for unsupported type"]
  click node16 openCode "include/perfetto/ext/base/murmur_hash.h:201:204"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Determine value type for hashing"] --> node2{"Is value an enum?"}
%%   click node1 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:179:205"
%%   node2 -->|"Yes"| node3["Hash as integer (enum underlying value)"]
%%   click node2 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:180:183"
%%   click node3 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:182:182"
%%   node2 -->|"No"| node4{"Is value an integer?"}
%%   click node4 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:183:184"
%%   node4 -->|"Yes"| node5["Hash as integer"]
%%   click node5 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:184:184"
%%   node4 -->|"No"| node6{"Is value a double?"}
%%   click node6 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:185:187"
%%   node6 -->|"Yes"| node7["Normalize double for consistent hashing, then hash"]
%%   click node7 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:186:187"
%%   node6 -->|"No"| node8{"Is value a float?"}
%%   click node8 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:188:190"
%%   node8 -->|"Yes"| node9["Normalize float for consistent hashing, then hash"]
%%   click node9 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:189:190"
%%   node8 -->|"No"| node10{"Is value a string, <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="192:10:10" line-data="                       std::is_same_v&lt;T, std::string_view&gt; ||">`string_view`</SwmToken>, or base::<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="193:10:10" line-data="                       std::is_same_v&lt;T, base::StringView&gt;) {">`StringView`</SwmToken>?"}
%%   click node10 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:191:194"
%%   node10 -->|"Yes"| node11["Hash as string bytes"]
%%   click node11 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:194:194"
%%   node10 -->|"No"| node12{"Is value a const char*?"}
%%   click node12 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:195:197"
%%   node12 -->|"Yes"| node13["Hash as string bytes"]
%%   click node13 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:197:197"
%%   node12 -->|"No"| node14{"Is value a pointer?"}
%%   click node14 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:198:200"
%%   node14 -->|"Yes"| node15["Hash as pointer address"]
%%   click node15 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:199:200"
%%   node14 -->|"No"| node16["Return invalid result for unsupported type"]
%%   click node16 openCode "<SwmPath>[include/…/base/murmur_hash.h](include/perfetto/ext/base/murmur_hash.h)</SwmPath>:201:204"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/include/perfetto/ext/base/murmur_hash.h" line="179">

---

In <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="179:2:2" line-data="auto MurmurHashBuiltinValue(const T&amp; value) {">`MurmurHashBuiltinValue`</SwmToken>, we hash enums, ints, and floats differently. For floats, we normalize special cases before hashing. Next, <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="187:3:3" line-data="        murmur_internal::NormalizeFloatToInt&lt;double, uint64_t&gt;(value));">`NormalizeFloatToInt`</SwmToken> does the actual normalization and bitwise conversion.

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

<SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="153:2:2" line-data="Int NormalizeFloatToInt(Float value) {">`NormalizeFloatToInt`</SwmToken> normalizes negative zero and <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="162:7:7" line-data="    // Turn arbtirary NaN representations to a consistent NaN repr.">`NaN`</SwmToken>, then uses memcpy to copy the float's bits into an int for hashing. This keeps the hash consistent for all float values.

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

After float and int handling in <SwmToken path="include/perfetto/ext/base/murmur_hash.h" pos="179:2:2" line-data="auto MurmurHashBuiltinValue(const T&amp; value) {">`MurmurHashBuiltinValue`</SwmToken>, we hash strings by their bytes and pointers by their address. If the type isn't supported, we return an invalid marker.

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

## Global Track Interning Finalization

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1{"What is the event type? (slice_type)"}
  click node1 openCode "src/trace_processor/importers/common/track_compressor.cc:190:205"
  node1 -->|"Begin"| node2["Register event start"]
  click node2 openCode "src/trace_processor/importers/common/track_compressor.cc:192:193"
  node1 -->|"End"| node3["Register event end"]
  click node3 openCode "src/trace_processor/importers/common/track_compressor.cc:195:196"
  node1 -->|"Instant"| node4["Register start and end, ensure they match"]
  click node4 openCode "src/trace_processor/importers/common/track_compressor.cc:198:204"
  node1 -->|"Other"| node6["Report fatal error"]
  click node6 openCode "src/trace_processor/importers/common/track_compressor.cc:206:207"
  node2 --> node5["Return result"]
  click node5 openCode "src/trace_processor/importers/common/track_compressor.cc:193:193"
  node3 --> node5
  node4 --> node5

classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1{"What is the event type? (<SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="142:3:3" line-data="                                                AsyncSliceType slice_type) {">`slice_type`</SwmToken>)"}
%%   click node1 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:190:205"
%%   node1 -->|"Begin"| node2["Register event start"]
%%   click node2 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:192:193"
%%   node1 -->|"End"| node3["Register event end"]
%%   click node3 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:195:196"
%%   node1 -->|"Instant"| node4["Register start and end, ensure they match"]
%%   click node4 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:198:204"
%%   node1 -->|"Other"| node6["Report fatal error"]
%%   click node6 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:206:207"
%%   node2 --> node5["Return result"]
%%   click node5 openCode "<SwmPath>[src/…/common/track_compressor.cc](src/trace_processor/importers/common/track_compressor.cc)</SwmPath>:193:193"
%%   node3 --> node5
%%   node4 --> node5
%% 
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/trace_processor/importers/common/track_compressor.cc" line="190">

---

After hashing with <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="189:3:3" line-data="      base::MurmurHashValue(trace_id), raw_name);">`MurmurHashValue`</SwmToken>, <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="137:4:4" line-data="TrackId TrackCompressor::InternLegacyAsyncTrack(StringId raw_name,">`InternLegacyAsyncTrack`</SwmToken> uses the hash to intern the global track, calling <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="192:3:3" line-data="      return InternBegin(kBlueprint, tracks::Dimensions(source_scope, *it),">`InternBegin`</SwmToken>, <SwmToken path="src/trace_processor/importers/common/track_compressor.cc" pos="195:3:3" line-data="      return InternEnd(kBlueprint, tracks::Dimensions(source_scope, *it),">`InternEnd`</SwmToken>, or both for instant slices. This wraps up the track interning for global async tracks.

```c++
  switch (slice_type) {
    case AsyncSliceType::kBegin:
      return InternBegin(kBlueprint, tracks::Dimensions(source_scope, *it),
                         trace_id, tracks::DynamicName(raw_name), args_fn);
    case AsyncSliceType::kEnd:
      return InternEnd(kBlueprint, tracks::Dimensions(source_scope, *it),
                       trace_id, tracks::DynamicName(raw_name), args_fn);
    case AsyncSliceType::kInstant:
      TrackId begin =
          InternBegin(kBlueprint, tracks::Dimensions(source_scope, *it),
                      trace_id, tracks::DynamicName(raw_name), args_fn);
      TrackId end = InternEnd(kBlueprint, tracks::Dimensions(source_scope, *it),
                              trace_id, tracks::DynamicName(raw_name), args_fn);
      PERFETTO_DCHECK(begin == end);
      return begin;
  }
  PERFETTO_FATAL("For GCC");
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
