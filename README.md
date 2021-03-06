# vue-cc-quaggajs

[quaggajs](https://serratus.github.io/quaggaJS/) 's wrapper for Vue.js

# Installtion

## npm

``` bash
npm i vue-cc-quaggajs
```

```vue
<template>
  <quagga-scanner ref="scanner"></quagga-scanner>
</template>

<script>
import { QuaggaScanner } from 'vue-cc-quaggajs';

export default {
  //...
  components: {
    QuaggaScanner,
  },
  // ...
}
</script>
```
# Usage

### `init` : function()

```js
function () {
  Quagga.init(this.quaggaState, function (err) {
    if (err) {
      return console.log(err);
    }

    Quagga.start();
  });

  Quagga.onDetected(this.onDetected ? this.onDetected : this._onDetected);
  Quagga.onProcessed(this.onProcessed ? this.onProcessed : this._onProcessed);
}
```

### `reInit` : function()

```js
function () {
  Quagga.stop();

  this.init();
}
```

### `getImage` : function()

```js
function () {
  const canvas = Quagga.canvas.dom.image;

  return canvas.toDataURL();
}
```

### `onDetected` : function(result)

Reference: [Quagga.onDetected(callback)](https://github.com/serratus/quaggaJS#quaggaondetectedcallback)

default function:

```js
function (result) {
  console.log('detected: ', result);
}
```

### `onProcessed` : function(result)

Reference: [Quagga.onProcessed(callback)](https://github.com/serratus/quaggaJS#quaggaonprocessedcallback)

default function:

```js
function (result) {
  let drawingCtx = Quagga.canvas.ctx.overlay,
    drawingCanvas = Quagga.canvas.dom.overlay;

  if (result) {
    if (result.boxes) {
      drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
      result.boxes.filter(function (box) {
        return box !== result.box;
      }).forEach(function (box) {
        Quagga.ImageDebug.drawPath(box, {x: 0, y: 1}, drawingCtx, {color: "green", lineWidth: 2});
      });
    }
    if (result.box) {
      Quagga.ImageDebug.drawPath(result.box, {x: 0, y: 1}, drawingCtx, {color: "#00F", lineWidth: 2});
    }

    if (result.codeResult && result.codeResult.code) {
      Quagga.ImageDebug.drawPath(result.line, {x: 'x', y: 'y'}, drawingCtx, {color: 'red', lineWidth: 3});
    }
  }
},
```

### readerType: Array

Set reading barcode type.

Reference: [https://github.com/serratus/quaggaJS#decoder](https://github.com/serratus/quaggaJS#decoder)

default: `[{
          format: 'code_128_reader',
          config: {}
        }]`

### readerSize: Object {width: Number, height: Number}

Set reader size. it affects <video> size.

default:

```js
{
  width: 640,
  height: 480,
}
```

# Basic Example

```
<template>
  <div>
    <quagga-scanner :onDetected="logIt" :readerSize="readerSize" :readerType="'ean_reader'"></quagga-scanner>
  </div>
</template>

<script>
import { QuaggaScanner } from 'vue-cc-quaggajs'
export default {
  name: 'VueBarcodeTest',
  data () {
    return {
      readerSize: {
        width: 640,
        height: 480
      }
    }
  },
  components: {
    QuaggaScanner
  },
  methods: {
    logIt (data) {
      console.log('detected', data)
    }

  }
}
</script>
```


