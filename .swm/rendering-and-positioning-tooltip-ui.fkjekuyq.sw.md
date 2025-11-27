---
title: Rendering and Positioning Tooltip UI
---
This document describes how the tooltip UI is rendered and positioned. When a user interacts with a trigger element, the tooltip content is displayed and positioned to ensure it appears above other interface elements, integrating with overlays and containers for a seamless experience.

# Rendering and Positioning Tooltip UI

<SwmSnippet path="/ui/src/widgets/tooltip.ts" line="71">

---

Tooltip.view renders the trigger and, if open, the tooltip content by delegating to Tooltip.renderToolip. This keeps the tooltip logic isolated and only active when needed.

```typescript
  view({attrs, children}: m.CVnode<TooltipAttrs>): m.Children {
    const {trigger} = attrs;

    return [
      this.renderTrigger(trigger),
      this.isOpen && this.renderToolip(attrs, children),
    ];
  }
```

---

</SwmSnippet>

<SwmSnippet path="/ui/src/widgets/tooltip.ts" line="97">

---

Tooltip.renderToolip handles rendering the tooltip content using a Portal, so it sits outside the regular DOM tree and avoids layout issues. It sets up lifecycle hooks to manage where the tooltip goes, initializes Popper for positioning, and cleans up when the tooltip is removed. It also checks for nested overlays and uses repo-specific classes and refs for styling and integration.

```typescript
  private renderToolip(attrs: TooltipAttrs, children: m.Children): m.Children {
    const {
      className,
      showArrow = true,
      onTooltipMount = () => {},
      onTooltipUnMount = () => {},
      fitContent,
    } = attrs;

    const portalAttrs: PortalAttrs = {
      className: 'pf-tooltip-portal',
      onBeforeContentMount: (dom: Element): MountOptions => {
        // Check to see if dom is a descendant of a popup or modal
        // If so, get the popup's "container" and put it in there instead
        // This handles the case where popups are placed inside the other popups
        // we nest outselves in their containers instead of document body which
        // means we become part of their hitbox for mouse events.
        const closestPopup = dom.closest(`[ref=${Tooltip.TOOLTIP_REF}]`);
        if (closestPopup) {
          return {container: closestPopup};
        }
        const closestModal = dom.closest('.pf-overlay-container');
        if (closestModal) {
          return {container: closestModal};
        }
        const closestContainer = dom.closest('.pf-overlay-container');
        if (closestContainer) {
          return {container: closestContainer};
        }
        return {container: undefined};
      },
      onContentMount: (dom: HTMLElement) => {
        const popupElement = toHTMLElement(
          assertExists(findRef(dom, Tooltip.TOOLTIP_REF)),
        );
        this.tooltipElement = popupElement;
        this.createOrUpdatePopper(attrs);
        onTooltipMount(popupElement);
      },
      onContentUpdate: () => {
        this.popper?.update();
      },
      onContentUnmount: () => {
        if (this.tooltipElement) {
          onTooltipUnMount(this.tooltipElement);
        }
        this.popper?.destroy();
        this.popper = undefined;
        this.tooltipElement = undefined;
      },
    };

    return m(
      Portal,
      portalAttrs,
      m(
        '.pf-popup', // Re-use popup styles
        {
          class: classNames(className, fitContent && 'pf-popup--fit-content'),
          ref: Tooltip.TOOLTIP_REF,
        },
        showArrow && m('.pf-popup-arrow[data-popper-arrow]'),
        m('.pf-popup-content', children),
      ),
    );
  }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY3BsdXNwbHVzLXBlcmZldHRvJTNBJTNBcmljYXJkb2xvcGV6Zw==" repo-name="cplusplus-perfetto"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
