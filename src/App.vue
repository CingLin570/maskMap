<template>
  <div id="app">
    <div class="row no-gutters">
      <div class="col-sm-3">
        <div class="toolbox">
          <div class="sticky-top bg-white shadow-sm p-2">
            <div class="form-group d-flex">
              <label for="cityName" class="mr-2 col-form-label text-right"
                >縣市</label
              >
              <div class="flex-fill">
                <select id="cityName" class="form-control" v-model="select.city"
                @change="select.area = ''">
                  <option value="" disabled>-- Select One --</option>
                  <option :value="item.CityName" v-for="(item, i) in cityName" :key="i">
                    {{ item.CityName }}</option>
                </select>
              </div>
            </div>
            <div class="form-group d-flex">
              <label for="area" class="mr-2 col-form-label text-right"
                >地區</label
              >
              <div class="flex-fill">
                <select id="area" class="form-control" v-model="select.area"
                @change="removeMarker(); getpharmacies()">
                  <option value="" disabled>-- Select One --</option>
                  <option :value="item.AreaName"
                  v-for="item in cityName.find((city) => city.CityName === select.city).AreaList"
                  :key="item.AreaName">
                    {{ item.AreaName }}
                    </option>
                </select>
              </div>
            </div>
            <p class="mb-0 small text-muted text-right">
              請先選擇區域查看（綠色表示還有口罩）
            </p>
          </div>
          <ul class="list-group">
            <template v-for="(item, i) in filterList">
              <a class="list-group-item text-left text-black" :key="i"
              @click="penTo(item)">
                <h3>{{ item.properties.name }}</h3>
                <p class="mb-0">
                  成人口罩：{{ item.properties.mask_adult }} | 兒童口罩：{{ item.properties.mask_child }}</p>
                <p class="mb-0">
                  地址：<a
                    :href="`https://www.google.com/maps/place/${item.properties.address}`"
                    target="_blank"
                    title="Google Map"
                  >
                    {{ item.properties.address }}</a
                  >
                </p>
              </a>
            </template>
          </ul>
        </div>
      </div>
      <div class="col-sm-9">
        <div id="map"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityName from './assets/cityName.json';

let osmMap = {};
console.log(L);

const iconsConfig = {
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
};

const icons = {
  green: new L.Icon({
    iconUrl: 'https://cdn.rawgit.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-green.png',
    ...iconsConfig,
  }),
  grey: new L.Icon({
    iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-grey.png',
    ...iconsConfig,
  }),
};

export default {
  name: 'App',
  components: {},
  data() {
    return {
      data: [],
      cityName,
      cityArea: {},
      select: {
        city: '臺北市',
        area: '大安區',
      },
    };
  },
  methods: {
    getpharmacies() {
      const pharmacies = this.data.filter((item) => (
        item.properties.county === this.select.city)
        && (item.properties.town === this.select.area));
      const icon = icons.green;
      pharmacies.forEach((item) => {
        L.marker([
          item.geometry.coordinates[1],
          item.geometry.coordinates[0],
        ], {
          icon,
        }).addTo(osmMap).bindPopup(`<div><h2 class="h3">藥局名稱:${item.properties.name}</h2>
        <ul class="list-unstyled font-16">
          <li>
            口罩剩餘:成人${item.properties.mask_adult ? `${item.properties.mask_adult}片` : 0},
            小孩${item.properties.mask_child ? `${item.properties.mask_child}片` : 0}
          </li>
          <li>地址:<a href="https://www.google.com/maps/place/${item.properties.address}">${item.properties.address}</a></li>
          <li>電話:${item.properties.phone}</li>
          <li>最後更新時間:${item.properties.updated}</li>
        </ul></div>`);
      });
      this.updateMarker();
    },
    removeMarker() {
      osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          osmMap.removeLayer(layer);
        }
      });
    },
    updateMarker() {
      const pharmacy = this.data.find((item) => (item.properties.town === this.select.area)
      && (item.properties.county === this.select.city));
      const { geometry, properties } = pharmacy;
      console.log(pharmacy);
      const icon = properties.mask_adult || properties.mask_child ? icons.green : icons.grey;
      L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
        icon,
      }).addTo(osmMap).bindPopup(`<strong>${properties.name}</strong> <br>
      口罩剩餘：<strong>成人 - ${properties.mask_adult ? `${properties.mask_adult} 個` : '未取得資料'}/ 兒童 - ${properties.mask_child ? `${properties.mask_child} 個` : '未取得資料'}</strong><br>
      地址: <a href="https://www.google.com.tw/maps/place/${properties.address}" target="_blank">${properties.address}</a><br>
      電話: ${properties.phone}<br>
      <small>最後更新時間: ${properties.updated}</small>`);
      osmMap.panTo(new L.LatLng(geometry.coordinates[1], geometry.coordinates[0]));
    },
    penTo(pharmacy) {
      const { properties, geometry } = pharmacy;
      const icon = properties.mask_adult || properties.mask_child ? icons.green : icons.grey;
      L.marker([geometry.coordinates[1], geometry.coordinates[0]], {
        icon,
      }).addTo(osmMap).bindPopup(`<strong>${properties.name}</strong> <br>
      口罩剩餘：<strong>成人 - ${properties.mask_adult ? `${properties.mask_adult} 個` : '未取得資料'}/ 兒童 - ${properties.mask_child ? `${properties.mask_child} 個` : '未取得資料'}</strong><br>
      地址: <a href="https://www.google.com.tw/maps/place/${properties.address}" target="_blank">${properties.address}</a><br>
      電話: ${properties.phone}<br>
      <small>最後更新時間: ${properties.updated}</small>`).openPopup();
      osmMap.panTo(new L.LatLng(geometry.coordinates[1], geometry.coordinates[0]));
    },
  },
  computed: {
    filterList() {
      const vm = this;
      return this.data.filter((item) => (item.properties.county === vm.select.city)
        && (item.properties.town === vm.select.area));
    },
  },
  mounted() {
    const url = 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json';
    this.$http.get(url).then((response) => {
      this.data = response.data.features;
      console.log(this.data);
      this.getpharmacies();
    });
    // 插入新的圖層
    osmMap = L.map('map', { // 選擇id
      center: [25.025664, 121.536221], // 座標
      zoom: 16, // 放大縮小
    });
    // 圖磚,載入的地圖種類
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors', // 授權
    }).addTo(osmMap);
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';
#map {
  height: 100vh;
  overflow: hidden;
}
.text-black {
  color: #000;
}
.text-black:hover {
  color: #000;
}
.highlight {
  background: #e9ffe3;
}
.toolbox {
  height: 100vh;
  overflow-y: auto;
  a {
    cursor: pointer;
  }
}
.font-16{
  font-size: 16px;
}
a:hover{
  text-decoration: none;
}
</style>
