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

### Coordinate space and scaling

The coordinates and dimensions of the `DOMRect` objects passed to
`window.chromeos.isolatedWebApp.setShape()` are specified in Device Independent
Pixels (DIPs) (the local coordinate space of the operating system window). At
100% page zoom, 1 DIP equals 1 CSS pixel.

When the browser applies the shape to the operating system window:

- **Fractional Coordinates**: While `DOMRect` properties (`x`, `y`, `width`,
  `height`) accept floating-point numbers, the underlying operating system
  window shape primitives operate on integer pixel boundaries. When applying
  the shape, fractional DIP coordinate values are truncated to integers and
  clamped to the range of a 32-bit signed integer.
- **Device Pixel Ratio (DPR)**: On high-density displays (where
  `window.devicePixelRatio` is not `1.0`), the window compositor and event
  targeter automatically scale the DIP coordinates by the display's device
  scale factor to determine physical device pixel boundaries. Developers do
  not need to perform manual coordinate scaling for high-DPI screens.
- **Page Zoom**: Because coordinates are evaluated in window DIPs rather than
  CSS pixels, applying page zoom to the web document does not change the
  physical window shape. Developers only need to adjust coordinates on page
  zoom changes if they intend for the window shape to scale alongside zoomed
  DOM content.

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

## Scope considerations

Details of the scope for this API has been discussed on this
[blink-api-owners-discuss thread](https://groups.google.com/a/chromium.org/g/blink-api-owners-discuss/c/q_bRRc_RxBo/m/iAaMUIbOAAAJ).
Below is a summary of the main points relevant for the Window Shape API.

### Why not standardize this API?

With the deprecation of Chrome Apps, Virtual Desktop Infrastructure (VDI)
partners transitioning to IWAs require more capabilities from the platform to
implement essential features (e.g., custom floating overlays and widgets). One
such capability available in Chrome Apps is the
`chrome.app.window.setShape([rects])` API.

The proposed `window.chromeos.isolatedWebApp.setShape` mirrors the existing
Chrome App API to IWAs. Doing this ensures timely migration to IWAs as it
minimizes window management logic changes for partner applications.

While `setShape` de-risks the Chrome App deprecation effort, the API does not
have a good use-case fit as is required for standardization. The VDI use cases
we are aware of are better served in the long run by more specific web APIs,
rather than general window shape masking.

For example, the use case of a floating overlay is better served with an API
specific for such floating windows.

### Why restrict to an allowlist?

It is undesirable that the API gets widespread adoption if it has no path to
standardization. The allowlist enforces the API is only used for the VDI
scenarios it is intended for.

Currently the allowlist contains the few VDI partners that use the Chrome App
API and want to continue to use it in their IWAs. Other IWA developers in the
VDI space expressed no interest in joining the allowlist. Although more apps can
be added to the allowlist in the future, we don't expect this will happen
frequently and the allowlist will remain small.

### Why restrict to ChromeOS?

This API aims to facilitate the migration from Chrome Apps to IWAs in existing
production deployments. Given that Chrome Apps are deprecated in all other
platforms, this API only needs to be available in ChromeOS to achieve its goal.

The API is therefore restricted to ChromeOS because the intended adoption is
limited to ChromeOS.

Additional reasons are:

- The API is implemented as a
  [Blink extension](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/renderer/extensions/README.md;drc=5a634aaffd86f5585918bd2050ab6bceabaac6cf).
  As such, it is restricted to the embedder that implements it. In this case,
  Chrome on ChromeOS.
- The original `chrome.app.window.setShape([rects])` API for Chrome Apps is
  only actively supported in ChromeOS.
- The `setShape` API requires the window to be in `unframed` display mode,
  which is only supported in ChromeOS. See details in the
  [Unframed Windows Explainer](https://github.com/WICG/manifest-incubations/blob/gh-pages/unframed-explainer.md).

### Why use the `chromeos.isolatedWebApp.<api>` namespace?

The API is exposed under `window.chromeos.isolatedWebApp.setShape` rather than a
standard web namespace (such as `window` or `navigator`) for the following
reasons:

- The `window.chromeos` namespace signals that this is **not a standard web
  API**, that it is not in a path to general web standardization, and that the
  capability is embedder-specific.
- The `isolatedWebApp` sub-namespace clarifies that the capability is
  restricted to IWAs. Standard web pages, browser tabs, PWAs, and other Web
  content in ChromeOS cannot access it.
- This approach follows the existing precedent from Android webview. That
  platform has Blink extensions defined in the
  [`window.android.webview.<api>` namespace](https://crsrc.org/c/third_party/blink/renderer/extensions/webview/android.idl).

## Alternatives considered

### Other input shape formats

Besides a list of `DOMRect[]`, we could use other formats to define the window
shape, such as CSS `clip-path`, polygons, or SVG path strings.

These alternatives are rejected because signature and behavior parity with
`chrome.app.window.setShape([rects])` is essential for partner migration.
Requiring partners to convert existing logic creates unnecessary migration
friction.

Another advantage of `DOMRect` is it has a straightforward mapping to the
underlying compositor primitives, which simplifies the implementation.
