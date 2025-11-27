---
title: Loading and Registering Bluetooth Trace Tracks
---
When a trace file is loaded, the system checks for Bluetooth or BLE features. If present, it registers tracks to visualize Bluetooth activities and metrics, enabling users to analyze Bluetooth behavior from the trace data.

# Loading and Registering Bluetooth Trace Tracks

<SwmSnippet path="/ui/src/plugins/com.android.Bluetooth/index.ts" line="474">

---

In <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="474:3:3" line-data="  async onTraceLoad(ctx: Trace): Promise&lt;void&gt; {">`onTraceLoad`</SwmToken>, we start by checking if the trace has Bluetooth or BLE features. If not, we bail early. If yes, we add a bunch of slice tracks for different Bluetooth activities, then query for activity data. After that, we call <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="542:5:5" line-data="    await support.addCounterTrack(">`addCounterTrack`</SwmToken> from the <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="25:12:12" line-data="import SupportPlugin from &#39;../com.android.AndroidLongBatterySupport&#39;;">`AndroidLongBatterySupport`</SwmToken> plugin to register counter tracks for metrics like active counts and duty cycles. Calling into that plugin lets us reuse its logic for creating and registering counter tracks, keeping things modular and consistent.

```typescript
  async onTraceLoad(ctx: Trace): Promise<void> {
    const support = this.support(ctx);
    const features = await support.features(ctx.engine);
    if (
      !Array.from(features.values()).some(
        (f) => f.startsWith('atom.bluetooth_') || f.startsWith('atom.ble_'),
      )
    ) {
      return;
    }

    const groupName = 'Bluetooth';
    await support.addSliceTrack(
      ctx,
      'BLE Scans (opportunistic)',
      bleScanDataset('opportunistic'),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'BLE Scans (filtered)',
      bleScanDataset('filtered'),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'BLE Scans (unfiltered)',
      bleScanDataset('not filtered'),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'BLE Scan Results',
      BLE_RESULTS_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'Connections (ACL)',
      BT_CONNS_ACL_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'Connections (SCO)',
      BT_CONNS_SCO_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'Link-level Events',
      BT_LINK_LEVEL_EVENTS_DATASET,
      groupName,
    );

    await support.addSliceTrack(
      ctx,
      'A2DP Audio',
      BT_A2DP_AUDIO_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'Bytes Transferred (L2CAP/RFCOMM)',
      BT_BYTES_DATASET,
      groupName,
    );
    await ctx.engine.query(BT_ACTIVITY);
    await support.addCounterTrack(
      ctx,
      'ACL Classic Active Count',
      'select ts, dur, acl_active_count as value from bt_activity',
      groupName,
    );
    await support.addCounterTrack(
      ctx,
      'ACL Classic Sniff Count',
      'select ts, dur, acl_sniff_count as value from bt_activity',
      groupName,
    );
    await support.addCounterTrack(
      ctx,
      'ACL BLE Count',
      'select ts, dur, acl_ble_count as value from bt_activity',
      groupName,
    );
    await support.addCounterTrack(
      ctx,
      'Advertising Instance Count',
      'select ts, dur, advertising_count as value from bt_activity',
      groupName,
    );
    await support.addCounterTrack(
      ctx,
      'LE Scan Duty Cycle Maximum',
      'select ts, dur, le_scan_duty_cycle as value from bt_activity',
      groupName,
      {unit: '%'},
    );
    await support.addSliceTrack(
      ctx,
      'Inquiry Active',
      new SourceDataset({
        src: `SELECT
                ts,
                dur,
                'Active' as name
              FROM bt_activity
              WHERE inquiry_active`,
        schema: {
          ts: LONG,
          dur: LONG_NULL,
          name: STR,
        },
      }),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'SCO Active',
      new SourceDataset({
        src: `SELECT
                ts,
                dur,
                'Active' as name
              FROM bt_activity
              WHERE sco_active`,
        schema: {
          ts: LONG,
          dur: LONG_NULL,
          name: STR,
        },
      }),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'A2DP Active',
      new SourceDataset({
        src: `SELECT
                ts,
                dur,
                'Active' as name
              FROM bt_activity
              WHERE a2dp_active`,
        schema: {
          ts: LONG,
          dur: LONG_NULL,
          name: STR,
        },
      }),
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'LE Audio Active',
      new SourceDataset({
        src: `SELECT
                ts,
                dur,
                'Active' as name
              FROM bt_activity
              WHERE le_audio_active`,
        schema: {
          ts: LONG,
          dur: LONG_NULL,
          name: STR,
        },
      }),
      groupName,
    );
    await support.addCounterTrack(
      ctx,
      'Controller Idle Time',
      'select ts, dur, controller_idle_pct as value from bt_activity',
      groupName,
      {yRangeSharingKey: 'bt_controller_time', unit: '%'},
    );
    await support.addCounterTrack(
      ctx,
      'Controller TX Time',
      'select ts, dur, controller_tx_pct as value from bt_activity',
      groupName,
      {yRangeSharingKey: 'bt_controller_time', unit: '%'},
    );
    await support.addCounterTrack(
      ctx,
      'Controller RX Time',
      'select ts, dur, controller_rx_pct as value from bt_activity',
      groupName,
      {yRangeSharingKey: 'bt_controller_time', unit: '%'},
    );
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.AndroidLongBatterySupport/index.ts" line="87">

---

<SwmToken path="ui/src/plugins/com.android.AndroidLongBatterySupport/index.ts" pos="87:3:3" line-data="  async addCounterTrack(">`addCounterTrack`</SwmToken> builds a URI using the track name, then creates a query counter track with fixed columns \['ts', 'value'\] and the provided SQL. It registers the track and adds a <SwmToken path="ui/src/plugins/com.android.AndroidLongBatterySupport/index.ts" pos="109:9:9" line-data="    const trackNode = new TrackNode({uri, name});">`TrackNode`</SwmToken> to the context, grouping and collapsing it. The URI prefix and columns are repo conventions, but there's no validation for unsafe names or invalid SQL.

```typescript
  async addCounterTrack(
    ctx: Trace,
    name: string,
    query: string,
    groupName: string,
    options?: Partial<CounterOptions>,
    groupCollapsed = true,
  ) {
    const uri = `/long_battery_tracing_${name}`;
    const track = await createQueryCounterTrack({
      trace: ctx,
      uri,
      data: {
        sqlSource: query,
        columns: ['ts', 'value'],
      },
      options,
    });
    ctx.tracks.registerTrack({
      uri,
      renderer: track,
    });
    const trackNode = new TrackNode({uri, name});
    this.addTrack(ctx, trackNode, groupName, groupCollapsed);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.Bluetooth/index.ts" line="666">

---

Back in <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="474:3:3" line-data="  async onTraceLoad(ctx: Trace): Promise&lt;void&gt; {">`onTraceLoad`</SwmToken>, after returning from <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="542:5:5" line-data="    await support.addCounterTrack(">`addCounterTrack`</SwmToken> in the <SwmToken path="ui/src/plugins/com.android.Bluetooth/index.ts" pos="25:12:12" line-data="import SupportPlugin from &#39;../com.android.AndroidLongBatterySupport&#39;;">`AndroidLongBatterySupport`</SwmToken> plugin, we keep adding more slice tracks for Bluetooth quality, RSSI, HAL crashes, and code path counters. The counter tracks are already registered, and these final slice tracks wrap up the trace loading.

```typescript
    await support.addSliceTrack(
      ctx,
      'Quality reports',
      BT_QUALITY_REPORTS_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'RSSI Reports',
      BT_RSSI_REPORTS_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'HAL Crashes',
      BT_HAL_CRASHES_DATASET,
      groupName,
    );
    await support.addSliceTrack(
      ctx,
      'Code Path Counter',
      BT_CODE_PATH_COUNTER_DATASET,
      groupName,
    );
  }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
