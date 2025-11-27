---
title: Organizing Unfold-Related Track Sections
---
This document describes how unfold-related track sections are organized in the workspace to improve clarity and analysis. The flow receives a workspace with unfold-related track data and outputs an organized workspace where tracks are grouped into core, miscellaneous, display switching, and animation-related sections.

# Organizing Unfold-Related Track Sections in the Callback

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="35">

---

In <SwmToken path="ui/src/plugins/com.android.LargeScreensPerf/index.ts" pos="35:1:1" line-data="      callback: async () =&gt; {">`callback`</SwmToken>, we first pin core tracks, then group the miscellaneous tracks to keep the workspace tidy before adding more specific sections.

```typescript
      callback: async () => {
        this.pinCoreTracks(ctx);
        this.addUnfoldMiscSection(ctx);
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="67">

---

<SwmToken path="ui/src/plugins/com.android.LargeScreensPerf/index.ts" pos="67:3:3" line-data="  private addUnfoldMiscSection(ctx: Trace) {">`addUnfoldMiscSection`</SwmToken> grabs less important tracks by name, clones them, and groups them together to keep the workspace organized.

```typescript
  private addUnfoldMiscSection(ctx: Trace) {
    // section for tracks that don't fit neatly in other sections and are not so important to be pinned
    const group = new TrackNode({name: 'Unfold misc'});
    ctx.currentWorkspace.addChildFirst(group);
    ctx.currentWorkspace.flatTracks
      .filter(
        (t) =>
          t.name.startsWith('waitForAllWindowsDrawn') ||
          t.name == 'Waiting for KeyguardDrawnCallback#onDrawn',
      )
      .map((t) => t.clone())
      .forEach((track) => group.addChildLast(track));
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="38">

---

After handling misc tracks, the callback continues by grouping display switching tracks to keep things structured.

```typescript
        await this.addUnfoldDisplaySwitchingSection(ctx);
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="96">

---

<SwmToken path="ui/src/plugins/com.android.LargeScreensPerf/index.ts" pos="96:5:5" line-data="  private async addUnfoldDisplaySwitchingSection(ctx: Trace) {">`addUnfoldDisplaySwitchingSection`</SwmToken> filters out display-related tracks by name, optionally adds a special photonic modulator track if found, sorts them, clones them, and groups them under 'Unfold display switching' for clear organization in the workspace.

```typescript
  private async addUnfoldDisplaySwitchingSection(ctx: Trace) {
    const group = new TrackNode({name: 'Unfold display switching'});
    ctx.currentWorkspace.addChildFirst(group);

    const displayTracks = ctx.currentWorkspace.flatTracks.filter(
      (t) =>
        t.name.includes('android.display') ||
        t.name.includes('Screen on blocked'),
    );
    const photonicModulatorTrack = await this.findPhotonicModulatorTrack(ctx);
    if (photonicModulatorTrack != undefined) {
      displayTracks.push(photonicModulatorTrack);
    }
    displayTracks
      // sorting so that "android.display" tracks are next to each other
      .sort((t1, t2) => t1.name.localeCompare(t2.name))
      .map((t) => t.clone())
      .forEach((t) => group.addChildFirst(t));
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="39">

---

Back in the callback, after grouping display switching tracks, we finish by calling <SwmToken path="ui/src/plugins/com.android.LargeScreensPerf/index.ts" pos="39:3:3" line-data="        this.addUnfoldAnimationSection(ctx);">`addUnfoldAnimationSection`</SwmToken>. This keeps the animation-related tracks grouped separately and wraps up the section organization.

```typescript
        this.addUnfoldAnimationSection(ctx);
      },
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/com.android.LargeScreensPerf/index.ts" line="81">

---

<SwmToken path="ui/src/plugins/com.android.LargeScreensPerf/index.ts" pos="81:3:3" line-data="  private addUnfoldAnimationSection(ctx: Trace) {">`addUnfoldAnimationSection`</SwmToken> filters tracks by known animation-related name patterns, clones them, and groups them under 'Unfold animation' to keep all animation events together in the workspace.

```typescript
  private addUnfoldAnimationSection(ctx: Trace) {
    const group = new TrackNode({name: 'Unfold animation'});
    ctx.currentWorkspace.addChildFirst(group);
    ctx.currentWorkspace.flatTracks
      .filter(
        (t) =>
          t.name == 'FoldUpdate' ||
          t.name.includes('UnfoldTransition') ||
          t.name.includes('UnfoldLightRevealOverlayAnimation') ||
          t.name.endsWith('UNFOLD_ANIM>'),
      )
      .map((t) => t.clone())
      .forEach((track) => group.addChildLast(track));
  }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
