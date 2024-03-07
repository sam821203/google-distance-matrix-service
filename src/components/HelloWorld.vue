<script setup>
import { ref, reactive, onMounted, shallowRef, toRaw, watch } from 'vue'

const map = shallowRef(null)
const infowindowAll = ref({})
const location = reactive({
  lat: 25.0374865, // 經度
  lng: 121.5647688 // 緯度
})

const nightMode = ref('close') // 夜間模式：open開啟

const nightModeStyles = ref([
  { elementType: 'geometry', stylers: [{ color: '#242f3e' }] },
  { elementType: 'labels.text.stroke', stylers: [{ color: '#242f3e' }] },
  { elementType: 'labels.text.fill', stylers: [{ color: '#746855' }] },
  {
    featureType: 'administrative.locality',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#d59563' }]
  },
  {
    featureType: 'poi',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#d59563' }]
  },
  {
    featureType: 'poi.park',
    elementType: 'geometry',
    stylers: [{ color: '#263c3f' }]
  },
  {
    featureType: 'poi.park',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#6b9a76' }]
  },
  {
    featureType: 'road',
    elementType: 'geometry',
    stylers: [{ color: '#38414e' }]
  },
  {
    featureType: 'road',
    elementType: 'geometry.stroke',
    stylers: [{ color: '#212a37' }]
  },
  {
    featureType: 'road',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#9ca5b3' }]
  },
  {
    featureType: 'road.highway',
    elementType: 'geometry',
    stylers: [{ color: '#746855' }]
  },
  {
    featureType: 'road.highway',
    elementType: 'geometry.stroke',
    stylers: [{ color: '#1f2835' }]
  },
  {
    featureType: 'road.highway',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#f3d19c' }]
  },
  {
    featureType: 'transit',
    elementType: 'geometry',
    stylers: [{ color: '#2f3948' }]
  },
  {
    featureType: 'transit.station',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#d59563' }]
  },
  {
    featureType: 'water',
    elementType: 'geometry',
    stylers: [{ color: '#17263c' }]
  },
  {
    featureType: 'water',
    elementType: 'labels.text.fill',
    stylers: [{ color: '#515c6d' }]
  },
  {
    featureType: 'water',
    elementType: 'labels.text.stroke',
    stylers: [{ color: '#17263c' }]
  }
])

// 建立地圖
const initMap = () => {
  // 透過 Map 物件建構子建立新地圖 map 物件實例，並將地圖呈現在 id 為 map 的元素中
  map.value = new google.maps.Map(document.getElementById('map'), {
    // 設定地圖的中心點經緯度位置
    center: { lat: location.lat, lng: location.lng },
    // 設定地圖縮放比例 0-20
    zoom: 15,
    // 限制使用者能縮放地圖的最大比例
    maxZoom: 20,
    // 限制使用者能縮放地圖的最小比例
    minZoom: 3,
    // 設定是否呈現右下角街景小人
    streetViewControl: false,
    // 設定是否讓使用者可以切換地圖樣：一般、衛星圖等
    mapTypeControl: false,
    styles: nightModeStyles.value,
    mapId: 'map_id'
  })
}

const setMarker = async () => {
  const { AdvancedMarkerElement } = await google.maps.importLibrary('marker')

  // 放置多個marker
  fetch('./src/services/apis/map.geojson')
    .then((response) => {
      if (!response.ok) {
        throw new Error('Network response was not ok')
      }
      return response.json()
    })
    .then((result) => {
      let res = result.features

      res.forEach((location) => {
        let latLng = new google.maps.LatLng(
          location.geometry.coordinates[0],
          location.geometry.coordinates[1]
        )
        const beachFlagImg = document.createElement('img')
        beachFlagImg.src =
          'https://developers.google.com/maps/documentation/javascript/examples/full/images/beachflag.png'

        // 設定地標要放在哪一個地圖
        let marker = new AdvancedMarkerElement({
          map: map.value,
          position: latLng,
          content: beachFlagImg,
          title: 'Uluru'
        })

        // info window
        let infowindow = new google.maps.InfoWindow({
          content: `<h6>${location.properties.name}</h6>` // 支援html
        })

        // 監聽 marker click 事件
        marker.addListener('click', (e) => {
          infowindow.open(map.value, marker)
        })

        // 加一個open的method，就可由外部按鈕開啟
        // infowindowAll.value[location.properties.id] = {
        //   open: function () {
        //     infowindow.open(map.value, marker)
        //   }
        // }
      })
    })
}

// const switchMode = () => {
//   nightMode.value = 'open'
// }

// nightMode的值變動時，切換地圖的日間/夜間樣式
// watch(
//   () => nightMode.value,
//   (newVal) => {
//     console.log('nightMode', newVal)
//     if (newVal === 'open') {
//       map.value.setOptions({
//         styles: nightModeStyles.value
//       })
//     } else {
//       map.value.setOptions({
//         styles: null
//       })
//     }
//   }
// )

onMounted(() => {
  initMap()
  setMarker()
})
</script>

<template>
  <div class="container mt-4">
    <h2 class="text-center text-secondary pb-2">台北市營運餐廳</h2>
    <button @click="switchMode">開啟</button>
    <div class="map-container border rounded">
      <ul class="nav justify-content-center border-bottom">
        <!--營運地區 nav-->
      </ul>
      <!--地圖呈現在此-->
      <div class="google-map" id="map"></div>
    </div>
  </div>
</template>

<style scoped>
.google-map {
  width: 100%;
  height: 400px;
}
</style>
