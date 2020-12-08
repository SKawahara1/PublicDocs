---
layout: default
title: トップページ
---

# Google Fonts (Noto Sans JP)を使用する方法

参照: <https://note.com/gooddoctor/n/nce84b2ef484d>

参照リンクを見ればだいたいわかるはず、一応詳細を以下に記述

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

**one-third**で3分割の1、**one-thirds**で3分割の2つ分という感じで
ページ幅からの分割数と比率を決めている

liquidファイルのクラス名これを入れればレイアウトできる

サイドバー幅を変更する場合、**layout/theme.liquid**, **sections/sidebar.liquid**
の2つのリキッドファイルにまたがって設定されているのを変更する

##### layout/theme.liquid: 66line~

```html
  <main class="main-content grid__item medium-up--four-fifths" id="MainContent" role="main">
```

&darr;

```html
  <main class="main-content grid__item medium-up--three-quarters" id="MainContent" role="main">
```

#### sections/sidebar.liquid: 2line~

```html
 <nav class="grid__item small--text-center medium-up--one-fifth" role="navigation">
```

&darr;

```html
 <nav class="grid__item small--text-center medium-up--one-quarter" role="navigation">
  ```

サイドバーが1/5,メインコンテンツが4/5のところを1/4, 3/4に変更することで、
ページ幅に占めるサイドバーの比率を上げている

### 注意

one-quarterのように左側の数字が1だと右は**quarter**と単数形だが
three-quartersのように左側が複数だと**quarters**と複数形になる
間違えやすいので注意

# リンクにマウスが乗った時に色を変える

### assets/theme.scss.liquid: 573line~

```css
 a:hover {
    background-color: rgb(162, 171, 255);
    outline: 0;
}
```

aタグ（リンクのセレクタ）:hoverでマウスが乗った時の設定ができる

# 背景色をグラデーションにする

linear-gradientで設定できる
ここでは3色でグラデーションを表している

### assets/theme.scss.liquid: 1503line~

```css
  body,
    html {
      /* background-color: $color-body; */
      background:
      linear-gradient(rgba(200,227,255,0.2),
                      rgba(246,250,255,0.2) 60%,
                      rgba(255,255,255,0.2));
    }
```

# ログインしたとき名前でなく姓が出るようにする

該当箇所のfirst_nameをlast_nameに変更

##### locales/ja.json: 263line~

```json
   "customer": {
      "account": "アカウント",
      "logged_in_as_html": "{{ last_name }}としてログインしました",
```

##### sections/header.liquid

```html
  {% if customer.last_name != blank %}
      {% capture last_name %}<a href="{{ routes.account_url }}">{{ customer.last_name }}</a>{% endcapture %}
      {{ 'layout.customer.logged_in_as_html' | t: last_name: last_name }}
```

# 住所入力フォームの順番を入れ替える

「新しい住所を追加する」場合、liquidファイルの該当箇所は

#### templates/customers/addresses.liquid: 19~89line

```html
<div id="AddressNewForm" class="form-address form-vertical hide">
  {% form 'customer_address', customer.new_address %}
    <div class="grid"></div>
    <div class="grid"></div>
    .
    .
</div>
```

というコード構成になっている
<class="grid">内の<class="grid-item">にフォーム項目が入っているので
その順番を入れ替える

もともとは英語圏向けの並びになっているので、日本向けに
[姓] [名] [郵便番号] [国] [都道府県] [市町村] [番地] [建物/部屋番号] [電話番号] [会社名]
と並び替える

仕様では「国」は日本で確定のため削除ということだが、「国」フォームをコードから消すとエラーになるので

#### templates/customers/addresses.liquid

 ```html
  <div class="grid__item hide">
    <label for="AddressCountryNew">{{ 'customer.addresses.country' | t }}</label>
```

のように親要素のクラス名に hide を入れ、見た目上消すことで対処する

「国」フォームをコードから消すとエラーになる理由はShopifyの住所テンプレートの仕様から外れるからだと思われる

参照：[Shopify Developers - customers/addresses.liquid](https://shopify.dev/docs/themes/files/customers-addresses-liquid)

ユーザーのフォーム入力が空でも送信できるが、
このリンク先の表にあるform typeとname attributeが一致しないと送信できない

郵便番号入力フォームを2つに分割できないのもこれが原因

## 住所入力フォームの順番を入れ替える-　コピー用コード

templates/customers/addresses.liquidの19～89行目を消し、下のコードをコピーして挿入すれば日本用の項目順になる

```html
<div id="AddressNewForm" class="form-address form-vertical hide">

  {% form 'customer_address', customer.new_address %}

    <div class="grid">

      <div class="grid__item">
        <label for="AddressLastNameNew">{{ 'customer.addresses.last_name' | t }}</label>
        <input type="text" id="AddressLastNameNew" name="address[last_name]" value="{{ customer.last_name }}" autocapitalize="words" readonly>
      </div>

      <div class="grid__item">
        <label for="AddressFirstNameNew">{{ 'customer.addresses.first_name' | t }}</label>
        <input type="text" id="AddressFirstNameNew" name="address[first_name]" value="{{ customer.first_name }}" autocapitalize="words" readonly>
      </div>

    </div>

    <div class="grid">

      <div class="grid__item">
        <label for="AddressZipNew">{{ 'customer.addresses.zip' | t }}</label>
        <script src="https://ajaxzip3.github.io/ajaxzip3.js" charset="UTF-8"></script>
        <input type="text" id="AddressZipNew" name="address[zip]" value="{{ form.zip }}" autocapitalize="characters" maxlength="8" onKeyUp="AjaxZip3.zip2addr(this,'','address[province]','address[city]');">
      </div>
      <div class="grid__item hide">
        <label for="AddressCountryNew">{{ 'customer.addresses.country' | t }}</label>
        <select id="AddressCountryNew" name="address[country]" data-default="{{ form.country }}">{{ all_country_option_tags }}</select>
        {% comment %} <input type="text" id="AddressCountryNew" name="address[country]" value="{{ form.country }}" autocapitalize="words"> {% endcomment %}
      </div>

      <div class="grid__item" id="AddressProvinceContainerNew">
        <label for="AddressProvinceNew">{{ 'customer.addresses.province' | t }}</label>
        <select id="AddressProvinceNew" name="address[province]" data-default="{{ form.province }}"></select>
        {% comment %} <input type="text" id="AddressProvinceNew" name="address[province]" value="{{ form.province }}" autocapitalize="words"> {% endcomment %}
      </div>
      <div class="grid__item">
        <label for="AddressCityNew">{{ 'customer.addresses.city' | t }}</label>
        <input type="text" id="AddressCityNew" name="address[city]" value="{{ form.city }}" autocapitalize="words">
      </div>

      <div class="grid__item">

        <label for="AddressAddress1New">{{ 'customer.addresses.address1' | t }}</label>
        <input type="text" id="AddressAddress1New" name="address[address1]" value="{{ form.address1 }}" autocapitalize="words">

        <label for="AddressAddress2New">{{ 'customer.addresses.address2' | t }}</label>
        <input type="text" id="AddressAddress2New" name="address[address2]" value="{{ form.address2 }}" autocapitalize="words">
      </div>

    </div>

    <div class="grid">
      <div class="grid__item">
        <label for="AddressPhoneNew">{{ 'customer.addresses.phone' | t }}</label>
        <input type="tel" id="AddressPhoneNew" name="address[phone]" value="{{ form.phone }}">

        <label for="AddressCompanyNew">{{ 'customer.addresses.company' | t }}</label>
        <input type="text" id="AddressCompanyNew" name="address[company]" value="{{ form.company }}" autocapitalize="words">
      </div>

    </div>

    <p>
      {{ form.set_as_default_checkbox }}
      <label for="address_default_address_new">{{ 'customer.addresses.set_default' | t }}</label>
    </p>

    <p><input type="submit" class="btn" value="{{ 'customer.addresses.add' | t }}"></p>
    <p><button type="button" class="btn--link address-new-toggle">{{ 'customer.addresses.cancel' | t }}</button></p>

  {% endform %}
</div>

```

# フォームの特定項目にフォーカスする

#### assets/theme.js.liquid: 254-257ine

```javascript
  // Toggle new/edit address forms
  $('.address-new-toggle').on('click', function() {
    $newAddressForm.toggleClass('hide');
  });
```

theme.js.liquidの254-257行目の上記コードを以下のように変更する

```javascript
  const focusMethod = (id)=> {
    document.getElementById(id).focus();
  }
  // Toggle new/edit address forms
  $('.address-new-toggle').on('click', function() {
    $newAddressForm.toggleClass('hide');
    focusMethod('AddressZipNew');
  });
```

`focusMethod('AddressZipNew');`の`'AddressZipNew'`を変更することでフォーカス先を変えられる<br>

例):

- `'AddressZipNew'` → `'AddressLastNameNew'`　で「姓」フォーム
- `'AddressZipNew'` → `'AddressFirstNameNew'`　で「名前」フォーム

document.getElementById().focus()でフォームの項目にフォーカスできる
`focusMethod(id)`の引数idはフォーカス先のidを入れる
住所作成ボタン(.address-new-toggle)をクリックした時に実行される

# フォームの項目を事前に入力し、変更不可にする

formのinputタグで設定する

#### templates/customers/addresses.liquid: 27line

```html
 <input type="text"
        id="AddressLastNameNew" name="address[last_name]"
        value="{{ customer.last_name }}"
        autocapitalize="words"
        readonly>
```

valueでデフォルト値を設定できるので、顧客情報データから{{ customer.last_name }}のようにして持ってくる
readonly属性を入れると変更不可になる

# 郵便番号から住所を自動入力する

ajaxzip3ライブラリを使って自動入力する
参照：<https://github.com/ajaxzip3/ajaxzip3.github.io>
-<https://haniwaman.com/ajaxzip3/>

#### templates/customers/address.liquid 40-42line

```html
<label for="AddressZipNew">{{ 'customer.addresses.zip' | t }}</label>
<script src="https://ajaxzip3.github.io/ajaxzip3.js" charset="UTF-8"></script>

 <input type="text"
        ...
        onKeyUp="AjaxZip3.zip2addr(this,'','address[province]','address[city]');">
```

onKeyUp(キーボードを離した時)、郵便番号が該当すれば都道府県(province)と市町村(city)に自動入力される
郵便番号は8桁まで入力可能で、ハイフンあり/なしのどちらでも自動入力される

# 入力フォームのスタイルを変更する

入力フォームのセル色はテーマのカスタマイズから変更できる

![a](https://imgur.com/2C90VMy.png)
テーマ設定→色

&darr;

![a](https://i.imgur.com/PoTqnwH.png)
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

  {% form 'customer_address', customer.new_address %}
      ...(略)
  {% endform %}
</div>
```

の所を以下のコードに置き換えれば設定できる

```html
<div id="AddressNewForm" class="form-address form-vertical hide">

  {% form 'customer_address', customer.new_address %}

    <div class="grid">

      <div class="grid__item">
        <label for="AddressLastNameNew">{{ 'customer.addresses.last_name' | t }}</label>
        <input type="text" id="AddressLastNameNew" name="address[last_name]" value="{{ customer.last_name }}" autocapitalize="words" readonly>
      </div>

      <div class="grid__item">
        <label for="AddressFirstNameNew">{{ 'customer.addresses.first_name' | t }}</label>
        <input type="text" id="AddressFirstNameNew" name="address[first_name]" value="{{ customer.first_name }}" autocapitalize="words" readonly>
      </div>

    </div>

    <div class="grid">

      <div class="grid__item">
        <label for="AddressZipNew">{{ 'customer.addresses.zip' | t }}</label>
        <script src="https://ajaxzip3.github.io/ajaxzip3.js" charset="UTF-8"></script>
        <input type="text" id="AddressZipNew" name="address[zip]" value="{{ form.zip }}" autocapitalize="characters" maxlength="8" placeholder="128443"
        onKeyUp="AjaxZip3.zip2addr(this,'','address[province]','address[city]');">
      </div>
      <div class="grid__item hide">
        <label for="AddressCountryNew">{{ 'customer.addresses.country' | t }}</label>
        <select id="AddressCountryNew" name="address[country]" data-default="{{ form.country }}">{{ all_country_option_tags }}</select>
        {% comment %} <input type="text" id="AddressCountryNew" name="address[country]" value="{{ form.country }}" autocapitalize="words"> {% endcomment %}
      </div>

      <div class="grid__item" id="AddressProvinceContainerNew">
        <label for="AddressProvinceNew">{{ 'customer.addresses.province' | t }}</label>
        <select id="AddressProvinceNew" name="address[province]" data-default="{{ form.province }}"></select>
        {% comment %} <input type="text" id="AddressProvinceNew" name="address[province]" value="{{ form.province }}" autocapitalize="words"> {% endcomment %}
      </div>
      <div class="grid__item">
        <label for="AddressCityNew">{{ 'customer.addresses.city' | t }}</label>
        <input type="text" id="AddressCityNew" name="address[city]" value="{{ form.city }}" autocapitalize="words" placeholder="京都市伏見区竹田藁屋町">
      </div>

      <div class="grid__item">

        <label for="AddressAddress1New">{{ 'customer.addresses.address1' | t }}</label>
        <input type="text" id="AddressAddress1New" name="address[address1]" value="{{ form.address1 }}" autocapitalize="words" placeholder="50">

        <label for="AddressAddress2New">{{ 'customer.addresses.address2' | t }}</label>
        <input type="text" id="AddressAddress2New" name="address[address2]" value="{{ form.address2 }}" autocapitalize="words" placeholder="増田医科器械ビル 101">
      </div>

    </div>

    <div class="grid">
      <div class="grid__item">
        <label for="AddressPhoneNew">{{ 'customer.addresses.phone' | t }}</label>
        <input type="tel" id="AddressPhoneNew" name="address[phone]" value="{{ form.phone }}">

        <label for="AddressCompanyNew">{{ 'customer.addresses.company' | t }}</label>
        <input type="text" id="AddressCompanyNew" name="address[company]" value="{{ form.company }}" autocapitalize="words">
      </div>

    </div>

    <p>
      {{ form.set_as_default_checkbox }}
      <label for="address_default_address_new">{{ 'customer.addresses.set_default' | t }}</label>
    </p>

    <p><input type="submit" class="btn" value="{{ 'customer.addresses.add' | t }}"></p>
    <p><button type="button" class="btn--link address-new-toggle">{{ 'customer.addresses.cancel' | t }}</button></p>

  {% endform %}
</div>
```

# 住所入力フォーム(選択タイプ)に初期値を設定する

#### assets/theme.js.liquid231-240

```javascript
  // Initialize observers on address selectors, defined in shopify_common.js
  if (Shopify) {
    new Shopify.CountryProvinceSelector(
      'AddressCountryNew',
      'AddressProvinceNew',
      {
        hideElement: 'AddressProvinceContainerNew'
      }
    );
  }
```

の箇所を以下のコードに変更する

```javascript
  const selectInitialProvince = (provinceName) =>{
    const provinceOptions =  document.getElementById('AddressProvinceNew').options;
    const provinceArray = [].slice.call(provinceOptions);
    const indexNum = provinceArray.findIndex(option => option.value === provinceName);
    provinceOptions[indexNum].selected = true;
  }
  // Initialize observers on address selectors, defined in shopify_common.js
  if (Shopify) {
    new Shopify.CountryProvinceSelector(
      'AddressCountryNew',
      'AddressProvinceNew',
      {
        hideElement: 'AddressProvinceContainerNew'
      }
    );
    selectInitialProvince('Kyōto');
  }
```

`const provinceOptions =  document.getElementById('AddressProvinceNew').options;`
で都道府県の[select 要素に含まれる option 要素をすべて取得する](https://developer.mozilla.org/ja/docs/Web/API/HTMLSelectElement/options)

`const provinceArray = [].slice.call(provinceOptions);`
取得したoption要素は配列ではないので、[配列に変換する](https://lab.syncer.jp/Web/JavaScript/Snippet/53/)  

`const indexNum = provinceArray.findIndex(option => option.value === provinceName);`  
配列から引数`provinceName`の[index番号を取得する](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)  

`provinceOptions[indexNum].selected = true;`  
取得したindex番号から初期値に設定したいoption要素を導き、[初期値として設定する](https://techacademy.jp/magazine/24290)  

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
