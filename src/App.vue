<template>
  <div class="App"/>
</template>

<script>
import gmapsInit from './utils/gmaps';

export default {
  name: 'App',
  async mounted() {
    const google   = await gmapsInit();
    const geocoder = new google.maps.Geocoder();
    const map      = new google.maps.Map(this.$el);

    var locations = []; // 現在地
    var shops     = []; // 店舗情報

    try {
      // 店舗情報を取得
      await this.$axios.get(process.env.VUE_APP_GET_SHOPS_API)
        .then(
          response => shops = response.data.data
        )
        .catch(
          /* eslint-disable */
          error => console.log(error)
        );
      // 住所から緯度経度を取得
      shops.forEach(shop => {
        console.log(shop.address);
        geocoder.geocode({ address: shop.address }, (results, status) => {
          if (status === google.maps.GeocoderStatus.OK) {
            locations.push({
              position: {
                lat: results[0].geometry.location.lat(),
                lng: results[0].geometry.location.lng(),
              }
            });
          } else {
            /* eslint-disable */
            console.log('error: geocode doesnt get latlng.');
          }
        });
      });

      // 現在地の取得
      navigator.geolocation.getCurrentPosition(
        (position) => {    // 成功時処理
          locations.push({ // 現在地設定
            position: {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            }
          });
        },
        (error) => {  // 失敗時処理
          /* eslint-disable */
          console.log('error');
          switch (error.code) {
            case error.PERMISSION_DENIED:
              alert('位置情報の提供を許可してください');
              break;
            case error.POSITION_UNAVAILABLE:
              alert('位置情報の取得に失敗しました');
              break;
            case error.TIMEOUT:
              alert('位置情報の取得がタイムアウトしました');
              break;
            case error.UNKNOWN_ERROR:
              alert('不明なエラーです');
          }
        },
        { // options
          timeout: 100000,          // タイムアウト(ミリ秒)
          enableHighAccuracy: true, // 精度の高い位置情報を取得
        }
      );

      // マップにマーカー描画
      geocoder.geocode({ address: 'Tokyo' }, (results, status) => {
        if (status !== 'OK' || !results[0]) {
          throw new Error(status);
        }

        map.setCenter(results[0].geometry.location);
        map.fitBounds(results[0].geometry.viewport);

        // マーカークリックでズームする
        const markerClickHandler = (marker) => {
          map.setZoom(16);
          map.setCenter(marker.getPosition());
        };

        // 現在地にマーカーを立てる
        const markers = locations.map((location) => {
          const marker = new google.maps.Marker({ ...location, map });
          marker.addListener('click', () => markerClickHandler(marker));
          return marker;
        });
      });
    } catch (error) {
      /* eslint-disable */
      console.error(error);
    }
  },
};

</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

.App {
  width: 100vw;
  height: 100vh;
}
</style>