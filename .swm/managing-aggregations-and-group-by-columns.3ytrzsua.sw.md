---
title: Managing Aggregations and Group-By Columns
---
This document describes how users manage aggregations and <SwmToken path="ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" pos="225:5:7" line-data="        &#39;.pf-group-by-selector&#39;,">`group-by`</SwmToken> columns through an interactive UI. The flow ensures there is always at least one editable aggregation, allows users to add or remove aggregations, and updates the interface in response to user actions.

# Rendering and Managing Aggregation Editors

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1["Are there any aggregations?"]
  click node1 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:621:623"
  node2["Add an editable aggregation"]
  click node2 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:622:623"
  node3["Select group-by columns"]
  click node3 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:824:859"

  node1 -- No --> node2
  node2 --> node3
  node1 -- Yes --> node3

  subgraph loop1["For each aggregation"]
    node4["Is aggregation being edited?"]
    click node4 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:803:807"
    node5["Show aggregation editor (choose operation type, column, name)"]
    click node5 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:628:756"
    node6["Show aggregation summary"]
    click node6 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:758:795"
    node4 -- Yes --> node5
    node4 -- No --> node6
  end
  node3 --> loop1

  node7["Is last aggregation valid?"]
  click node7 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:799:800"
  node8["Show 'Add more aggregations' button"]
  click node8 openCode "ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts:809:820"
  loop1 --> node7
  node7 -- Yes --> node8
  node8 --> loop1
  node7 -- No --> end["Done"]
classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;

%% Swimm:
%% %%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
%% flowchart TD
%%   node1["Are there any aggregations?"]
%%   click node1 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:621:623"
%%   node2["Add an editable aggregation"]
%%   click node2 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:622:623"
%%   node3["Select <SwmToken path="ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" pos="225:5:7" line-data="        &#39;.pf-group-by-selector&#39;,">`group-by`</SwmToken> columns"]
%%   click node3 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:824:859"
%% 
%%   node1 -- No --> node2
%%   node2 --> node3
%%   node1 -- Yes --> node3
%% 
%%   subgraph loop1["For each aggregation"]
%%     node4["Is aggregation being edited?"]
%%     click node4 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:803:807"
%%     node5["Show aggregation editor (choose operation type, column, name)"]
%%     click node5 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:628:756"
%%     node6["Show aggregation summary"]
%%     click node6 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:758:795"
%%     node4 -- Yes --> node5
%%     node4 -- No --> node6
%%   end
%%   node3 --> loop1
%% 
%%   node7["Is last aggregation valid?"]
%%   click node7 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:799:800"
%%   node8["Show 'Add more aggregations' button"]
%%   click node8 openCode "<SwmPath>[ui/…/nodes/aggregation_node.ts](ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts)</SwmPath>:809:820"
%%   loop1 --> node7
%%   node7 -- Yes --> node8
%%   node8 --> loop1
%%   node7 -- No --> end["Done"]
%% classDef HeadingStyle fill:#777777,stroke:#333,stroke-width:2px;
```

<SwmSnippet path="/ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" line="619">

---

In <SwmToken path="ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" pos="619:1:1" line-data="  view({attrs}: m.CVnode&lt;AggregationOperationComponentAttrs&gt;) {">`view`</SwmToken>, we kick things off by making sure there's always at least one aggregation editor visible—if the list is empty, we add a new editable aggregation. We set up helpers for column validation and aggregation validation, and define the editor and viewer components for each aggregation. This sets up the core UI logic for editing and displaying aggregations.

```typescript
  view({attrs}: m.CVnode<AggregationOperationComponentAttrs>) {
    // Initialize with an aggregation editor if we don't have any aggregations yet
    if (attrs.aggregations.length === 0) {
      attrs.aggregations.push({isEditing: true});
    }

    // Use the utility function to determine if a column is valid for the given operation
    const isColumnValidForOp = isColumnValidForAggregation;

    const aggregationEditor = (agg: Aggregation, index: number): m.Child => {
      const columnOptions = attrs.groupByColumns.map((col) => {
        const isValid = isColumnValidForOp(col, agg.aggregationOp);
        return m(
          'option',
          {
            value: col.name,
            selected: agg.column?.name === col.name,
            disabled: !isValid,
          },
          col.name,
        );
      });

      // Validation function that checks if the aggregation is complete and valid
      const isAggregationValid = (): boolean => {
        return validateAggregation(agg);
      };

      return m(
        Form,
        {
          submitLabel: 'Apply',
          submitIcon: Icons.Check,
          cancelLabel: 'Cancel',
          required: true,
          validation: isAggregationValid,
          onSubmit: (e: Event) => {
            e.preventDefault();
            if (!agg.newColumnName) {
              agg.newColumnName = placeholderNewColumnName(agg);
            }
            agg.isEditing = false;
            attrs.onchange?.();
          },
          onCancel: () => {
            // If this is a new aggregation that hasn't been confirmed yet, remove it
            if (!agg.isValid) {
              attrs.aggregations.splice(index, 1);
            } else {
              // Otherwise just stop editing
              agg.isEditing = false;
            }
            m.redraw();
          },
        },
        m(
          '.pf-exp-aggregation-editor',
          m(
            Select,
            {
              required: true,
              onchange: (e: Event) => {
                agg.aggregationOp = (e.target as HTMLSelectElement).value;
                // Clear percentile when changing operation
                if (agg.aggregationOp !== 'PERCENTILE') {
                  agg.percentile = undefined;
                }
                // Clear column when switching to COUNT_ALL
                if (agg.aggregationOp === 'COUNT_ALL') {
                  agg.column = undefined;
                }
                m.redraw();
              },
            },
            m(
              'option',
              {disabled: true, selected: !agg.aggregationOp, value: ''},
              'Select operation',
            ),
            AGGREGATION_OPS.map((op) =>
              m(
                'option',
                {
                  value: op,
                  selected: op === agg.aggregationOp,
                },
                op,
              ),
            ),
          ),
          // Percentile value input (only for PERCENTILE operation, shown before column)
          agg.aggregationOp === 'PERCENTILE' &&
            m(TextInput, {
              placeholder: 'percentile (0-100)',
              type: 'number',
              min: 0,
              max: 100,
              required: true,
              oninput: (e: InputEvent) => {
                const value = parseFloat((e.target as HTMLInputElement).value);
                agg.percentile = isNaN(value) ? undefined : value;
                m.redraw();
              },
              value: agg.percentile?.toString() ?? '',
            }),
          // Column selector (not shown for COUNT_ALL)
          agg.aggregationOp &&
            agg.aggregationOp !== 'COUNT_ALL' &&
            m(
              Select,
              {
                required: true,
                onchange: (e: Event) => {
                  const target = e.target as HTMLSelectElement;
                  agg.column = attrs.groupByColumns.find(
                    (c) => c.name === target.value,
                  );
                  m.redraw();
                },
              },
              m(
                'option',
                {disabled: true, selected: !agg.column, value: ''},
                'Select column',
              ),
              columnOptions,
            ),
          'AS',
          m(TextInput, {
            placeholder: placeholderNewColumnName(agg),
            oninput: (e: Event) => {
              agg.newColumnName = (e.target as HTMLInputElement).value.trim();
            },
            value: agg.newColumnName,
          }),
        ),
      );
    };

    const aggregationViewer = (agg: Aggregation, index: number): m.Child => {
      let aggDisplay = '';
      if (agg.aggregationOp === 'COUNT_ALL') {
        aggDisplay = 'COUNT(*)';
      } else if (
        agg.aggregationOp === 'PERCENTILE' &&
        agg.percentile !== undefined
      ) {
        aggDisplay = `PERCENTILE(${agg.column?.name}, ${agg.percentile})`;
      } else {
        aggDisplay = `${agg.aggregationOp}(${agg.column?.name})`;
      }

      return m(
        '.pf-exp-aggregation-viewer',
        m(
          'span',
          {
            onclick: () => {
              attrs.aggregations.forEach((a, i) => {
                a.isEditing = i === index;
              });
              m.redraw();
            },
          },
          `${aggDisplay} AS ${agg.newColumnName}`,
        ),
        m(Button, {
          icon: Icons.Close,
          onclick: (e: Event) => {
            e.stopPropagation();
            attrs.aggregations.splice(index, 1);
            attrs.onchange?.();
            m.redraw();
          },
        }),
      );
    };

    const aggregationsList = (): m.Children => {
      const lastAgg = attrs.aggregations[attrs.aggregations.length - 1];
      const showAddButton = lastAgg.isValid;

      return [
        ...attrs.aggregations.map((agg, index) => {
          if (agg.isEditing) {
            return aggregationEditor(agg, index);
          } else {
            return aggregationViewer(agg, index);
          }
        }),
        showAddButton &&
          m(Button, {
            label: 'Add more aggregations',
            onclick: () => {
              if (!lastAgg.newColumnName) {
                lastAgg.newColumnName = placeholderNewColumnName(lastAgg);
              }
              lastAgg.isEditing = false;
              attrs.aggregations.push({isEditing: true});
              attrs.onchange?.();
            },
          }),
      ];
    };

    const selectGroupByColumns = (): m.Child => {
      const groupByOptions: MultiSelectOption[] = attrs.groupByColumns.map(
        (col) => ({
          id: col.name,
          name: col.name,
          checked: col.checked,
        }),
      );

      const selectedGroupBy = attrs.groupByColumns.filter((c) => c.checked);
      const label =
        selectedGroupBy.length > 0
          ? selectedGroupBy.map((c) => c.name).join(', ')
          : 'None';

      return m(
        '.pf-exp-multi-select-container',
        m('label', 'GROUP BY columns'),
        m(PopupMultiSelect, {
          label,
          options: groupByOptions,
          showNumSelected: false,
          onChange: (diffs: MultiSelectDiff[]) => {
            for (const diff of diffs) {
              const column = attrs.groupByColumns.find(
                (c) => c.name === diff.id,
              );
              if (column) {
                column.checked = diff.checked;
              }
            }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" line="855">

---

We finish up by rendering the aggregation editors and viewers using <SwmToken path="ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" pos="867:15:15" line-data="          m(&#39;.pf-exp-aggregations-list&#39;, aggregationsList()),">`aggregationsList`</SwmToken>, so the UI always matches the current aggregation state.

```typescript
            attrs.onchange?.();
          },
        }),
      );
    };

    return m(
      '.pf-exp-query-operations',
      m(Card, {}, [
        m(
          '.pf-exp-operations-container',
          selectGroupByColumns(),
          m('.pf-exp-aggregations-list', aggregationsList()),
        ),
      ]),
    );
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" line="797">

---

<SwmToken path="ui/src/plugins/dev.perfetto.ExplorePage/query_builder/nodes/aggregation_node.ts" pos="797:3:3" line-data="    const aggregationsList = (): m.Children =&gt; {">`aggregationsList`</SwmToken> builds the UI for all aggregations, switching between editor and viewer modes based on state. It also handles adding new aggregations with side effects—mutating the list, updating flags, and triggering callbacks—so the UI and logic stay in sync with user actions.

```typescript
    const aggregationsList = (): m.Children => {
      const lastAgg = attrs.aggregations[attrs.aggregations.length - 1];
      const showAddButton = lastAgg.isValid;

      return [
        ...attrs.aggregations.map((agg, index) => {
          if (agg.isEditing) {
            return aggregationEditor(agg, index);
          } else {
            return aggregationViewer(agg, index);
          }
        }),
        showAddButton &&
          m(Button, {
            label: 'Add more aggregations',
            onclick: () => {
              if (!lastAgg.newColumnName) {
                lastAgg.newColumnName = placeholderNewColumnName(lastAgg);
              }
              lastAgg.isEditing = false;
              attrs.aggregations.push({isEditing: true});
              attrs.onchange?.();
            },
          }),
      ];
    };
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
