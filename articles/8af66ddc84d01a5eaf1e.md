---
title: "cesiumのreact wrapperであるresiumでPLATEAUのデータを表示する"
emoji: "⛳"
type: "tech"
topics: ["react", "cesiumjs", "PLATEAU"]
published: true
---

こんにちは。Web エンジニアの Yuichiro Izumi です。

日本の都市の 3D 都市モデルの整備を推進するプロジェクト「PLATEAU」にて、
公開されている 3D 都市モデルが最近話題になっていました。

公式サイトはこちら
https://www.mlit.go.jp/plateau/

GitHub ページはこちら
https://github.com/Project-PLATEAU

今回はこのデータを React を利用して Web で表示するところまで実装してみたところ、
とてもお手軽にできたので、記事にまとめました。

# 利用するライブラリ

### react

### create-react-app

### cesiumjs

Web で 3D 都市モデルをビジュアライズするライブラリ
https://cesium.com/platform/cesiumjs/

### resium

Cesium.js を React で使えるライブラリ
https://resium.darwineducation.com/

### craco

create-react-app を利用した際の設定をいい感じに上書きできるライブラリ
craco は、「Create React App Configuration Override」の略称です。
https://github.com/gsoft-inc/craco

### craco-cesium

cesium/resium を利用するための設定をいい感じにしてくれる craco plugin
https://github.com/reearth/craco-cesium

### craco-plugin-react-hot-reload

hot reload を利用するための設定をいい感じにしてくれる craco plugin

# 実装手順

### create-react-app でプロジェクトを作成

```
create-react-app plateau-sample
```

### 必要な Package を追加

```
yarn add cesium @craco/craco craco-cesium craco-plugin-react-hot-reload resium
```

### package.json の修正

package.json の script を一部を`react-scripts`から`craco`に修正します。

```json:package.json
{
  // ...
  "scripts": {
    "start": "craco start", // react-scripts -> craco
    "build": "craco build", // react-scripts -> craco
    "test": "craco test",   // react-scripts -> craco
    "eject": "react-scripts eject"
  },
  // ...
}
```

### craco.config.js を追加

ルートディレクトリに craco.config.js を作成し、下記設定を記述します。
詳細は、https://github.com/reearth/craco-cesium を参照ください。

```js:craco.config.js
module.exports = {
  webpack: {
    alias: {
      "react-dom": "@hot-loader/react-dom",
    },
  },
  plugins: [
    { plugin: require("craco-plugin-react-hot-reload") },
    { plugin: require("craco-cesium")() },
  ],
};
```

### 3D 都市モデルの追加

G 空間情報センターの Web ページから 3D 都市モデルをダウンロードします。

1. G 空間情報センターの Web ページにアクセス
   https://www.geospatial.jp/ckan/dataset/plateau-tokyo23ku

2. 「3D Tiles / GeoPakage / JSON 形式」を選択
   https://www.geospatial.jp/ckan/dataset/plateau-tokyo23ku-3dtiles-2020

3. お好みの区のファイルを一覧からダウンロード
   :::message
   数 10~数 100MB くらいあるので、通信環境が良いところでダウンロードしてください。
   テクスチャ無しの方がデータ量が小さいため、検証としてはお手軽です。
   :::

4. zip ファイルを解凍
   適当なツールで zip ファイルを解凍してください。

5. ファイルの移動
   解凍したファイルをディレクトリごと/public に移動してください。

### App.tsx の編集

```ts: App.tsx
import React from "react";
import { hot } from "react-hot-loader/root";

import Cesium, { Cartesian3 } from "cesium";
import {
  Viewer,
  Entity,
  PointGraphics,
  EntityDescription,
  Cesium3DTileset,
} from "resium";

function App() {
  let viewer: Cesium.Viewer; // This will be raw Cesium's Viewer object.
  const handleReady = (tileset: Cesium.Cesium3DTileset) => {
    if (viewer) {
      viewer.zoomTo(tileset);
    }
  };
  return (
    <Viewer
      full
      ref={(e) => {
        if (e?.cesiumElement) {
          viewer = e.cesiumElement;
        }
      }}
    >
      <Cesium3DTileset
        url="http://localhost:3000/13102_chuo-ku/tileset.json" // ダウンロードした地図データのパス
        onReady={handleReady}
      />
    </Viewer>
  );
}

export default hot(App);
```

### いざ、プレビュー

```sh
yarn start
```

https://localhost:3000 を開いてみてください

地図上に都市 3D モデルが表示されているはずです！

![](https://storage.googleapis.com/zenn-user-upload/62d11ef5b0374303ca09f53f.png)

移動操作や拡大縮小操作が可能です。
また、オブジェクトをクリックすると情報が表示されます。
![](https://storage.googleapis.com/zenn-user-upload/2a65fd37e5deb3a4a35a6643.png)

実際にプレビューしてみて、読み込みまでの時間やブラウザの重さが気になりました。
3D で表示するエリアを狭めて読み込むデータを小さくしたり、工夫が必要な部分がありそうです。

# 最後に

わずかなコード記述で 3D 都市データの描画ができました。
ライブラリもとても充実しており、色々なことができそうと感じました。

面白いアイディアが浮かべば、3D 都市データを利用したクールなサービス作りたいなと思いました。
