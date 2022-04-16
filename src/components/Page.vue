<template>
  <div>
    <b-form-input class="input" v-model="text" v-on:change="Geocode" placeholder="Enter place"></b-form-input>
    <b-form-input class="input" v-model="radius" placeholder="Enter radius in meters"></b-form-input>
    <b-form-select class="input" v-model="selected" :options="options" v-on:change="Call"></b-form-select>

    <b-card class="weather" v-if="weather">
      <h8 id="state"><b>Weather:</b> {{ weather.weather[0].description }} <img :src="weatherImage" class="weatherImg"/></h8>
      <table>
        <tr>
          <td><b>Temperature:</b> {{ (parseFloat(weather.main.temp) - 273.1).toFixed(1) }}°C</td>
          <td><b>Humidity:</b> {{ weather.main.humidity }}%</td>
          <td><b>Wind speed:</b> {{ weather.wind.speed }}mps</td>
        </tr>
        <tr>
          <td><b>Feels like:</b> {{ (parseFloat(weather.main.feels_like) - 273.1).toFixed(1) }}°C</td>
          <td><b>Pressure:</b> {{ weather.main.pressure }}hPa</td>
          <td><b>Wind direction:</b> {{ weather.wind.deg }}°</td>
        </tr>
      </table>
    </b-card>

    <b-card class="places"
            v-for="place in placesOnPage"
            :key="place.properties.xid"
            :title="place.properties.name">
      <b-button
          id="button"
          pill variant="outline-dark"
          v-b-toggle.sidebar-right
          v-on:click="Info(place.properties.xid, place.properties.name)">Show info
      </b-button>
    </b-card>

    <b-pagination
        pills
        class="pagination"
        v-model="currentPage"
        :total-rows="rows"
        :per-page="perPage"
        aria-controls="my-table"
        v-show="isDisabled"
    ></b-pagination>


    <b-sidebar id="sidebar-right"
               width="50%"
               :title="placeName"
               v-if="description"
               bg-variant="dark"
               text-variant="light"
               right shadow="lg">
      <div class="px-3 py-2">
        <div class="d-block text-center">
          <img style="padding-top: 5px" :src="description.image" alt="Sorry, nothing to show :^(" class="image"/>
          <div style="padding-top: 25px" v-html="description_info"></div>
        </div>
        <div>
          <h4 style="padding-left: 240px; padding-top: 25px"><b>Learn more: </b><a :href="description.wikipedia">Wikipedia</a></h4>
        </div>
      </div>
    </b-sidebar>


  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: "Page",
  data() {
    return {
      selected: 0,
      options: [],
      hits: null,
      lat: 0,
      lng: 0,
      text: "",
      radius: "",
      places: [],
      description: null,
      weather: null,
      detailsDialogVisible: false,
      placeName: "",
      perPage: 5,
      currentPage: 1,
      isDisabled: false
    }
  },
  methods: {
    async Geocode() {
      try {
        const response = await axios.get('https://graphhopper.com/api/1/geocode', {
          params: {
            q: this.text,
            key: "14b971b8-3432-425f-87e2-c4fd19cbcb12",
            locale: "en"
          }
        });
        this.hits = response.data.hits;
        console.log(this.hits);
        this.options = [];
        for (const hit in this.hits) {
          let elem = {};
          elem.value = this.hits[hit].point.lat + this.hits[hit].point.lng;
          elem.text = this.hits[hit].country;
          if (this.hits[hit].state !== undefined || this.hits[hit].city !== undefined || this.hits[hit].name !== undefined) {
            if (this.hits[hit].state !== undefined) {
              elem.text += ", " + this.hits[hit].state;
            }
            if (this.hits[hit].city !== undefined) {
              elem.text += ", " + this.hits[hit].city;
            }
            if (this.hits[hit].name !== undefined) {
              elem.text += ", " + this.hits[hit].name;
            }
            this.options.push(elem);
          }

          //elem.text = this.hits[hit].country + ", " + this.hits[hit].state + ", " + this.hits[hit].city + ", " + this.hits[hit].name;
          //this.options.push(elem);
        }
      } catch (e) {
        console.log(e);
      }
    },
    async Weather(hit) {
      const response = await axios.get('https://api.openweathermap.org/data/2.5/weather', {
        params: {
          lat: this.hits[hit].point.lat,
          lon: this.hits[hit].point.lng,
          appid: "23fb2d4c1104c2c1ec9c0e2ccd9866e1"
        }
      });
      console.log(response);
      this.weather = response.data
    },
    async PlacesInRadius(hit) {
      const response = await axios.get('http://api.opentripmap.com/0.1/en/places/radius', {
        params: {
          lon: this.hits[hit].point.lng,
          lat: this.hits[hit].point.lat,
          radius: this.radius,
          apikey: "5ae2e3f221c38a28845f05b65848fa8a1459875c9cb4958916e1f0f5"
        }
      });
      console.log(response);
      this.places = response.data.features.filter(function (e) {
        return e.properties.name !== ""
      })
      this.isDisabled = true;
      console.log("places");
      console.log(this.places);
    },
    async Call() {
      for (const hit in this.hits) {
        let temp = this.hits[hit].point.lat + this.hits[hit].point.lng;
        if (temp === this.selected) {
          try {
            this.Weather(hit);
            this.PlacesInRadius(hit);
          } catch (e) {
            console.log(e);
          }
        }
      }
    },
    async Info(xid, name) {
      this.placeName = name;
      try {
        const response = await axios.get('http://api.opentripmap.com/0.1/en/places/xid/' + xid, {
          params: {
            apikey: "5ae2e3f221c38a28845f05b65848fa8a1459875c9cb4958916e1f0f5"
          }
        });
        console.log(response)
        this.description = response.data
      } catch (e) {
        console.log(e);
      }
    }
  },
  computed: {
    // eslint-disable-next-line vue/return-in-computed-property
    description_info: function () {
      if (this.description != null) {
        if (this.description.info != null) {
          return this.description.info.descr
        }
      } else {
        return "Nothing to tell"
      }
    },
    // eslint-disable-next-line vue/return-in-computed-property
    placesOnPage: function () {
      return this.places.slice((this.currentPage - 1) * this.perPage, (this.currentPage - 1) * this.perPage + this.perPage)
    },
    rows() {
      return this.places.length
    },
    // eslint-disable-next-line vue/return-in-computed-property
    weatherImage: function () {
      return "https://openweathermap.org/img/wn/" + this.weather.weather[0].icon + "@2x.png"
    }
  }
}
</script>


<style scoped>
.weather {
  color: rgba(128, 28, 168, 0.68);
  font-size: medium;
  height: 150px;
  margin-bottom: 10px;
  margin-top: 10px;
}
.image {
  width: 500px;
}
.pagination {
  justify-content: center;
  margin-top: 10px;
}
td {
  padding-left: 62px;
}
.weatherImg {
  width: 50px;
}
.places {
  color: rgba(138, 28, 168, 0.68);
  height: 75px;
}
.input {
  color: rgba(110, 22, 145, 0.68);
}
#state {
  padding-left: 230px;
}
#button {
  margin-top: -85px;
  margin-left: 580px;
}
</style>