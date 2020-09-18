<template>
  <div>
    <div ref="map" id="map"></div>
  </div>
</template>

<script>

export default {
  name: "HereMap",
  data() {
    return {
      platform: {},
      map: {},
      apiUrl: 'http://203.154.39.65/poc/',
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
      group: {
        data: [],
      },
      receiverMarkup:
        '<svg width="42" height="42" viewBox="0 0 42 42" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M33.6789 14.8706C33.6789 8.11125 28.1926 2.625 21.4332 2.625C14.6739 2.625 9.18762 8.11125 9.18762 14.8706C9.18762 17.115 9.79137 19.215 10.8414 21.0263L20.8032 39.0075C21.0789 39.5062 21.7876 39.5062 22.0632 39.0075L32.0251 21.0263C33.0751 19.215 33.6789 17.115 33.6789 14.8706Z" fill="#6FCF97"/><path d="M21.4998 23.2444C26.1245 23.2444 29.8736 19.4953 29.8736 14.8706C29.8736 10.2459 26.1245 6.49687 21.4998 6.49687C16.8752 6.49687 13.1261 10.2459 13.1261 14.8706C13.1261 19.4953 16.8752 23.2444 21.4998 23.2444Z" fill="white"/></svg>',
      deliverMarkup:
        '<svg width="42" height="42" viewBox="0 0 42 42" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M33.6789 14.8706C33.6789 8.11125 28.1926 2.625 21.4332 2.625C14.6739 2.625 9.18762 8.11125 9.18762 14.8706C9.18762 17.115 9.79137 19.215 10.8414 21.0263L20.8032 39.0075C21.0789 39.5062 21.7876 39.5062 22.0632 39.0075L32.0251 21.0263C33.0751 19.215 33.6789 17.115 33.6789 14.8706Z" fill="#EC870E"/><path d="M21.4998 23.2444C26.1245 23.2444 29.8736 19.4953 29.8736 14.8706C29.8736 10.2459 26.1245 6.49687 21.4998 6.49687C16.8752 6.49687 13.1261 10.2459 13.1261 14.8706C13.1261 19.4953 16.8752 23.2444 21.4998 23.2444Z" fill="white"/></svg>',
      driverMarkup:
        '<svg width="42" height="42" viewBox="0 0 42 42" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M33.6789 14.8706C33.6789 8.11125 28.1926 2.625 21.4332 2.625C14.6739 2.625 9.18762 8.11125 9.18762 14.8706C9.18762 17.115 9.79137 19.215 10.8414 21.0263L20.8032 39.0075C21.0789 39.5062 21.7876 39.5062 22.0632 39.0075L32.0251 21.0263C33.0751 19.215 33.6789 17.115 33.6789 14.8706Z" fill="#EC870E"/><path d="M21.4998 23.2444C26.1245 23.2444 29.8736 19.4953 29.8736 14.8706C29.8736 10.2459 26.1245 6.49687 21.4998 6.49687C16.8752 6.49687 13.1261 10.2459 13.1261 14.8706C13.1261 19.4953 16.8752 23.2444 21.4998 23.2444Z" fill="white"/></svg>',
    };
  },
  created() {
    this.init();
  },
  mounted() {
    this.initMap();
    this.getDeliverLocation();
    this.addEvent();
    setInterval(this.getDriverLocation, 5000);
  },

  methods: {
    init() {
      // eslint-disable-next-line
      this.platform = new H.service.Platform({
        apikey: 'y-xvunT-82HiKuCyINzNx4k-ClzOK7k5vriHRyvrECo',
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
          // center: { lat: 0, lng: 0 },
          zoom: 5,
          padding: { top: 50, left: 50, bottom: 50, right: 50 },
          pixelRatio:
            window.devicePixelRatio && window.devicePixelRatio > 1 ? 2 : 1,
        }
      );

      //eslint-disable-next-line
      var ui = H.ui.UI.createDefault(this.map, this.defaultLayers);

      // var customUI = function (opt_options) {
      //   "use strict";
      //   var options = opt_options || {};
      //   console.log(this);
      //   // eslint-disable-next-line
      //   H.ui.Control.call(this);
      //   this.onButtonClick = this.onButtonClick.bind();

      //   // create a button element
      //   // eslint-disable-next-line
      //   this.increaseBtn_ = new H.ui.base.Button({
      //     label:
      //       '<svg class="H_icon H_icon" viewBox="0 0 25 25">' +
      //       '<path d="M 18.5,11 H 14 V 6.5 c 0,-.8 -.7,-1.5 -1.5,-1.5 -.8,0 -1.5,.7 -1.5,1.5 V 11 H 6' +
      //       ".5 C 5.7,11 5,11.7 5,12.5 5,13.3 5.7,14 6.5,14 H 11 v 4.5 c 0,.8 .7,1.5 1.5,1.5 .8,0 1.5," +
      //       '-.7 1.5,-1.5 V 14 h 4.5 C 19.3,14 20,13.3 20,12.5 20,11.7 19.3,11 18.5,11 z" />' +
      //       "</svg>",
      //     onStateChange: this.onButtonClick,
      //   });

      //   //add the buttons as this control's children
      //   this.addChild(this.increaseBtn_);

      //   this.setAlignment(options["alignment"] || "top-right");

      //   this.options_ = options;
      // };
      // // eslint-disable-next-line
      // this.inherits(customUI, H.ui.Control);
      
      // customUI.prototype.onButtonClick = function (evt) {
      //   "use strict";
      //   if (evt.currentTarget.getState() === "down") {
           
      //     //this.setViewBounds();
      //     // console.log('onClick=>:',this);
      //   }
      // };

      // var buttonCustomUI = new customUI();
      // ui.addControl("buttonCustomUI", buttonCustomUI);
    },

    addEvent() {
      window.addEventListener("resize", function () {
        this.map.getViewPort().resize();
      });
      // eslint-disable-next-line
      let mapEvent = new H.mapevents.MapEvents(this.map);
      // eslint-disable-next-line
      let behavior = new H.mapevents.Behavior(mapEvent);
    },

    async getDeliverLocation() {
      try {
        if (this.ref_id != null) {
          let request = JSON.stringify({
            reference_id: this.ref_id,
            reference_item_id: this.ref_item_id,
          });
          // console.log("getParam =>", reference_id, '&', reference_item_id);
          var { data } = await this.axios.post(
            this.apiUrl + "TransportJob.api/EDI/tracking_transportjob_detail",
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
          await this.setViewBounds();
        }
      } catch (error) {
        console.log("getDeliverLocation =>", error.message);
      }
    },

    async getDriverLocation() {
      try {
        if (this.ref_id != null) {
          var { data } = await this.axios.post(
            this.apiUrl +
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

     setViewBounds() {
      // eslint-disable-next-line
      var group = new H.map.Group();

      group.addObjects(this.group.data);
      this.map.addObject(group);
      this.map.getViewModel().setLookAtData({
        bounds: group.getBoundingBox(),
      });
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

    // inherits: (childCtor, parentCtor) => {
    //   function tempCtor() {}
    //   tempCtor.prototype = parentCtor.prototype;
    //   childCtor.superClass_ = parentCtor.prototype;
    //   childCtor.prototype = new tempCtor();
    //   childCtor.prototype.constructor = childCtor;
    //   // eslint-disable-next-line
    //   childCtor.base = function (me, methodName, var_args) {
    //     var args = new Array(arguments.length - 2);
    //     for (var i = 2; i < arguments.length; i++) {
    //       args[i - 2] = arguments[i];
    //     }
    //     return parentCtor.prototype[methodName].apply(me, args);
    //   };
    // },

    dropMarker(position, type) {
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
      // type == "R" || type == "D"
      //   ? this.group.data.push(marker)
      //   : (this.driver_marker = marker);
      if (type == "R" || type == "D") {
        this.group.data.push(marker);
      } else {
        this.driver_marker = marker;
        console.log("before");
      }
      this.map.addObject(marker);
    },
    drawCircle(position, radius) {
      // eslint-disable-next-line
      let circle = new H.map.Circle(position, radius);
      this.map.addObject(circle);
    },
    drawLine(start, finish) {
      // eslint-disable-next-line
      let lineString = new H.geo.LineString();
      lineString.pushPoint(start);
      lineString.pushPoint(finish);
      // eslint-disable-next-line
      let polyline = new H.map.Polyline(lineString, {
        style: {
          lineWidth: 7,
        },
      });
      this.map.addObject(polyline);
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
