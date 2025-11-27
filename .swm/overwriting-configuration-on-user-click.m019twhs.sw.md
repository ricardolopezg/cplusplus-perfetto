---
title: Overwriting configuration on user click
---
This document describes how users can update an existing configuration by clicking on a configuration option. The system prompts for confirmation, saves the new configuration if confirmed, and refreshes the UI to reflect the changes.

# Handling config overwrite on user click

<SwmSnippet path="/ui/src/plugins/dev.perfetto.RecordTraceV2/pages/target_selection_page.ts" line="263">

---

Onclick kicks off the flow by handling the user's click event. It stops the event from bubbling up, asks the user if they want to overwrite the current config with the latest settings, and if confirmed, saves the session state under that config name and triggers a full UI redraw. This ties together user intent, state persistence, and UI update.

```typescript
                onclick: (e: Event) => {
                  e.stopPropagation();
                  if (
                    confirm(
                      `Overwrite config "${config.name}" with current settings?`,
                    )
                  ) {
                    recMgr.saveConfig(config.name, recMgr.serializeSession());
                    recMgr.app.raf.scheduleFullRedraw();
                  }
                },
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
