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
Note that there are some constraints to the argument expression, as explained
in the "Dynamic Argument Expression Constraints" section below.
-->
<a v-bind:[attributeName]="url"> ... </a>
```

Here `attributeName` will be dynamically evaluated as a JavaScript expression, and its evaluated value will be used as the final value for the argument. For example, if your Vue instance has a data property, `attributeName`, whose value is `"href"`, then this binding will be equivalent to `v-bind:href`.

Similarly, you can use dynamic arguments to bind a handler to a dynamic event name:

``` html
<a v-on:[eventName]="doSomething"> ... </a>
```

In this example, when `eventName`'s value is `"focus"`, `v-on:[eventName]` will be equivalent to `v-on:focus`.

#### Dynamic Argument Value Constraints

Dynamic arguments are expected to evaluate to a string, with the exception of `null`. The special value `null` can be used to explicitly remove the binding. Any other non-string value will trigger a warning.

#### Dynamic Argument Expression Constraints

Dynamic argument expressions have some syntax constraints because certain characters, such as spaces and quotes, are invalid inside HTML attribute names. For example, the following is invalid:

``` html
<!-- This will trigger a compiler warning. -->
<a v-bind:['foo' + bar]="value"> ... </a>
```

The workaround is to either use expressions without spaces or quotes, or replace the complex expression with a computed property.

When using in-DOM templates (templates directly written in an HTML file), you should also avoid naming keys with uppercase characters, as browsers will coerce attribute names into lowercase:

``` html
<!--
This will be converted to v-bind:[someattr] in in-DOM templates.
Unless you have a "someattr" property in your instance, your code won't work.
-->
<a v-bind:[someAttr]="value"> ... </a>
```

### Modifiers

Modifiers are special postfixes denoted by a dot, which indicate that a directive should be bound in some special way. For example, the `.prevent` modifier tells the `v-on` directive to call `event.preventDefault()` on the triggered event:

``` html
<form v-on:submit.prevent="onSubmit"> ... </form>
```

You'll see other examples of modifiers later, [for `v-on`](events.html#Event-Modifiers) and [for `v-model`](forms.html#Modifiers), when we explore those features.

## Shorthands

The `v-` prefix serves as a visual cue for identifying Vue-specific attributes in your templates. This is useful when you are using Vue.js to apply dynamic behavior to some existing markup, but can feel verbose for some frequently used directives. At the same time, the need for the `v-` prefix becomes less important when you are building a [SPA](https://en.wikipedia.org/wiki/Single-page_application), where Vue manages every template. Therefore, Vue provides special shorthands for two of the most often used directives, `v-bind` and `v-on`:

### `v-bind` Shorthand

``` html
<!-- full syntax -->
<a v-bind:href="url"> ... </a>

<!-- shorthand -->
<a :href="url"> ... </a>

<!-- shorthand with dynamic argument (2.6.0+) -->
<a :[key]="url"> ... </a>
```

### `v-on` Shorthand

``` html
<!-- full syntax -->
<a v-on:click="doSomething"> ... </a>

<!-- shorthand -->
<a @click="doSomething"> ... </a>

<!-- shorthand with dynamic argument (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

They may look a bit different from normal HTML, but `:` and `@` are valid characters for attribute names and all Vue-supported browsers can parse it correctly. In addition, they do not appear in the final rendered markup. The shorthand syntax is totally optional, but you will likely appreciate it when you learn more about its usage later.
