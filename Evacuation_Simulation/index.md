---
layout: default
marp: false
title: トップページ
---

# Blender導入手順

## インストール

<https://www.blender.org/download/releases/2-83/>

上記リンクからBlender2.83LTSをインストール
最新バージョンは2.9だが互換性を保つためにこのバージョンで

## Blenderの日本語化

## コンソールの出し方、コンソールの日本語化

## 提供したBlenderファイルを開く

## スクリプトを実行する

スクリプトは提供したBlenderファイルに内蔵されている
GitHubにオリジナルがあり、更新するときはこうやってください

### スクリプト｜Instance_Character

レイアウト選択でScript
生成する数を指定し、Instance_Character実行
コンソールで進行状況を確認
指定回数生成できたことを確認

### スクリプト｜Import_CSV

csvを開き、"Cube"を"Character"に置換、ShiftJSで保存
Import_CSVで先のcsvファイルのパスを指定して実行
下の方にキーフレームが現れれば成功
Spaceキーでアニメーションを開始する

## Tips

### 視点操作方法、平行投影方法

### アニメーションを最初から開始する

### 上のタブの意味

### スクリプトの更新方法

**layout/theme.liquid**の<head>タグ内にFontリンクを貼る

**assets/theme.scss.liquid** のプロパティ

- \$font-stack-header
- \$font-stack-body
- \$font-stack-cart-notification

でフォント値を**Noto Sans JP**と**sans-serif**にする
sans-serifのところ表示不可を回避するためのセーフフォントであれば何でも良い

# スライドバーの幅を伸ばす

CSSのGrid Layoutを使ってページ幅を分割し、レイアウトしている
この比率を変えてやれば幅を伸ばせる

**theme.scss.liquid**の1292行目～でクラスが定義されてる

```scss
  .#{$grid-breakpoint-type}one-third { width: percentage(1 / 3); }
  .#{$grid-breakpoint-type}two-thirds { width: percentage(2 / 3); }
```

<!-- ![a](https://imgur.com/2C90VMy.png) -->
テーマ設定→色

&darr;

<!-- ![a](https://i.imgur.com/PoTqnwH.png) -->
フォームフィールドの背景

**フォームの枠線**は、

#### assets/theme.scss.liquid 2639-2650line

```css
.form-vertical {
  input,
  select,
  textarea {
    display: block;
    margin-bottom: $padding-form;
    min-width: 300px;

    @include media-query($small) {
      width: 100%;
    }
  }
```

`min-width:300px;`の下の行に`border: 1px solid #000;`を挿入すれば枠線が表示される
枠線のスタイルを変更したいときは[こちら](https://techacademy.jp/magazine/8626)を参照

# 住所入力フォーム(テキスト入力タイプ)にグレー文字の初期値を設定する

htmlのinput要素にplaceholder属性を追加すると初期値を設定できる [こちらを参照](https://weback.net/htmlcss/1284/)
**注意:** placeholderで初期値を設定できるのはテキスト入力の項目のみで、都道府県項目のように選択タイプだと設定できない
選択タイプの設定方法は次に説明する

#### templates/customers/address.liquid 19-89line

```html
<div id="AddressNewForm" class="form-address form-vertical hide">


</div>
```

の所を以下のコードに置き換えれば設定できる

`selectInitialProvince('Kyōto');`で引数に京都を設定し、実行する  
`Kyoto`ではなく`Kyōto`なことに注意  

# 住所入力フィールドのラベルを変更する

#### locales/ja.json 153-154line

```json
  "address1": "住所",
  "address2": "住所",
```

を

```json
  "address1": "地名\/番地",
  "address2": "建物名\/部屋番号",
```

に変更する  
`"地名\/番地",`の`\`はjsonファイルにおける[/(スラッシュ)文字のエスケープ処理](https://www.ipentec.com/document/json-character-escape)
