# drs-maps

## Description

DANCERUSH STARDOMの設置店舗を表示するアプリ

## Image

<img width="402" alt="20190813" src="https://user-images.githubusercontent.com/17407690/62876746-a092dc00-bd60-11e9-80e0-3360276d12f9.png">

## Project setup
```
npm install
```
```
touch .env
echo VUE_APP_GOOGLE_MAPS_API_KEY={{GOOGLE MAPSのAPIキー}} > .env
echo VUE_APP_GET_SHOPS_API={{店舗APIエンドポイントURL}} > .env
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Run your tests
```
npm run test
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
