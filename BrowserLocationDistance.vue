<template>
  <table>
    <tr>
      <th>Location</th>
      <th>Distance</th> 
    </tr>
    <tr v-for="location in list" v-bind:key="location.name">
      <td>{{ location.name }}</td>
      <td>{{ location.distance }} {{ unitLabel }}</td>
    </tr>
  </table>
</template>

<script>
if (typeof helper == "undefined") {
  var helper = {};
}

helper.arr = {
  /**
   * Function to sort multidimensional array
   *
   * <a href="/param">@param</a> {array} [arr] Source array
   * <a href="/param">@param</a> {array} [columns] List of columns to sort
   * <a href="/param">@param</a> {array} [order_by] List of directions (ASC, DESC)
   * @returns {array}
   */
  multisort: function(arr, columns, order_by) {
    if (typeof columns == "undefined") {
      columns = [];
      for (x = 0; x < arr[0].length; x++) {
        columns.push(x);
      }
    }

    if (typeof order_by == "undefined") {
      order_by = [];
      for (x = 0; x < arr[0].length; x++) {
        order_by.push("ASC");
      }
    }

    function multisort_recursive(a, b, columns, order_by, index) {
      var direction = order_by[index] == "DESC" ? 1 : 0;

      var is_numeric = !isNaN(+a[columns[index]] - +b[columns[index]]);

      var x = is_numeric ? +a[columns[index]] : a[columns[index]].toLowerCase();
      var y = is_numeric ? +b[columns[index]] : b[columns[index]].toLowerCase();

      if (x < y) {
        return direction == 0 ? -1 : 1;
      }

      if (x == y) {
        return columns.length - 1 > index
          ? multisort_recursive(a, b, columns, order_by, index + 1)
          : 0;
      }

      return direction == 0 ? 1 : -1;
    }

    return arr.sort(function(a, b) {
      return multisort_recursive(a, b, columns, order_by, 0);
    });
  }
};

export default {
  data: function() {
    return {
      list: [],
      coordinates: [],
      unitLabel: 'mi'
    };
  },
  props: ["locations", "unit", "coordinates", "order"],
  methods: {
    distance: function (lat1, lon1, lat2, lon2, unit) {
      var radlat1 = Math.PI * lat1/180;
      var radlat2 = Math.PI * lat2/180;
      var theta = lon1 - lon2;
      var radtheta = Math.PI * theta/180;
      var dist = Math.sin(radlat1) * Math.sin(radlat2) + Math.cos(radlat1) * Math.cos(radlat2) * Math.cos(radtheta);
      dist = Math.acos(dist);
      dist = dist * 180/Math.PI;
      dist = dist * 60 * 1.1515;

      if (unit == "K") { 
        dist = dist * 1.609344;
        this.unitLabel = 'km';
      }
      
      if (unit == "N") { 
        dist = dist * 0.8684;
        this.unitLabel = 'n.m.';
      } 

      return dist
    },
    getGeolocation: function() {
      var self = this;

      var apiGeolocationSuccess = function(position) {
        self.fillAndSortList(
          position.coords.latitude,
          position.coords.longitude
        );
      };

      var tryAPIGeolocation = function() {
        axios
          .get(`http://ip-api.com/json`)
          .then(response => {
            apiGeolocationSuccess({
              coords: {
                latitude: response.data.lat,
                longitude: response.data.lon
              }
            });
          })
          .catch(e => {
            alert("API Geolocation error! \n\n" + e);
          });
      };

      var browserGeolocationSuccess = function(position) {
        this.fillAndSortList(
          position.coords.latitude,
          position.coords.longitude
        );
      };

      var browserGeolocationFail = function(error) {
        switch (error.code) {
          case error.TIMEOUT:
            alert("Browser geolocation error !\n\nTimeout.");
            break;
          case error.PERMISSION_DENIED:
            if (error.message.indexOf("Only secure origins are allowed") == 0) {
              tryAPIGeolocation();
            }
            break;
          case error.POSITION_UNAVAILABLE:
            alert("Browser geolocation error !\n\nPosition unavailable.");
            break;
        }
      };

      
      
      if (this.coordinates) {
        this.coordinates = this.coordinates.split(',');
        this.locations.forEach(function(element) {
          self.list.push({
            name: element.name,
            distance: parseFloat(
              self.distance(element.lat, element.lon, self.coordinates[0].trim(), self.coordinates[1].trim(), self.unit).toFixed(2)
            )
          });
        });

        if (!this.order) {
          this.order = 'ASC';
        }

        this.list = helper.arr.multisort(this.list, ['distance'], [this.order]);
      } else {
        if (navigator.geolocation) {
          navigator.geolocation.getCurrentPosition(
            browserGeolocationSuccess,
            browserGeolocationFail,
            { maximumAge: 50000, timeout: 20000, enableHighAccuracy: true }
          );
        }
      }
    },
    fillAndSortList: function(lat, lng) {
      var self = this;

      this.locations.forEach(function(element) {
        self.list.push({
          name: element.name,
          distance: parseFloat(
            self.distance(element.lat, element.lon, lat, lng, self.unit).toFixed(2)
          )
        });
      });

      this.list = helper.arr.multisort(this.list, ['distance'], ['ASC']);
    }
  },
  mounted() {
    this.getGeolocation();
  }
};
</script>

<style scoped>

</style>