<template>
  <div>
    <div ref="map" id="map"></div>
  </div>
</template>

<script>
import config from "../config/config";

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
      receiverMarkup:
        '<svg width="42" height="42" viewBox="0 0 42 42" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M33.6789 14.8706C33.6789 8.11125 28.1926 2.625 21.4332 2.625C14.6739 2.625 9.18762 8.11125 9.18762 14.8706C9.18762 17.115 9.79137 19.215 10.8414 21.0263L20.8032 39.0075C21.0789 39.5062 21.7876 39.5062 22.0632 39.0075L32.0251 21.0263C33.0751 19.215 33.6789 17.115 33.6789 14.8706Z" fill="#6FCF97"/><path d="M21.4998 23.2444C26.1245 23.2444 29.8736 19.4953 29.8736 14.8706C29.8736 10.2459 26.1245 6.49687 21.4998 6.49687C16.8752 6.49687 13.1261 10.2459 13.1261 14.8706C13.1261 19.4953 16.8752 23.2444 21.4998 23.2444Z" fill="white"/></svg>',
      deliverMarkup:
        '<svg width="42" height="42" viewBox="0 0 42 42" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M33.6789 14.8706C33.6789 8.11125 28.1926 2.625 21.4332 2.625C14.6739 2.625 9.18762 8.11125 9.18762 14.8706C9.18762 17.115 9.79137 19.215 10.8414 21.0263L20.8032 39.0075C21.0789 39.5062 21.7876 39.5062 22.0632 39.0075L32.0251 21.0263C33.0751 19.215 33.6789 17.115 33.6789 14.8706Z" fill="#EC870E"/><path d="M21.4998 23.2444C26.1245 23.2444 29.8736 19.4953 29.8736 14.8706C29.8736 10.2459 26.1245 6.49687 21.4998 6.49687C16.8752 6.49687 13.1261 10.2459 13.1261 14.8706C13.1261 19.4953 16.8752 23.2444 21.4998 23.2444Z" fill="white"/></svg>',
      driverMarkup:
        '<svg width="54" height="78" viewBox="0 0 14 38" fill="none" xmlns="http://www.w3.org/2000/svg"><g clip-path="url(#clip0)"><path d="M11.951 10.9793H2.04907V26.9931H11.951V10.9793Z" fill="#71876D"/><path d="M11.9118 25.8906C11.7549 24.9259 11.3823 24.2185 10.8431 23.8969C10.3137 23.5754 8.07842 22.9323 6.99999 22.9323C5.92156 22.9323 3.69607 23.5754 3.15685 23.8969C2.62744 24.2185 2.24509 24.9259 2.08822 25.8906C1.93136 26.8553 1.7843 32.4138 2.81371 34.2972C3.84312 36.1898 3.99018 36.9708 5.44116 36.9708C6.88234 36.9708 6.99999 36.9708 6.99999 36.9708C6.99999 36.9708 7.11763 36.9708 8.55881 36.9708C9.99999 36.9708 10.1568 36.1898 11.1863 34.2972C12.2157 32.4046 12.0588 26.8553 11.9118 25.8906Z" fill="#68B651"/><path d="M9.81369 23.0976C9.81369 23.0976 10.2843 25.615 9.94114 29.8872C9.598 34.1594 9.15683 36.934 8.69604 36.9616C8.22545 36.9891 7.00977 36.9616 7.00977 36.9616C7.00977 36.9616 5.79408 36.9891 5.32349 36.9616C4.8529 36.934 4.42153 34.1502 4.07839 29.8872C3.73526 25.6242 4.20585 23.0976 4.20585 23.0976" fill="#63A94B"/><path d="M7.00005 11.0069C6.46083 11.0069 6.01965 10.5934 6.01965 10.0881V1.54375C6.01965 1.03844 6.46083 0.625 7.00005 0.625C7.53926 0.625 7.98044 1.03844 7.98044 1.54375V10.0881C7.98044 10.5934 7.53926 11.0069 7.00005 11.0069Z" fill="#4E4D4D"/><path d="M9.23524 9.68387C9.23524 9.68387 9.23524 6.90006 9.23524 6.20181C9.23524 5.50356 9.17641 3.96006 8.8823 2.57274C8.58818 1.18543 7.62739 1.20381 6.99994 1.20381C6.37249 1.20381 5.41171 1.17624 5.11759 2.57274C4.82347 3.96924 4.76465 5.51274 4.76465 6.21099C4.76465 6.90924 4.76465 9.69306 4.76465 9.69306" fill="#68B651"/><path d="M7.00005 11.209C10.0001 11.209 11.951 12.5228 11.951 12.5228V10.5016C10.6373 9.26125 8.27456 9.05913 7.00005 9.05913C5.72554 9.05913 3.3726 9.26125 2.04907 10.5016V12.5228C2.04907 12.5228 4.00005 11.209 7.00005 11.209Z" fill="#609E48"/><path d="M7.00001 25.8998C9.73531 25.8998 9.4706 23.6856 9.4706 23.1803C9.4706 22.6842 8.13727 19.6799 8.03923 19.3399C7.94119 19.0919 7.73531 19.1102 7.73531 19.1102H7.00001H6.26472C6.26472 19.1102 6.06864 19.0919 5.9608 19.3399C5.86276 19.6799 4.52943 22.6842 4.52943 23.1803C4.53923 23.6764 4.26472 25.8998 7.00001 25.8998Z" fill="#513520"/><path d="M6.99996 35.8591H5.85291V37.2556C5.85291 37.3199 5.91173 37.375 5.98036 37.375H6.99996H8.01957C8.0882 37.375 8.13722 37.3199 8.13722 37.2648V35.8683H6.99996V35.8591Z" fill="#BE1622"/><path d="M8.46083 34.5636H7.00005H5.54907C5.47063 34.5636 5.42161 34.6279 5.43142 34.6923L5.6373 36.0428C5.64711 36.0979 5.69612 36.1439 5.75495 36.1439H7.00985H8.26475C8.32358 36.1439 8.3726 36.0979 8.3824 36.0428L8.58828 34.6923C8.58828 34.6279 8.52946 34.5636 8.46083 34.5636Z" fill="#AFD39A"/><path d="M12.9118 13.561L6.99999 12.4034L1.09802 13.561L0.941162 12.8903L6.99999 11.7051L13.0588 12.8903L12.9118 13.561Z" fill="#474551"/><path d="M7.00004 10.5475C7.5415 10.5475 7.98044 10.4364 7.98044 10.2994C7.98044 10.1624 7.5415 10.0514 7.00004 10.0514C6.45859 10.0514 6.01965 10.1624 6.01965 10.2994C6.01965 10.4364 6.45859 10.5475 7.00004 10.5475Z" fill="#F9B233"/><path d="M8.45102 11.0436C8.45102 10.4373 8.45102 10.3178 8.45102 10.2994H7.00004H5.54906C5.54906 10.3178 5.54906 10.4464 5.54906 11.0436C5.54906 11.8981 2.14709 12.4493 2.14709 12.4493L2.35298 13.4599C2.35298 13.4599 3.97062 13.0189 5.04906 13.0189C6.12749 13.0189 6.32357 13.3589 7.00004 13.3589C7.68631 13.3589 7.88239 13.0189 8.95102 13.0189C10.0294 13.0189 11.6471 13.4599 11.6471 13.4599L11.853 12.4493C11.853 12.4493 8.45102 11.8981 8.45102 11.0436Z" fill="#68B651"/><path d="M7.00002 13.2303C7.3303 13.2303 7.59806 12.9793 7.59806 12.6698C7.59806 12.3603 7.3303 12.1094 7.00002 12.1094C6.66973 12.1094 6.40198 12.3603 6.40198 12.6698C6.40198 12.9793 6.66973 13.2303 7.00002 13.2303Z" fill="#319037"/><path d="M7.00976 24.7697H6.99015C4.77446 24.7697 4.39211 23.0976 4.20584 22.2891C4.03917 21.5633 3.55878 19.0643 3.34309 17.1901C3.30387 16.8134 3.58819 16.4734 3.99015 16.4367C4.39211 16.3999 4.75485 16.6664 4.79407 17.0431C4.99995 18.8622 5.47054 21.2969 5.6274 21.9951C5.8627 23.0241 6.1078 23.3916 6.98035 23.3916H6.99995C7.88231 23.3916 8.1176 23.0241 8.3529 21.9951C8.51956 21.2877 8.99015 18.8622 9.18623 17.0431C9.22544 16.6664 9.58819 16.3907 9.99015 16.4367C10.3921 16.4734 10.6862 16.8134 10.6372 17.1901C10.4313 19.0643 9.94113 21.5633 9.77446 22.2891C9.6078 23.0976 9.22544 24.7697 7.00976 24.7697Z" fill="#363851"/><path d="M7.00005 21.8481C5.05888 21.8481 3.48045 21.2509 2.29417 20.0749C-0.235239 17.5668 5.52539e-05 13.3313 0.00985918 13.1476C0.029467 12.7709 0.382408 12.4861 0.784369 12.5044C1.18633 12.5228 1.49025 12.8536 1.47064 13.2303C1.47064 13.2854 1.27456 17.0706 3.3628 19.1378C4.26476 20.029 5.49025 20.4792 7.00986 20.4792C8.52947 20.4792 9.75496 20.029 10.6569 19.1378C12.755 17.0614 12.5491 13.267 12.5491 13.2303C12.5295 12.8536 12.8334 12.532 13.2353 12.5044C13.6373 12.4861 13.9804 12.7709 14.0099 13.1476C14.0197 13.3313 14.255 17.5668 11.7255 20.0749C10.5197 21.2509 8.94123 21.8481 7.00005 21.8481Z" fill="#FDD8AC"/><path d="M7 21.8481C1.87255 21.8481 0.0196078 17.5943 0 13.6069L1.46078 13.5977C1.47059 15.2147 1.90196 20.4792 7 20.4792C12.098 20.4792 12.5294 15.2147 12.5392 13.5977L14 13.6069C13.9804 17.5943 12.1275 21.8481 7 21.8481Z" fill="#6ABE9F"/><path d="M9.77457 20.5435C9.68633 20.9661 9.58829 21.4439 9.50006 21.8573C9.26476 22.9598 8.67653 23.5846 7.65692 23.5846H6.353C5.33339 23.5846 4.73535 22.9506 4.50986 21.8573C4.42163 21.4439 4.32359 20.9753 4.23535 20.5435H9.77457Z" fill="#6ABE9F"/><path d="M9.77453 18.5866C9.77453 20.1392 8.53924 21.3979 7.00982 21.3979C5.48041 21.3979 4.24512 20.1392 4.24512 18.5866C4.24512 18.3752 4.26473 18.1639 4.31374 17.9618C4.58825 16.7123 5.6961 15.7752 7.00982 15.7752C8.32355 15.7752 9.43139 16.7123 9.7059 17.9618C9.74512 18.1639 9.77453 18.3752 9.77453 18.5866Z" fill="#B0D9C6"/><path d="M9.77453 18.5866C9.77453 20.1392 8.53924 21.3979 7.00982 21.3979C5.48041 21.3979 4.24512 20.1392 4.24512 18.5866C4.24512 18.3752 4.26473 18.1639 4.31374 17.9618C4.78433 16.9328 5.82355 16.2162 7.00982 16.2162C8.1961 16.2162 9.23531 16.9328 9.7059 17.9618C9.74512 18.1639 9.77453 18.3752 9.77453 18.5866Z" fill="#6ABE9F"/><path d="M7.00003 20.9569C5.80396 20.9569 4.77454 20.2403 4.30396 19.2113C4.57846 20.4608 5.68631 21.3979 7.00003 21.3979C8.31376 21.3979 9.4216 20.4608 9.69611 19.2113C9.22553 20.2403 8.19611 20.9569 7.00003 20.9569Z" fill="#599E82"/></g><defs><clipPath id="clip0"><rect width="14" height="36.75" fill="white" transform="translate(0 0.625)"/></clipPath></defs></svg>',
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
          // center: { lat: 0, lng: 0 },
          zoom: 10,
          padding: { top: 200, left: 200, bottom: 200, right: 200 },
          pixelRatio:
            window.devicePixelRatio && window.devicePixelRatio > 1 ? 2 : 1,
        }
      );

      //eslint-disable-next-line
      this.ui = H.ui.UI.createDefault(this.map, this.defaultLayers);

      var setViewBounds = this.setViewBounds;
      let svg =
        '<svg class="H_icon H_icon" viewBox="0 0 25 25">' +
        '<path d="M 18.5,11 H 14 V 6.5 c 0,-.8 -.7,-1.5 -1.5,-1.5 -.8,0 -1.5,.7 -1.5,1.5 V 11 H 6' +
        ".5 C 5.7,11 5,11.7 5,12.5 5,13.3 5.7,14 6.5,14 H 11 v 4.5 c 0,.8 .7,1.5 1.5,1.5 .8,0 1.5," +
        '-.7 1.5,-1.5 V 14 h 4.5 C 19.3,14 20,13.3 20,12.5 20,11.7 19.3,11 18.5,11 z" />' +
        "</svg>";
      let goToDriver = this.goToDriver;
      var customUI = this.createButton(svg, setViewBounds);
      var customUI2 = this.createButton(svg, goToDriver);

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

        this.setAlignment(options["alignment"] || "right-top");

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
      this.setViewBounds()
    },

    setViewBounds() {
      let bounds = this.group.getBoundingBox();
      this.map.getViewModel().setLookAtData(
        {
          bounds: bounds,
          position: bounds.getCenter(),
        },
        true
      );
    },

    goToDriver() {
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
