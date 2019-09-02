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

      // 店舗情報の店舗名、住所、緯度、経度を描画用の連想配列に整形
      shops.forEach(shop => {
        // 緯度経度の設定
        if ((typeof shop.lat == 'number') && (typeof shop.lng == 'number')) {
          // APIのレスポンスに緯度経度がある場合はそのまま設定
          locations.push(this.getFormattedLocation(shop.name, shop.lat, shop.lng, 'red'));
        } else {
          // APIレスポンスに緯度経度がないときは住所から緯度経度を取得して設定
          geocoder.geocode({ address: shop.address }, (results, status) => {
            if (status === google.maps.GeocoderStatus.OK) {
              locations.push(this.getFormattedLocation(shop.name, results[0].geometry.location.lat(), results[0].geometry.location.lng(), 'red'));
            } else {
              /* eslint-disable */
              console.log('error: geocode doesnt get latlng.');
            }
          });
        }
      });

      // 現在地設定後、マップにマーカー描画
      navigator.geolocation.getCurrentPosition(
        (position) => {    // 成功時処理
          // 現在地の緯度経度取得
          var currentLat = position.coords.latitude;
          var currentLng = position.coords.longitude;

          // 現在地を元にマップを初期化
          var currentLatLng = new google.maps.LatLng(currentLat, currentLng);
          var map = new google.maps.Map(
            this.$el, // マップを表示する要素
            {
              zoom: 15,                  // 拡大倍率
              center: currentLatLng,     // 緯度・経度
              gestureHandling: 'greedy', // 一本指でマップ移動
            }
          );

          // 現在地をマーカーに追加
          locations.push(this.getFormattedLocation('現在地', currentLat, currentLng, 'blue'));

          // マップにマーカーを立てる
          const markers = locations.map((location) => {
            const marker = new google.maps.Marker({ ...location, map });
            marker.addListener('click', () => {
              // クリックでズームする
              map.setZoom(16);
              map.setCenter(marker.getPosition());
            });
            return marker;
          });
        },
        (error) => {  // 失敗時処理
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
    } catch (error) {
      /* eslint-disable */
      console.error(error);
    }
  },
  methods: {
    getFormattedLocation: function (lacationName, lat, lng, iconColor = 'red') {
      // アイコンの選択 red or bule
      var iconUrl = (iconColor == 'blue') ? 'images/blue-dot.png' : 'images/red-dot.png';

      // フォーマット
      var location = {
        animation: google.maps.Animation.DROP, // 上から落ちてくるアニメーション
        label: {
          text:       lacationName,
          color:      'black',
          fontWeight: 'bold',
          fontSize:   '12px',
        },
        icon: {
          url: iconUrl,
        },
        position: {
          lat: lat,
          lng: lng,
        }
      };
      return location;
    }
  }
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