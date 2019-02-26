<template>
  <div id="interactive" class="viewport scanner">
    <video></video>
    <canvas class="drawingBuffer"></canvas>
  </div>
</template>

<script>
  import Quagga from 'quagga';

  export default {
    name: "scanner",
    props: {
      onDetected: {
        type: Function,
      },
      onProcessed: {
        type: Function,
      },
      readerType: {
        type: Array,
        default: [{
          format: 'code_128_reader',
          config: {}
        }]
      },
      readerSize: {
        width : {
          type: Number,
          default: 640,
        },
        height: {
          type: Number,
          default: 480,
        }
      }
    },
    data () {
      return {
        quaggaState: {
          inputStream: {
            type: 'LiveStream',
            constraints: {
              width: {min: this.readerSize.width},
              height: {min: this.readerSize.height},
              facingMode: 'environment',
              aspectRatio: {min: 1, max: 2}
            }
          },
          locator: {
            patchSize: 'medium',
            halfSample: true
          },
          numOfWorkers: 4,
          frequency: 10,
          decoder: {
            readers: this.readerType
          },
          locate: true
        },
      }
    },
    mounted () {
      this.init();
    },
    methods: {
      init () {
        Quagga.init(this.quaggaState, function (err) {
          if (err) {
            return console.log(err);
          }

          Quagga.start();
        });

        Quagga.onDetected(this.onDetected ? this.onDetected : this._onDetected);
        Quagga.onProcessed(this.onProcessed ? this.onProcessed : this._onProcessed);
      },
      reInit () {
        Quagga.stop();

        this.init();
      },
      getImage () {
        const canvas = Quagga.canvas.dom.image;

        return canvas.toDataURL();
      },
      _onProcessed (result) {
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
      _onDetected (result) {
        console.log('detected: ', result);
      },
    }
  }
</script>

<style lang="scss">
  .viewport {
    position: relative;

    canvas, video {
      position: absolute;
      left: 0;
      top: 0;
    }
  }
</style>
