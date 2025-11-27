---
title: Emitting a High-Level Trace Event
---
This document describes how the system emits a high-level trace event for profiling and analysis. The process receives event details and optional metadata, prepares the tracing state, records the event, and finalizes the emission. This enables developers and system integrators to capture detailed activity for later analysis.

# Entry Point: Emitting a High-Level Trace Event

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="549">

---

<SwmToken path="src/shared_lib/track_event/hl.cc" pos="549:2:2" line-data="void PerfettoTeHlEmitImpl(struct PerfettoTeCategoryImpl* cat,">`PerfettoTeHlEmitImpl`</SwmToken> is just the entry point for emitting a high-level trace event. It forwards all its arguments directly to <SwmToken path="src/shared_lib/track_event/hl.cc" pos="553:5:5" line-data="  perfetto::shlib::TeHlEmit(cat, type, name, extra_data);">`TeHlEmit`</SwmToken>, which does the actual work. We call <SwmToken path="src/shared_lib/track_event/hl.cc" pos="553:5:5" line-data="  perfetto::shlib::TeHlEmit(cat, type, name, extra_data);">`TeHlEmit`</SwmToken> next because that's where the trace event emission logic lives; this function just bridges the API boundary.

```c++
void PerfettoTeHlEmitImpl(struct PerfettoTeCategoryImpl* cat,
                          int32_t type,
                          const char* name,
                          struct PerfettoTeHlExtra* const* extra_data) {
  perfetto::shlib::TeHlEmit(cat, type, name, extra_data);
}
```

---

</SwmSnippet>

# Preparing Trace Event Emission

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="510">

---

In <SwmToken path="src/shared_lib/track_event/hl.cc" pos="510:2:2" line-data="void TeHlEmit(struct PerfettoTeCategoryImpl* cat,">`TeHlEmit`</SwmToken>, we check if there are any active trace instances, prep the data source and thread-local state, and run a prologue. Then, for each active instance, we call <SwmToken path="src/shared_lib/track_event/hl.cc" pos="539:5:5" line-data="    perfetto::shlib::InstanceOp(ds, &amp;ii, tls_state, cat,">`InstanceOp`</SwmToken> to actually emit the trace event for that instance. Without calling <SwmToken path="src/shared_lib/track_event/hl.cc" pos="539:5:5" line-data="    perfetto::shlib::InstanceOp(ds, &amp;ii, tls_state, cat,">`InstanceOp`</SwmToken>, nothing would get written to the trace.

```c++
void TeHlEmit(struct PerfettoTeCategoryImpl* cat,
              int32_t type,
              const char* name,
              struct PerfettoTeHlExtra* const* extra_data) {
  uint32_t cached_instances =
      perfetto::shlib::TracePointTraits::GetActiveInstances({cat})->load(
          std::memory_order_relaxed);
  if (!cached_instances) {
    return;
  }

  perfetto::internal::DataSourceType* ds =
      perfetto::shlib::TrackEvent::GetType();

  perfetto::internal::DataSourceThreadLocalState*& tls_state =
      *perfetto::shlib::TrackEvent::GetTlsState();

  if (!ds->TracePrologue<perfetto::shlib::TrackEventDataSourceTraits,
                         perfetto::shlib::TracePointTraits>(
          &tls_state, &cached_instances, {cat})) {
    return;
  }

  for (perfetto::internal::DataSourceType::InstancesIterator ii =
           ds->BeginIteration<perfetto::shlib::TracePointTraits>(
               cached_instances, tls_state, {cat});
       ii.instance;
       ds->NextIteration</*Traits=*/perfetto::shlib::TracePointTraits>(
           &ii, tls_state, {cat})) {
    perfetto::shlib::InstanceOp(ds, &ii, tls_state, cat,
                                perfetto::shlib::EventType(type), name,
                                extra_data);
  }
```

---

</SwmSnippet>

## Parsing Extra Data and Track Selection

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1{"Is there a valid instance?"}
  click node1 openCode "src/shared_lib/track_event/hl.cc:346:348"
  node1 --|"No"| nodeEnd["Stop processing"]
  click nodeEnd openCode "src/shared_lib/track_event/hl.cc:347:348"
  node1 --|"Yes"| node2["Extract extra metadata"]
  click node2 openCode "src/shared_lib/track_event/hl.cc:364:402"
  subgraph loop1["For each extra metadata item"]
    node2 --> node3{"Type of metadata?"}
    click node3 openCode "src/shared_lib/track_event/hl.cc:366:401"
    node3 -->|"Track"| node4["Set track"]
    node3 -->|"Timestamp"| node5["Set custom timestamp"]
    node3 -->|"Dynamic category"| node6["Set dynamic category"]
    node3 -->|"Flush"| node7["Set flush flag"]
  end
  node2 --> node8{"Is custom timestamp present?"}
  click node8 openCode "src/shared_lib/track_event/hl.cc:405:410"
  node8 --|"Yes"| node9["Use custom timestamp"]
  click node9 openCode "src/shared_lib/track_event/hl.cc:406:408"
  node8 --|"No"| node10["Use default timestamp"]
  click node10 openCode "src/shared_lib/track_event/hl.cc:409:410"
  node9 --> node11{"Is dynamic category present?"}
  node10 --> node11
  click node11 openCode "src/shared_lib/track_event/hl.cc:412:417"
  node11 --|"Yes"| node12["Enable dynamic category"]
  click node12 openCode "src/shared_lib/track_event/hl.cc:413:416"
  node12 --> node13{"Is there a valid instance after enabling?"}
  click node13 openCode "src/shared_lib/track_event/hl.cc:414:416"
  node13 --|"No"| nodeEnd
  node13 --|"Yes"| node14{"Track type?"}
  node11 --|"No"| node14
  click node14 openCode "src/shared_lib/track_event/hl.cc:429:485"
  node14 --|"Registered"| node15["Emit registered track"]
  node14 --|"Named"| node16["Emit named track"]
  node14 --|"Proto"| node17["Emit proto track"]
  node14 --|"Nested"| node18["Emit nested tracks"]
  subgraph loop2["For each nested track"]
    node18 --> node19{"Nested track type?"}
    click node19 openCode "src/shared_lib/track_event/hl.cc:455:482"
    node19 -->|"Named"| node20["Emit named track"]
    node19 -->|"Process"| node21["Emit process track"]
    node19 -->|"Thread"| node22["Emit thread track"]
    node19 -->|"Proto"| node23["Emit proto track"]
    node19 -->|"Registered"| node24["Emit registered track"]
  end
  node15 --> node25["Write trace event"]
  node16 --> node25
  node17 --> node25
  node18 --> node25
  click node25 openCode "src/shared_lib/track_event/hl.cc:487:494"
  node25 --> node26{"Should flush?"}
  click node26 openCode "src/shared_lib/track_event/hl.cc:505:507"
  node26 --|"Yes"| node27["Immediately make event visible"]
  click node27 openCode "src/shared_lib/track_event/hl.cc:506:507"
  node26 --|"No"| node28["Done"]
  click node28 openCode "src/shared_lib/track_event/hl.cc:508:508"
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1{"Is there a valid instance?"}
%%   click node1 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:346:348"
%%   node1 --|"No"| nodeEnd["Stop processing"]
%%   click nodeEnd openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:347:348"
%%   node1 --|"Yes"| node2["Extract extra metadata"]
%%   click node2 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:364:402"
%%   subgraph loop1["For each extra metadata item"]
%%     node2 --> node3{"Type of metadata?"}
%%     click node3 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:366:401"
%%     node3 -->|"Track"| node4["Set track"]
%%     node3 -->|"Timestamp"| node5["Set custom timestamp"]
%%     node3 -->|"Dynamic category"| node6["Set dynamic category"]
%%     node3 -->|"Flush"| node7["Set flush flag"]
%%   end
%%   node2 --> node8{"Is custom timestamp present?"}
%%   click node8 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:405:410"
%%   node8 --|"Yes"| node9["Use custom timestamp"]
%%   click node9 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:406:408"
%%   node8 --|"No"| node10["Use default timestamp"]
%%   click node10 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:409:410"
%%   node9 --> node11{"Is dynamic category present?"}
%%   node10 --> node11
%%   click node11 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:412:417"
%%   node11 --|"Yes"| node12["Enable dynamic category"]
%%   click node12 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:413:416"
%%   node12 --> node13{"Is there a valid instance after enabling?"}
%%   click node13 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:414:416"
%%   node13 --|"No"| nodeEnd
%%   node13 --|"Yes"| node14{"Track type?"}
%%   node11 --|"No"| node14
%%   click node14 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:429:485"
%%   node14 --|"Registered"| node15["Emit registered track"]
%%   node14 --|"Named"| node16["Emit named track"]
%%   node14 --|"Proto"| node17["Emit proto track"]
%%   node14 --|"Nested"| node18["Emit nested tracks"]
%%   subgraph loop2["For each nested track"]
%%     node18 --> node19{"Nested track type?"}
%%     click node19 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:455:482"
%%     node19 -->|"Named"| node20["Emit named track"]
%%     node19 -->|"Process"| node21["Emit process track"]
%%     node19 -->|"Thread"| node22["Emit thread track"]
%%     node19 -->|"Proto"| node23["Emit proto track"]
%%     node19 -->|"Registered"| node24["Emit registered track"]
%%   end
%%   node15 --> node25["Write trace event"]
%%   node16 --> node25
%%   node17 --> node25
%%   node18 --> node25
%%   click node25 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:487:494"
%%   node25 --> node26{"Should flush?"}
%%   click node26 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:505:507"
%%   node26 --|"Yes"| node27["Immediately make event visible"]
%%   click node27 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:506:507"
%%   node26 --|"No"| node28["Done"]
%%   click node28 openCode "<SwmPath>[src/…/track_event/hl.cc](src/shared_lib/track_event/hl.cc)</SwmPath>:508:508"
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="339">

---

In <SwmToken path="src/shared_lib/track_event/hl.cc" pos="339:2:2" line-data="void InstanceOp(internal::DataSourceType* ds,">`InstanceOp`</SwmToken>, we parse the <SwmToken path="src/shared_lib/track_event/hl.cc" pos="345:9:9" line-data="                struct PerfettoTeHlExtra* const* extra_data) {">`extra_data`</SwmToken> array to extract all the optional parameters for the trace event—track info, timestamps, counters, flags, etc. The function uses a variant to handle different track types and optionals for counters and UUIDs, so it can flexibly support whatever the caller provides. This setup is needed before we can actually emit the event data.

```c++
void InstanceOp(internal::DataSourceType* ds,
                internal::DataSourceType::InstancesIterator* ii,
                internal::DataSourceThreadLocalState* tls_state,
                struct PerfettoTeCategoryImpl* cat,
                protos::pbzero::TrackEvent::Type type,
                const char* name,
                struct PerfettoTeHlExtra* const* extra_data) {
  if (!ii->instance) {
    return;
  }

  std::variant<std::monostate, const PerfettoTeRegisteredTrackImpl*,
               const PerfettoTeHlExtraNamedTrack*,
               const PerfettoTeHlExtraProtoTrack*,
               const PerfettoTeHlExtraNestedTracks*>
      track;
  std::optional<uint64_t> track_uuid;

  const struct PerfettoTeHlExtraTimestamp* custom_timestamp = nullptr;
  const struct PerfettoTeCategoryDescriptor* dynamic_cat = nullptr;
  std::optional<int64_t> int_counter;
  std::optional<double> double_counter;
  bool use_interning = true;
  bool flush = false;

  for (const auto* it = extra_data; *it != nullptr; it++) {
    const struct PerfettoTeHlExtra& extra = **it;
    if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_REGISTERED_TRACK) {
      const auto& cast =
          reinterpret_cast<const struct PerfettoTeHlExtraRegisteredTrack&>(
              extra);
      track = cast.track;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_NAMED_TRACK) {
      track =
          &reinterpret_cast<const struct PerfettoTeHlExtraNamedTrack&>(extra);
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_PROTO_TRACK) {
      track =
          &reinterpret_cast<const struct PerfettoTeHlExtraProtoTrack&>(extra);
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_NESTED_TRACKS) {
      auto* nested =
          &reinterpret_cast<const struct PerfettoTeHlExtraNestedTracks&>(extra);
      track = nested;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_TIMESTAMP) {
      custom_timestamp =
          &reinterpret_cast<const struct PerfettoTeHlExtraTimestamp&>(extra);
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_DYNAMIC_CATEGORY) {
      dynamic_cat =
          reinterpret_cast<const struct PerfettoTeHlExtraDynamicCategory&>(
              extra)
              .desc;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_COUNTER_INT64) {
      int_counter =
          reinterpret_cast<const struct PerfettoTeHlExtraCounterInt64&>(extra)
              .value;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_COUNTER_DOUBLE) {
      double_counter =
          reinterpret_cast<const struct PerfettoTeHlExtraCounterInt64&>(extra)
              .value;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_NO_INTERN) {
      use_interning = false;
    } else if (extra.type == PERFETTO_TE_HL_EXTRA_TYPE_FLUSH) {
      flush = true;
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="404">

---

After parsing the parameters, we pick the right timestamp (custom or current), handle dynamic categories by advancing the iterator, and then get the trace writer and TLS state. Next, we emit the track—registered, named, proto, or nested—using the right Emit\* function. For nested tracks, we walk through each one and emit them in order, updating the parent UUID as we go. This sets up everything for writing the actual event data.

```c++
  TraceTimestamp ts;
  if (custom_timestamp) {
    ts.clock_id = custom_timestamp->timestamp.clock_id;
    ts.value = custom_timestamp->timestamp.value;
  } else {
    ts = TrackEventInternal::GetTraceTime();
  }

  if (PERFETTO_UNLIKELY(dynamic_cat)) {
    AdvanceToFirstEnabledDynamicCategory(ii, tls_state, cat, *dynamic_cat);
    if (!ii->instance) {
      return;
    }
  }

  perfetto::TraceWriterBase* trace_writer = ii->instance->trace_writer.get();

  const auto& track_event_tls = *static_cast<TrackEventTlsState*>(
      ii->instance->data_source_custom_tls.get());

  auto* incr_state = static_cast<TrackEventIncrementalState*>(
      ds->GetIncrementalState(ii->instance, ii->i));
  ResetIncrementalStateIfRequired(trace_writer, incr_state, track_event_tls,
                                  ts);

  if (std::holds_alternative<const PerfettoTeRegisteredTrackImpl*>(track)) {
    auto* registered_track =
        std::get<const PerfettoTeRegisteredTrackImpl*>(track);
    track_uuid =
        EmitRegisteredTrack(registered_track, incr_state, trace_writer);
  } else if (std::holds_alternative<const PerfettoTeHlExtraNamedTrack*>(
                 track)) {
    auto* named_track = std::get<const PerfettoTeHlExtraNamedTrack*>(track);
    track_uuid = EmitNamedTrack(named_track->parent_uuid, named_track->name,
                                named_track->id, incr_state, trace_writer);
  } else if (std::holds_alternative<const PerfettoTeHlExtraProtoTrack*>(
                 track)) {
    auto* proto_track = std::get<const PerfettoTeHlExtraProtoTrack*>(track);
    track_uuid = EmitProtoTrack(proto_track->uuid, proto_track->fields,
                                incr_state, trace_writer);
  } else if (std::holds_alternative<const PerfettoTeHlExtraNestedTracks*>(
                 track)) {
    auto* nested = std::get<const PerfettoTeHlExtraNestedTracks*>(track);

    uint64_t uuid = 0;

    for (PerfettoTeHlNestedTrack* const* tp = nested->tracks; *tp != nullptr;
         tp++) {
      auto track_type =
          static_cast<enum PerfettoTeHlNestedTrackType>((*tp)->type);

      switch (track_type) {
        case PERFETTO_TE_HL_NESTED_TRACK_TYPE_NAMED: {
          auto* named_track =
              reinterpret_cast<PerfettoTeHlNestedTrackNamed*>(*tp);
          uuid = EmitNamedTrack(uuid, named_track->name, named_track->id,
                                incr_state, trace_writer);
        } break;
        case PERFETTO_TE_HL_NESTED_TRACK_TYPE_PROCESS: {
          uuid = perfetto_te_process_track_uuid;
        } break;
        case PERFETTO_TE_HL_NESTED_TRACK_TYPE_THREAD: {
          uuid = perfetto_te_process_track_uuid ^
                 static_cast<uint64_t>(perfetto::base::GetThreadId());
        } break;
        case PERFETTO_TE_HL_NESTED_TRACK_TYPE_PROTO: {
          auto* proto_track =
              reinterpret_cast<PerfettoTeHlNestedTrackProto*>(*tp);
          uuid = EmitProtoTrackWithParentUuid(proto_track->id ^ uuid, uuid,
                                              proto_track->fields, incr_state,
                                              trace_writer);
        } break;
        case PERFETTO_TE_HL_NESTED_TRACK_TYPE_REGISTERED: {
          auto* registered_track =
              reinterpret_cast<PerfettoTeHlNestedTrackRegistered*>(*tp);
          uuid = EmitRegisteredTrack(registered_track->track, incr_state,
                                     trace_writer);
        } break;
      }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="484">

---

After emitting the track and writing the event data, we finalize the trace packet. If there's interned data, we append it and reset the buffer. If the flush flag was set, we flush the trace writer so the data is sent out immediately. Nothing is returned; the work is done via side effects on the trace writer.

```c++
    track_uuid = uuid;
  }

  {
    auto packet = NewTracePacketInternal(
        trace_writer, incr_state, track_event_tls, ts,
        protos::pbzero::TracePacket::SEQ_NEEDS_INCREMENTAL_STATE);
    auto* track_event = packet->set_track_event();
    WriteTrackEvent(incr_state, track_event, cat, type, name, extra_data,
                    track_uuid, dynamic_cat, use_interning);
    track_event->Finalize();

    if (!incr_state->serialized_interned_data.empty()) {
      auto ranges = incr_state->serialized_interned_data.GetRanges();
      packet->AppendScatteredBytes(
          protos::pbzero::TracePacket::kInternedDataFieldNumber, ranges.data(),
          ranges.size());
      incr_state->serialized_interned_data.Reset();
    }
  }

  if (PERFETTO_UNLIKELY(flush)) {
    trace_writer->Flush();
  }
}
```

---

</SwmSnippet>

## Finalizing Trace Event Emission

<SwmSnippet path="/src/shared_lib/track_event/hl.cc" line="543">

---

We just got back from <SwmToken path="src/shared_lib/track_event/hl.cc" pos="339:2:2" line-data="void InstanceOp(internal::DataSourceType* ds,">`InstanceOp`</SwmToken>, so all the trace event data has been emitted. Now, <SwmToken path="src/shared_lib/track_event/hl.cc" pos="510:2:2" line-data="void TeHlEmit(struct PerfettoTeCategoryImpl* cat,">`TeHlEmit`</SwmToken> calls <SwmToken path="src/shared_lib/track_event/hl.cc" pos="543:3:3" line-data="  ds-&gt;TraceEpilogue(tls_state);">`TraceEpilogue`</SwmToken> to wrap up and clean up any state related to the trace emission. This keeps the tracing system consistent after each event.

```c++
  ds->TraceEpilogue(tls_state);
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
