<template>
  <div class="App"/>
</template>

<script>
import gmapsInit from './utils/gmaps';

export default {
  name: 'App',
  async mounted() {
    var locations = []; // 現在地
    try {
      // 現在地の取得
      navigator.geolocation.getCurrentPosition(
        (position) => {   // 成功時処理
          locations = [{  // 現在地設定
            position: {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            },
          }];
        },
        (error) => {  // 失敗時処理
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
          timeout: 10000,           // タイムアウト(ミリ秒)
          enableHighAccuracy: true, // 精度の高い位置情報を取得
        }
      );

      const google = await gmapsInit();
      const geocoder = new google.maps.Geocoder();
      const map = new google.maps.Map(this.$el);

      geocoder.geocode({ address: 'Tokyo' }, (results, status) => {
        if (status !== 'OK' || !results[0]) {
          throw new Error(status);
        }

        map.setCenter(results[0].geometry.location);
        map.fitBounds(results[0].geometry.viewport);

        // 現在地にマーカーを立てる
        const markers = locations.map(x => new google.maps.Marker({ ...x, map }));
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