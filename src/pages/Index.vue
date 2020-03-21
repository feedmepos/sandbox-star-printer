<template>
  <q-page>
    <div>
      <h6>Star Printer</h6>
      <canvas ref="canvas" width="200" height="200" />
      <div v-if="hasStarPrint">
        <q-btn @click="getDeviceClicked">
          Get Devices
        </q-btn>
        <q-select
          v-model="emulation"
          :options="emulations"
          label="Emulation Mode"
        />
        <div v-for="(p, i) in printers" :key="i">
          {{ p.modelName }}:
          <q-btn @click="sendText(p)">Text</q-btn>
          <q-btn @click="sendImage(p, 300)">Image 300</q-btn>
          <q-btn @click="sendImage(p, 600)">Image 600</q-btn>
          <q-btn @click="sendImage(p, 900)">Image 900</q-btn>
          <q-btn @click="sendDrawer(p)">Drawer</q-btn>
          <q-btn @click="sendCutter(p)">Cutter</q-btn>
        </div>
        <q-separator spaced />
      </div>
      <div v-else>
        No Star Print Driver detected
      </div>
    </div>
  </q-page>
</template>

<script>
import Vue from "vue";

export default Vue.extend({
  data() {
    return {
      result: "",
      printers: [],
      emulation: "StarPRNT",
      emulations: [
        "StarPRNT",
        "StarLine",
        "StarGraphic",
        "StarDotImpact",
        "EscPosMobile",
        "EscPos"
      ]
    };
  },
  computed: {
    hasStarPrint() {
      return !!starprnt;
    }
  },
  mounted() {
    this.$q.loading.hide();
    console.log(starprnt);
    console.log(this.$refs.canvas);
    if (this.$refs.canvas) {
      const ctx = this.$refs.canvas.getContext("2d");
      this.drawStar(ctx, 100, 100, 5, 80, 40);
    }
  },
  methods: {
    drawStar(ctx, cx, cy, spikes, outerRadius, innerRadius) {
      var rot = (Math.PI / 2) * 3;
      var x = cx;
      var y = cy;
      var step = Math.PI / spikes;

      ctx.strokeSyle = "#000";
      ctx.beginPath();
      ctx.moveTo(cx, cy - outerRadius);
      for (let i = 0; i < spikes; i++) {
        x = cx + Math.cos(rot) * outerRadius;
        y = cy + Math.sin(rot) * outerRadius;
        ctx.lineTo(x, y);
        rot += step;

        x = cx + Math.cos(rot) * innerRadius;
        y = cy + Math.sin(rot) * innerRadius;
        ctx.lineTo(x, y);
        rot += step;
      }
      ctx.lineTo(cx, cy - outerRadius);
      ctx.closePath();
      ctx.lineWidth = 5;
      ctx.strokeStyle = "black";
      ctx.stroke();
    },
    getDeviceClicked() {
      this.$q.loading.show();
      if (this.hasStarPrint) {
        starprnt.portDiscovery(
          "All",
          result => {
            this.$q.loading.hide();
            console.log(result);
            this.printers = result;
          },
          error => {
            this.$q.loading.hide();
            console.log(error);
          }
        );
      }
    },
    sendText(printer) {
      this.$q.loading.show();
      if (this.hasStarPrint) {
        starprnt.printRawText(
          printer.portName,
          this.emulation,
          {
            text: "This is test print from feedme POS star printer tester"
          },
          result => {
            this.$q.loading.hide();
            console.log(result);
          },
          error => {
            this.$q.loading.hide();
            console.log(error);
          }
        );
      }
    },
    sendImage(printer, width) {
      this.$q.loading.show();
      if (this.hasStarPrint && this.$refs.canvas) {
        starprnt.printBase64Image(
          printer.portName,
          this.emulation,
          {
            base64Image: this.$refs.canvas.toDataURL().split("base64,")[1],
            width: width
          },
          result => {
            this.$q.loading.hide();
            console.log(result);
          },
          error => {
            this.$q.loading.hide();
            console.log(error);
          }
        );
      }
    },
    sendDrawer(printer) {
      this.$q.loading.show();
      if (this.hasStarPrint) {
        starprnt.openCashDrawer(
          printer.portName,
          this.emulation,
          result => {
            this.$q.loading.hide();
            console.log(result);
          },
          error => {
            this.$q.loading.hide();
            console.log(error);
          }
        );
      }
    },
    sendCutter(printer) {
      this.$q.loading.show();
      if (this.hasStarPrint) {
        starprnt.print(
          printer.portName,
          this.emulation,
          [{ appendCutPaper: "FullCut" }],
          result => {
            this.$q.loading.hide();
            console.log(result);
          },
          error => {
            this.$q.loading.hide();
            console.log(error);
          }
        );
      }
    }
  }
});
</script>
