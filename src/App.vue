<template>
  <div id="app">
    <a-layout>
      <a-spin size="large" tip="Loading" :spinning="loading" />
      <a-affix :offset-top="0">
        <a-layout-header
          :style="{
            background: '#fff',
            padding: '0 16px',
          }"
        >
          <a-row type="flex" align="middle">
            <a-col flex="auto">
              <div
                :style="{
                  color: 'rgba(0,0,0,.85)',
                  fontSize: '16px',
                  lineHeight: '1.8',
                }"
              >
                {{ selectedCity.LocalizedName }}
              </div>
              <div
                :style="{
                  color: 'rgba(0,0,0,.45)',
                  fontSize: '12px',
                  lineHeight: '1.4',
                }"
              >
                {{ date }}
              </div>
            </a-col>
            <a-col flex="56px">
              <a
                :style="{
                  display: 'block',
                  fontSize: '16px',
                  textAlign: 'center',
                }"
                @click="handleDrawer"
              >
                <a-icon type="environment" theme="twoTone" />
              </a>
            </a-col>
          </a-row>
        </a-layout-header>
      </a-affix>

      <a-layout-content
        :style="{
          margin: '24px 16px',
          minHeight: '100vh',
        }"
      >
        <a-result v-show="error" status="error" title="Error">
          {{ error }}
        </a-result>

        <div
          :style="{
            display: 'flex',
            flexDirection: 'column',
            height: '80vh',
            justifyContent: 'space-between',
          }"
        >
          <a-row type="flex" justify="end" align="top">
            <img
              :style="{
                margin: '10px',
                width: '150px',
              }"
              :src="icons[weather.WeatherIcon - 1]"
            />
          </a-row>
          <div>
            <h1 :style="{ fontSize: '9rem', marginBottom: '0' }">
              {{ weather.Temperature.Metric.Value }}°
            </h1>
            <!-- prettier-ignore -->
            <p
              :style="{
                fontSize: '16px',
                lineHeight: '1.5',
                marginBottom: '20px'
              }"
            >
              {{weather.WeatherText}} {{weather.TemperatureSummary.Past24HourRange.Minimum.Metric.Value}} / {{weather.TemperatureSummary.Past24HourRange.Maximum.Metric.Value}} °C
            </p>
          </div>
        </div>

        <a-card>
          <a-descriptions
            layout="vertical"
            :column="{ xxl: 6, xl: 6, lg: 6, md: 3, sm: 3, xs: 2 }"
          >
            <a-descriptions-item
              :label="`${weather.Wind.Direction.Localized} wind`"
            >
              {{ weather.Wind.Speed.Metric.Value }}
              {{ weather.Wind.Speed.Metric.Unit }}
            </a-descriptions-item>
            <a-descriptions-item label="Feels like">
              {{ weather.RealFeelTemperature.Metric.Value }}°
            </a-descriptions-item>
            <a-descriptions-item label="Humidity">
              {{ weather.RelativeHumidity }} %
            </a-descriptions-item>
            <a-descriptions-item label="Visibility">
              {{ weather.Visibility.Metric.Value }}
              {{ weather.Visibility.Metric.Unit }}
            </a-descriptions-item>
            <a-descriptions-item label="UV">
              {{ weather.UVIndexText }}
            </a-descriptions-item>
            <a-descriptions-item label="Pressure">
              {{ weather.Pressure.Metric.Value }}
              {{ weather.Pressure.Metric.Unit }}
            </a-descriptions-item>
          </a-descriptions>
        </a-card>

        <!-- Debugging -->
        <!-- <a-descriptions title="Data" layout="vertical" bordered>
          <a-descriptions-item label="Selected City">
            {{ selectedCity }}
          </a-descriptions-item>
          <a-descriptions-item label="Cities">
            {{ cities }}
          </a-descriptions-item>
          <a-descriptions-item label="Weather">
            {{ weather }}
          </a-descriptions-item>
        </a-descriptions> -->
      </a-layout-content>
    </a-layout>

    <a-drawer
      title="Select location"
      placement="right"
      width="90%"
      :visible="drawer"
      @close="handleDrawer"
    >
      <!-- Search bar -->
      <a-input-search
        v-model="searchValue"
        placeholder="Search your city"
        :style="{ marginBottom: '10px' }"
        size="large"
        enter-button
        allow-clear
        @search="onSearch"
      />

      <!-- Search results -->
      <a-list v-show="searchValue" :data-source="searchResults">
        <a-list-item slot="renderItem" slot-scope="city, id">
          <a @click="addCity(city)" :key="id">
            {{ city.LocalizedName }},
            {{ city.AdministrativeArea.LocalizedName }},
            {{ city.Country.LocalizedName }}
          </a>
        </a-list-item>
      </a-list>

      <!-- Cities list -->
      <a-list v-show="!searchValue" :data-source="cities">
        <a-list-item slot="renderItem" slot-scope="city, id">
          <a-list-item-meta
            :description="`${city.AdministrativeArea.LocalizedName}, ${city.Country.LocalizedName}`"
          >
            <a slot="title" @click="selectCity(city)">
              {{ city.LocalizedName }}
            </a>
          </a-list-item-meta>
          <a-icon
            type="delete"
            theme="twoTone"
            two-tone-color="#ff4d4f"
            slot="actions"
            @click="deleteCity(id)"
          />
        </a-list-item>
      </a-list>
    </a-drawer>
  </div>
</template>

<script>
import axios from "axios";
import dayjs from "dayjs";

export default {
  name: "app",

  data() {
    return {
      loading: true,
      drawer: false,
      searchValue: "",
      searchResults: [],
      selectedCity: {},
      date: dayjs().format("dddd, MMM D YYYY, hh:mm"),
      cities: [],
      duplicate: null,
      weather: {},
      error: "",
      icons: [
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439159.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439159.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439159.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439129.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439129.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439116.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439116.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439307.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439307.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439307.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439307.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439245.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439123.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439237.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439133.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439218.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439199.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439373.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439146.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439153.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439153.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439153.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439153.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439153.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439103.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439282.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439294.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439164.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439402.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439402.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439108.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439108.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439108.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439123.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439123.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439218.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439218.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439363.svg",
        "https://www.flaticon.com/svg/static/icons/svg/3439/3439118.svg",
      ],
    };
  },

  async created() {
    try {
      const currentCity = await this.getIpInfo();
      const cityData = await this.onSearch(currentCity);
      await this.addCity(cityData[0]);
    } catch (error) {
      console.log(error);
      this.error = error;
    }
  },

  methods: {
    handleDrawer() {
      this.drawer = !this.drawer;
    },
    getIpInfo: async () => {
      const res = await axios.get("https://ipinfo.io/json");
      const city = res.data.city;
      return city;
    },
    async onSearch(searchValue) {
      try {
        // Debugging
        // const url = `https://my-json-server.typicode.com/vitalikda/WeatherApp/cities`;
        const url = `https://dataservice.accuweather.com/locations/v1/search?apikey=cVO3LSVok3soU7CPtgcqP8nyjdhpSckQ&q=${searchValue}`;
        const res = await axios.get(url);
        const data = res.data;
        this.searchResults = data;
        return data;
      } catch (error) {
        console.log(error);
        this.error = `AccuWeather Unavailable: ${error}`;
      }
    },
    selectCity(cityData) {
      this.selectedCity = cityData;
      this.drawer = false;
    },
    async addCity(cityData) {
      this.selectCity(cityData);
      this.loading = true;
      this.checkIfDuplicate(cityData.Key);
      if (!this.duplicate) {
        this.cities.unshift(cityData);
      }
      this.searchValue = "";
      this.searchResults = [];
      try {
        await this.getWeather(cityData.Key);
        this.loading = false;
      } catch (error) {
        console.log(error);
        this.error = error;
      }
    },
    checkIfDuplicate(cityId) {
      this.duplicate = this.cities.some((city) => {
        return city.Key === cityId;
      });
    },
    deleteCity(index) {
      this.cities.splice(index, 1);
    },
    async getWeather(cityId) {
      // Debugging
      // const url = `https://my-json-server.typicode.com/vitalikda/WeatherApp/currentconditions`;
      const url = `https://dataservice.accuweather.com/currentconditions/v1/ ${cityId} ?apikey=cVO3LSVok3soU7CPtgcqP8nyjdhpSckQ&details=true`;
      const res = await axios.get(url);
      const data = res.data[0];
      this.weather = data;
    },
  },
};
</script>

<style></style>
