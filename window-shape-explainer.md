# Window Shape API Explainer

## Introduction

This explainer proposes the `window.chromeos.isolatedWebApp.setShape` API. This
API allows allowlisted Isolated Web Apps (IWAs) to customize the shape of their
window, enabling non-rectangular window layouts.

To use this API, the window must be in the `unframed` display mode (which in
turn requires the `window-management` permission).

## Goal

Allow developers to implement experiences with non-rectangular and
non-contiguous shapes for IWA windows on ChromeOS.

## Non-goals

- Support this API on platforms other than ChromeOS.
- Allow non-allowlisted IWAs or standard web pages to use this API.
- Support complex custom shapes, e.g. based on SVG paths. The custom shape is
  restricted to a set of rectangles.

## Proposed API

The API is exposed under the `window.chromeos` namespace:

```typescript
interface ChromeOSIsolatedWebApp {
  setShape(rectangles: DOMRect[]): Promise<void>;
}
```

Example usage:

```javascript
// Set a window shape consisting of two overlapping rectangles.
await window.chromeos.isolatedWebApp.setShape([
  new DOMRect(0, 0, 200, 200),
  new DOMRect(180, 190, 100, 300),
]);

// Clear the custom shape (reset to default rectangular window).
await window.chromeos.isolatedWebApp.setShape([]);
```

### Requirements

- The IWA origin must be allowlisted to use the `setShape` API.
- The window must be in the `unframed` display mode.
- The origin must be granted the `window-management` permission.
- Rectangles must meet a minimum size requirement of 10x10 pixels to avoid
  invisible windows. At least one rectangle in the custom shape must meet this
  requirement.

## Security & privacy considerations

### Invisible/tiny windows

Developers could use `setShape` to make the window tiny or invisible, so it is
effectively hidden. To mitigate this we enforce a minimum size for the custom
shape.

### Clickjacking

Shaped windows can be used to overlay elements confusingly. For example, a donut
shaped window can partially occlude another app and cause users to mistakenly
click the wrong UI elements. The API is restricted to allowlisted IWAs and
requires the `window-management` permission, reducing the risk of malicious
exploitation.

### Allowlist

Restricting the API to allowlisted origins ensures that only trusted, vetted
applications can use this capability on ChromeOS.
