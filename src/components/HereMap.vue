<template>
  <div>
    <div ref="map" id="map"></div>
  </div>
</template>

<script>
import config from "../config/config";
import svg from "../assets/resource";

export default {
  name: "HereMap",
  data() {
    return {
      platform: {},
      map: {},
      ui: null,
      bubbleOpen: true,
      defaultLayers: null,
      header: {
        headers: {
          "Content-Type": "application/json",
        },
      },
      is_first: true,
      ref_id: null,
      ref_item_id: null,
      driver_position: null,
      driver_marker: null,
      icon: {},
      group: null,
      groupMarker: {
        data: [],
      },
      receiverMarkup: svg.receiver.replace("${SEQ}", "2"),
      deliverMarkup: svg.deliver.replace("${SEQ}", "1"),
      driverMarkup: svg.driver,
    };
  },
  created() {
    this.init();
  },
  mounted() {
    this.initMap();
    this.getDriverLocation();
    this.getDeliverLocation();
    this.addEvent();
    // setInterval(this.getDriverLocation, 5000);
  },

  methods: {
    init() {
      // eslint-disable-next-line
      this.platform = new H.service.Platform({
        apikey: config.apiKey,
      });
      this.defaultLayers = this.platform.createDefaultLayers();

      this.ref_id = this.$route.query.reference_id;
      this.ref_item_id = this.$route.query.reference_item_id;
      // eslint-disable-next-line
      this.icon.receiver = new H.map.Icon(this.receiverMarkup);
      // eslint-disable-next-line
      this.icon.deliver = new H.map.Icon(this.deliverMarkup);
      // eslint-disable-next-line
      this.icon.driver = new H.map.Icon(this.driverMarkup);
    },

    initMap() {
      // eslint-disable-next-line
      this.map = new H.Map(
        this.$refs.map,
        this.defaultLayers.vector.normal.map,
        {
          center: { lat: 13.809535, lng: 100.595998 },
          zoom: 10,
          padding: { top: 200, left: 200, bottom: 200, right: 200 },
          pixelRatio:
            window.devicePixelRatio && window.devicePixelRatio > 1 ? 2 : 1,
        }
      );

      //eslint-disable-next-line
      this.ui = H.ui.UI.createDefault(this.map, this.defaultLayers);

      var setViewBounds = this.setViewBounds;
      let ic_location = svg.location
      let ic_driver = svg.driver_location
      let goToDriver = this.goToDriver;
      var customUI = this.createButton(ic_location, setViewBounds);
      var customUI2 = this.createButton(ic_driver, goToDriver);

      // eslint-disable-next-line
      this.inherits(customUI, H.ui.Control);
      // eslint-disable-next-line
      this.inherits(customUI2, H.ui.Control);

      var buttonCustomUI = new customUI();
      this.ui.addControl("buttonCustomUI", buttonCustomUI);
      var buttonCustomUI2 = new customUI2();
      this.ui.addControl("buttonCustomUI2", buttonCustomUI2);
    },

    createButton(svg, func) {
      var customUI = function (opt_options) {
        // "use strict";
        var options = opt_options || {};
        // eslint-disable-next-line
        H.ui.Control.call(this);
        // create a button element
        // eslint-disable-next-line
        this.newButton = new H.ui.base.Button({
          label: svg,
          // onStateChange: this.onButtonClick,
          onStateChange: (evt) => {
            if (evt.currentTarget.getState() === "down") {
              func();
            }
          },
        });

        //add the buttons as this control's children
        this.addChild(this.newButton);

        this.setAlignment(options["alignment"] || "left-bottom");

        this.options_ = options;
      };

      return customUI;
    },

    addEvent() {
      window.addEventListener("resize", () => this.map.getViewPort().resize());
      // eslint-disable-next-line
      let mapEvent = new H.mapevents.MapEvents(this.map);
      // eslint-disable-next-line
      let behavior = new H.mapevents.Behavior(mapEvent);
      
      // let map = this.map
      // map.addEventListener("mapviewchange", function () {
      //   var zoom = map.getZoom();
      //   console.log(zoom);
      // });
    },

    async getDeliverLocation() {
      try {
        if (this.ref_id != null) {
          let request = JSON.stringify({
            reference_id: this.ref_id,
            reference_item_id: this.ref_item_id,
          });
          var { data } = await this.axios.post(
            config.apiUrl + "TransportJob.api/EDI/tracking_transportjob_detail",
            request,
            this.header
          );
          console.log("DeliverLocation =>", data);
          await data.forEach((element) => {
            let position = {
              lat: element.lat,
              lng: element.lng,
            };
            this.dropMarker(position, element.type);
          });
          this.setGroupMarker();
        }
      } catch (error) {
        console.log("getDeliverLocation =>", error.message);
      }
    },

    async getDriverLocation() {
      try {
        if (this.ref_id != null) {
          var { data } = await this.axios.post(
            config.apiUrl +
              "TransportJob.api/EDI/tracking_transportjob_current",
            JSON.stringify({ reference_id: this.ref_id }),
            this.header
          );
          console.log("driverLocation =>", data);
          let position = {
            lat: data.lat,
            lng: data.lng,
          };
          if (this.is_first) {
            this.dropMarker(position);
            this.is_first = false;
          } else {
            this.driver_marker.setGeometry(position); //not animetion
            // this.updateMarkerPositions(      //animation
            //   this.driver_position,
            //   position,
            //   4000,
            //   function (coord) {
            //     this.driver_marker.setGeometry(coord);
            //   }
            // );
          }
          this.driver_position = position;
          console.log("driverLocation =>", this.driver_position);
        }
      } catch (error) {
        console.log("getDriverLocation =>", error.message);
      }
    },

    setGroupMarker() {
      // eslint-disable-next-line
      this.group = new H.map.Group();
      this.group.addObjects(this.groupMarker.data);
      this.map.addObject(this.group);
      this.setViewBounds();
    },

    setViewBounds() {
      if (this.group != null) {
        let bounds = this.group.getBoundingBox();
        this.map.getViewModel().setLookAtData(
          {
            bounds: bounds,
            position: bounds.getCenter(),
            // zoom: bounds.getZoom(),
          },
          true
        );
      }
    },

    goToDriver() {
      this.driver_position != null &&
        this.map.setCenter(this.driver_position, true);
    },

    dropMarker(
      position,
      type = null
      // title = null,
      // adddress = null
    ) {
      var icon;
      if (type == "R") {
        icon = this.icon.receiver;
      } else if (type == "D") {
        icon = this.icon.deliver;
      } else {
        icon = this.icon.driver;
      }
      // eslint-disable-next-line
      let marker = new H.map.Marker(position, { icon: icon });
      // var xy = this.map.geoToScreen(position);
      if (type == "R" || type == "D") {
        marker.setData("<div><h3>AAAA</h3></div><div>asdsakdosakdo</div>");
        this.groupMarker.data.push(marker);
        let ui = this.ui;
        marker.addEventListener(
          "tap",
          function (evt) {
            let position = evt.target.getGeometry();
            // eslint-disable-next-line
            var bubble = new H.ui.InfoBubble(
              {
                lat: position.lat + 0.00005,
                lng: position.lng,
              },
              {
                // read custom data
                content: evt.target.getData(),
                // onStateChange: (evt) => {
                //   // eslint-disable-next-line
                //   if (evt.target.getState() === H.ui.InfoBubble.State.CLOSED) {
                //     console.log("isClosed");
                //     this.bubbleOpen = false;
                //   }
                // },
              }
            );
            ui.addBubble(bubble);
          },
          false
        );
      } else {
        this.groupMarker.data.push(marker);
        this.driver_marker = marker;
      }
      this.map.addObject(marker);
    },

    // updateMarkerPositions(                 //animation
    //   startCoord = { lat: 0, lng: 0 },
    //   endCoord = { lat: 1, lng: 1 },
    //   durationMs = 200,
    //   onStep = console.log,
    //   onComplete = function () {}
    // ) {
    //   var raf =
    //       window.requestAnimationFrame ||
    //       function (f) {
    //         window.setTimeout(f, 16);
    //       },
    //     stepCount = durationMs / 16,
    //     valueIncrementLat = (endCoord.lat - startCoord.lat) / stepCount,
    //     valueIncrementLng = (endCoord.lng - startCoord.lng) / stepCount,
    //     sinValueIncrement = Math.PI / stepCount,
    //     currentValueLat = startCoord.lat,
    //     currentValueLng = startCoord.lng,
    //     currentSinValue = 0;

    //   function step() {
    //     currentSinValue += sinValueIncrement;
    //     currentValueLat +=
    //       valueIncrementLat * Math.sin(currentSinValue) ** 2 * 2;
    //     currentValueLng +=
    //       valueIncrementLng * Math.sin(currentSinValue) ** 2 * 2;

    //     if (currentSinValue < Math.PI) {
    //       onStep({ lat: currentValueLat, lng: currentValueLng });
    //       raf(step);
    //     } else {
    //       onStep(endCoord);
    //       onComplete();
    //     }
    //   }

    //   raf(step);
    // },

    inherits: (childCtor, parentCtor) => {
      function tempCtor() {}
      tempCtor.prototype = parentCtor.prototype;
      childCtor.superClass_ = parentCtor.prototype;
      childCtor.prototype = new tempCtor();
      childCtor.prototype.constructor = childCtor;
      // eslint-disable-next-line
      childCtor.base = function (me, methodName, var_args) {
        var args = new Array(arguments.length - 2);
        for (var i = 2; i < arguments.length; i++) {
          args[i - 2] = arguments[i];
        }
        return parentCtor.prototype[methodName].apply(me, args);
      };
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#map {
  width: 100vw;
  height: 100vh;
}
</style>
