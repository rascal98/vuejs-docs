---
title: Template Syntax
type: guide
order: 4
---
Vue.js Html tabanlı bir şablon sözdizimi kullanır bu size DOM'a Vue örneğinin altında yatan veriyi bildirimsel bağlayıp ekrana oluşturabilmeyi sağlar. Tüm Vue.js şablonları özel tarayıcılar ve html çözücüleri ile çözülebilir.

Arkaplanda Vue şablonları Sanal DOM render fonksiyonlarında derler. Vue az sayıda komponenti tekrar basmak ve uygulama durumu değiştiğinde az miktardaki dom manüpilasyonlarını şablonlarda derler.

Eğer sanal DOM konseptlerine ve JavaScriptin raw(htmli ekrana javascript ile yazma) gücüne aşinaysanız şablon yerine isteğe bağlı JSX desteğiyle [direkt render fonksiyonlarını yaz](render-function.html).

## Interpolations

### Text

"Mustache" sözdizimi (çift süslü parantez) veri bağlamanın en temel yöntemidir:

``` html
<span>Message: {{ msg }}</span>
```

Mustache etiketi karşılık gelen `msg` özelliğinin değeri ile değiştirilir. Bu  `msg` özelliği her ne zaman değişirse güncellenecektir.

Ayrıca bir kereliğine ekleme yapmak ve veri değiştiriğinde güncellenme yapmak istemiyorsanız [v-once directive](../api/#v-once) direktifini kullanabilirsiniz. Fakat aklınızda olsun aynı düğümdeki diğer veri bağlama kısımlarına da etki edecektir:

``` html
<span v-once>Bu asla değişmeyecek: {{ msg }}</span>
```

### Raw HTML

Double mustaches veriyi düz yazı gibi yorumlar, Html gibi değil. Çıktıyı html olarak almak isterseniz [`v-html` direktifi](../api/#v-html)'ni kullanmanız gerekecektir:

``` html
<p>Mustaches kullanarak: {{ rawHtml }}</p>
<p>v-html direktifiyle: <span v-html="rawHtml"></span></p>
```

{% raw %}
<div id="example1" class="demo">
  <p>mustaches kullanarak: {{ rawHtml }}</p>
  <p>v-html direktifiyle: <span v-html="rawHtml"></span></p>
</div>
<script>
new Vue({
  el: '#example1',
  data: function () {
    return {
      rawHtml: '<span style="color: red">Bu kırmızı olmalı.</span>'
    }
  }
})
</script>
{% endraw %}


`span`'ın içeriği `rawHtml` özelliğinin değeriyle değiştirilecektir  ve veri bağlama işlemleri göz önünde bulundurulmaksızın düz HTML olarak yorumlanacaktır. `v-html`'i parça şablon oluşturmak için kullanmayın çünkü Vue bir yazı tabanlı şablon frameworkü değildir. Bunun yerine, komponentler UI (kullanıcı arayüzü-user interface) tekrar kullanılabilirlik ve düzenleme için esas birim olarak tercih edilir.

<p class="tip">Dinamik HTML i keyfi olarak websitenizde kullanırsanız siteniz kolayca [XSS saldırılarına](https://en.wikipedia.org/wiki/Cross-site_scripting) maruz kalabilir bu yüzden çok tehlikelidir. Html gösterme işlemini güvenilir içeriklerde kullanın ve **asla** kullanıcı tarafından sağlanan içeriklerde kullanmayın.</p>

### Attributes

Mustaches HTML nitelikleri içerisinde kullanılamaz. Bunun yerine a [`v-bind` direktifi](../api/#v-bind) kullanılır:
Mustaches cannot

``` html
<div v-bind:id="dynamicId"></div>
```

Bu durumda bool nitelikler, kendi bataklığında mevcudiyeti adı üstünde `true` olan, `v-bind` biraz farklı çalışır. Bu örnekle göreceğiz.
In the case of boolean attributes, where their mere existence implies `true`, `v-bind` works a little differently. In this example:

``` html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

Eğer `isButtonDisabled` bu değerlere sahipse `null`,`undefined` veya `false` , kapalı niteliği bile `<button>` elementinde oluşturulamayacaktır.

### JavaScript İfadelerini Kullanmak

Basit bir özellik anahtarını şablonlara bağlayalı uzun zaman oldu. Fakat Vue.js Javascript kendi içinde tüm veri bağlama işlemlerinin tam gücünü fiilen destekler:

``` html
{{ number + 1 }}

{{ ok ? 'EVET' : 'HAYIR' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

Bu ifadeler Javascript gibi içerisinde kendi Vue örneğinin kapsamında gibi değerlendirilir. Bu herbir bağlama sadece **bir ifade** içerebilir bir sınırlamadır, bu sayede bunu takip eden ifadeler **çalışmayacaktır**:

``` html
<!-- bu bir beyandır, ifade değildir: -->
{{ var a = 1 }}

<!-- flow control won't work either, use ternary expressions -->
{{ if (ok) { return message } }}
```

<p class="tip">Şablon ifadeleri korumalıdır ve sadece erişime sahip olan bir [whitelist of globals](https://github.com/vuejs/vue/blob/v2.6.10/src/core/instance/proxy.js#L9) dir `Math` ve `Date` gibi. Kullanıcı tarafından tanımlanan globallere erişime şablon ifadelerinde girişimde bulunmamalısınız.</p>

## Direktifler

Direktifler özel niteliklerdir `v-` ön ekiyle başlar. Direktif nitelik değerleri **yanlız bir javascript ifadesi** (`v-for` istisna, bunu daha sonra bakacağız) olması beklenir.
Bir direktifin işi DOM'a değer ve ifade değişikliklerinde reaktif olarak yan etkilerini uygulamasıdır.Hadi bir örnekle göz gezdirelim, dökümana girişte görmüştük:

``` html
<p v-if="seen">Beni görüyorsun</p>
```

Burada, `v-if` direktifi, `seen` ifadesinin değerinin doğruluğuna bağlı olarak `<p>` elementini ekler/siler.

### Argümanlar

Bazı direktifler "argüman" alabilir, direktif adından sonra bir kolon tarafından belirtilir. Örneğin ,`v-bind` direktifi kullanıldığında bir HTML niteliğini reaktif olarak günceller:

``` html
<a v-bind:href="url"> ... </a>
```

Burada `href` bir argümandır, `v-bind` direktifine elementin `href` niteliğinin değerini `url` ifadesine bağladığını söyler.

Bir başka örnek ise `v-on` direktifidir. Bu DOM etkinliklerini dinler:

``` html
<a v-on:click="doSomething"> ... </a>
```

Burada argüman bir dinlenecek event işleminin ismidir. Daha sonra event yakalama hakkında birçok detay hakkında konuşacağız.

### Dinamik Argümanlar

> 2.6.0+

2.6.0 versiyonuyla başlayan, ayrıca Javascript ifadelerini direktif argümanlarında köşeli parantezle sarmalayarak kullanmamızı mümkün kılar:

``` html
<!--
Dikkate alın burada argüman ifadelerine bazı kısıtlamalar var,"Dinamik Argüman İfade Kısıtlamaları"'nın aşağıdaki bölümde açıklandığı gibi 

-->
<a v-bind:[attributeName]="url"> ... </a>
```
Burada `attributeName` dinamik olarak JavaScript ifadesi olarak değişecektir ve bu değişen değer argümanın son değeri olarak kullanılacaktır. Örneğin, eğer Vue örneğinizin bir veri özelliği varsa ve `attributeName`'i `href` olan ile bu bağlama `v-bind:href` ile eşdeğer olacaktır.

Benzer olarak, dinamik argümanları dinamik eventlerde de bağlamak için ismiyle kullanabilirsiniz:

``` html
<a v-on:[eventName]="doSomething"> ... </a>
```

Bu örnekte `eventName`'nin değeri `"focus"` olduğunda `v-on:focus` ile eşdeğer olacaktır.


#### Dinamik Argüman Değer Kısıtlamaları

Dinamik argümanlar yazılan yazıyı değerlendirmesi içindir, `null` ile beraber. Özel değer olan `null` bağı açıkça silmek için kullanılır. Diğer her string olmayan değerler bir uyarıyı tetikleyecektir.

#### Dinamik Argüman Değer Kısıtlamaları

Dinamik argüman ifadeleri bazı belirli karakter söz dizimi kısıtlamalarına sahiptir boşluklar ve tırnaklar gibi, bunlar HTML nitelik isimlerinin içinde geçersizdir. Örneğin aşağıdaki geçersizdir:

``` html
<!-- Bu bir derleyici uyarısı tetikler. -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

Geçici çözüm her ikisine de ifadeleri boşluk ve tırnak olmadan veya bir karmaşık ifadeyle hesaplanan özellik olarak değiştirip kullanabilirsiniz. 

DOM içi şablonları kullanırsanız (şablonlar direkt olarak html dosyasında yazılmış halde), ayrıca isimlendirme anahtarlarını büyük karakterleri önlemelisiniz, tarayıcılar nitelik isimlerini küçük karakter olması için zorlayacaktır:

``` html
<!--
  Bu DOM şablonu içerisinde buna dönüştürülecektir v-bind:[someattr].
  "someattr" özelliği örneğiniz içerisinde olmadıkça kodunuz çalışacaktır.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

### Değiştiriciler

Değiştiriceler, bir direktifin belirli bir şekilde bağlanması gerektiğini belirten, nokta ile gösterilen özel son düzeltmedir. Örneğin, `.prevent` değiştiricisi `v-on` direktifinin eventi tetiklendiğinde  `event.preventDefault()` fonksiyonunu çağırır.


``` html
<form v-on:submit.prevent="onSubmit"> ... </form>
```

Diğer değiştiricilerin örneklerini [for `v-on`](events.html#Event-Modifiers) ve [for `v-model`](forms.html#Modifiers) bunların özelliklerini keşfederken sonra göreceğiz.


## Kısaltmalar (Shorthands)

`v-` ön eki Vue'nin niteliklerinin kimliğini belirlemeyi  şablonlarda görsel bir ipucu olarak bize sunar. Dinamik davranışları kabul ettiğinizde bazı hali hazırda olan işaretlemeler Vue.js kullandığınızda kullanışlıdır , fakat sık sık kullanılan direktifleri gereksiz sözlerle dolu hissedeceksiniz. Aynı zamanda, bir [SPA] inşa ettiğinizde (https://en.wikipedia.org/wiki/Single-page_application) `v-` ön eki daha az önemli hale gelir. Bu yüzden, iki veya daha sık kullanılan direktifler  `v-bind` ve `v-on` için Vue özel kısaltmalar sağlar.


### `v-bind` Kısaltması

``` html
<!-- Tam söz dizimi -->
<a v-bind:href="url"> ... </a>

<!-- kısaltma -->
<a :href="url"> ... </a>

<!-- kısaltma ve dinamik argüman (2.6.0+) -->
<a :[key]="url"> ... </a>
```

### `v-on` Shorthand

``` html
<!-- Tam söz dizimi -->
<a v-on:click="doSomething"> ... </a>

<!-- kısaltma -->
<a @click="doSomething"> ... </a>

<!-- kısaltma ve dinamik argüman (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

Bunlar normal HTML'den biraz farklı görünmektedir fakat `:` ile `@` nitelikler için geçerli karakterlerdir ve tüm Vue-destekli tarayıcılarda doğru şekilde gösterilecektir. İlave olarak, son oluşturma işaretlemesinde görünmeyecektir. Kısaltma sözdizimi tamamen seçenekseldir fakat daha fazla öğrenip kullandığınızda muhtemelen takdir edeceksiniz.

