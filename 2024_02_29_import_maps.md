# Control ES modules with Import maps

## EcmaScript bare modules

The algorithm for resolving bare modules is not specified:

```js
// These imports will NOT work in browsers
import Layer from "ol";
const {Layer} = await import("ol");
```

## Import maps

A mechanism for apps to control the module resolution algorithm, in the browser.

For example, to select some specific version of OL, we might do:

```html
<script type="importmap">
{
  "imports": {
    "ol": "https://cdn.jsdelivr.net/npm/ol@v9.0.0/dist/ol.js"
  }
}
</script>
```

It is also possible to create the mapping programmatically.

For example, to implement a demo page allowing testing a range of versions:

```html
<script>
  const olVersion = new URLSearchParams(document.location.search).get('olVersion') || '9.0.0';
  const cesiumVersion = new URLSearchParams(document.location.search).get('olVersion') || '1.115';
  // sanitizeIt(olVersion, cesiumVersion)
  const importMap = {
    imports: {
      ol: `https://cdn.jsdelivr.net/npm/ol@v${olVersion}/dist/ol.js`
      cesium: `https://cesium.com/downloads/cesiumjs/releases/${cesiumVersion}/Build/Cesium/Cesium.js`
    },
  };
  const im = document.createElement('script');
  im.type = 'importmap';
  im.textContent = JSON.stringify(importMap);
  document.currentScript.after(im);
</script>
```
