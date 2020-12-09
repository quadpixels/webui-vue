<template>
  <b-container fluid="xl">
    <page-title />
    <page-section>
      <p>{{ dbusVisMessage }}</p>
      <p style="color:red">{{ dbusVisErrorMessage }}</p>
      <b-button @click="startCapture">Start Capture</b-button>
      <b-button @click="captureStatus">Capture Status</b-button>
      <b-button @click="stopCapture">Stop Capture</b-button>
      <b-button @click="downloadCapture">Download Capture</b-button>
    </page-section>

    <hr />

    <div id="div_group_by">
      <span id="span_group_by_dbus">
        Group DBus Requests by:
        <input id="dbus_column1" type="checkbox" value="Type" /> Type
        <input id="dbus_column2" type="checkbox" value="Serial" /> Serial
        <input id="dbus_column3" type="checkbox" value="Sender" /> Sender
        <input id="dbus_column4" type="checkbox" value="Destination" />
        Destination
        <input id="dbus_column5" type="checkbox" value="Path" checked /> Path
        <input id="dbus_column6" type="checkbox" value="Interface" checked />
        Interface
        <input id="dbus_column7" type="checkbox" value="Member" /> Member
      </span>
      <span id="span_group_by_ipmi">
        Group IPMI Requests by:
        <input id="c1" type="checkbox" value="NetFN" checked />NetFN
        <input id="c2" type="checkbox" value="CMD" checked />CMD
      </span>
    </div>

    <div id="div_canvas">
      <canvas id="my_canvas_dbus" width="1400" height="300"></canvas>
    </div>

    <div id="div_navi_and_replay">
      <div>
        <span>
          Navigation Control:
          <button id="btn_zoom_in">Zoom In</button>
          <button id="btn_zoom_out">Zoom Out</button>
          <button id="btn_pan_left">&lt;&lt;</button>
          <button id="btn_pan_right">&gt;&gt;</button>
          <button id="btn_zoom_reset">Reset</button>
        </span>
      </div>
      <div>
        Keyboard: [Left]/[right] arrow keys to pan; [Up]/[down] arrow keys to
        zoom in/out; Hold [shift] to move faster<br />Mouse: [Left click]:
        highlight an interval; [wheel up/down]: zoom in/out; click overflow
        triangles to warp to out-of-viewport requests
      </div>
      <div id="highlight_hint">
        Click highlighted region to zoom into the region
      </div>
      <br />
    </div>
    <!-- navi and replay -->
    <br />
  </b-container>
</template>

<script>
import PageTitle from '../../../components/Global/PageTitle';
import PageSection from '../../../components/Global/PageSection';
import DBusVis from './DBusVis';

export default {
  name: 'MyHelloWorld',
  components: { PageTitle, PageSection },
  computed: {
    dbusVisMessage() {
      return this.$store.getters['dbusVis/getMessage'];
    },
    dbusVisErrorMessage() {
      return this.$store.getters['dbusVis/getErrorMessage'];
    }
  },
  created() {
    console.log('created()');
  },
  mounted() {
    DBusVis.Hey();
  },
  beforeRouteLeave(to, from, next) {
    console.log(
      'beforeRouteLeave, to=' + to + ', from=' + from + ', next=' + next
    );
    next();
  },
  methods: {
    startCapture() {
      console.log('startCapture');
      console.log(this.$store.dispatch('dbusVis/startDbusCapture'));
    },
    captureStatus() {
      console.log('captureStatus');
      console.log(this.$store.dispatch('dbusVis/getDbusCaptureStatus'));
    },
    stopCapture() {
      console.log('stopCapture');
      console.log(this.$store.dispatch('dbusVis/stopDbusCapture'));
    },
    downloadCapture() {
      console.log('downloadCapture');
      console.log(this.$store.dispatch('dbusVis/downloadCapture'));
    }
  }
};
</script>
