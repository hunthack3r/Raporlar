# Raporlar 
```
https://github.com/reddelexc/hackerone-reports
```

### SSTI Notlarim

Bir hata yakalamaya çalışırken @inflection programında test yapıyordum ve bu sırada goodhire.com'u incelemeye başladım. 
Bu site test kapsamındaydı, ancak HubSpot CMS kullandığını fark etmedim. Uzun süre test yaptıktan sonra, 
potansiyel bir Server-Side Template Injection (SSTI) zafiyeti buldum. Bu süreçte @fransrosen'in yardımıyla,
SSTI'yi parçalayıp Reflected XSS zafiyetine dönüştürmeyi başardık.

Google Dork sorgusu şu şekildeydi: 
```
inurl:/_hcms/
```

Ayrıntılı PoC (Proof of Concept):
Zafiyeti /hcms/cta yolunda buldum, bu da bu yolun HubSpot hizmeti tarafından kontrol edildiğini gösteriyordu.
Etkilenen parametre ?referrerUrl= idi.
İlk olarak, Server-Side Template Injection (SSTI) denemesi yaptım:
Kullanıcı tarafından kontrol edilen girdi sunucu tarafında bir şablona gömüldüğünde SSTI meydana gelir, bu da kötü niyetli şablon yönergelerinin enjekte edilmesine olanak tanır. Bu, saldırganın sunucuda rastgele kod çalıştırmasına neden olabilir.
URL kodlaması yapılmış GET girdisi referrerUrl, {{7*7}} olarak ayarlandı.
Yanıtta, bu ifadenin sonucu olan 49 değeri döndü.
Jinja Injection denemesi yaptım, ancak şu hatayı aldım: Malformed escape pair at index 78 veya Illegal character in query at index 81.
Reflected XSS:
@fransrosen, aşağıdaki payload ile elementi parçalayarak XSS zafiyetini ortaya çıkardı:
plaintext
```
{%25+macro+field(x)+%25}www.com{{x}}+<b>ok</b>{%25+endmacro+%25}{{+field(1)%7curlize+}}
```
Örnek PoC:
plaintext
```
https://www.example.com/_hcms/cta?referrerUrl={%25+macro+field(x)+%25}www.com{{x}}+<b>ok</b>{%25+endmacro+%25}{{+field(1)%7curlize+}}
```
XSS Payload oldukça etkileyiciydi:
```
{%25+macro+field()+%25}moc.okok//:ptth//)niamod.tnemucod(trela:tpircsavaj=daolno+gvshttp://http:""//{%25+endmacro+%25}{{+field(1)%7curlize%7creverse%7curlize%7creverse%7curlize%7creverse+}}
```
Sonuçlar:
Rapor Durumu: 22 Ocak 2018'de HubSpot Güvenlik ekibi önceliği P2'ye değiştirdi ve raporu çözülmüş olarak işaretledi.
23 Ocak 2018'de, rapor kapatıldı ve sorun çözüldü.
11 Eylül 2018'de raporun şiddeti yüksek olarak güncellendi.
Bu süreçte HubSpot ekibinin hızlı tepki vermesi ve zafiyeti kısa sürede çözmesi büyük bir başarıydı. Ancak, BugCrowd ekibinin ilk gönderimi "Kapsam Dışı" olarak kapatması hayal kırıklığı yarattı.
___________________________________
# SSTI 2 

Glovoapp'teki zafiyetleri araştırırken ilginç bir keşifte bulundum. 
SSTI (Server-Side Template Injection) zafiyetini test ederken, 
Glovoapp'te (https://www.glovoapp.com/kg/en/bishkek/) bu zafiyetin olabileceğini fark ettim.
Aşağıda bulgularımı ve adımları detaylı olarak anlatıyorum.

Zafiyeti Ortaya Çıkarma Adımları:
Adım 1: İlk olarak, Glovoapp adresine giderek "Kayıt Ol" (Register) butonuna tıkladım. 
Yeni bir hesap oluşturmak için kayıt sayfasına yönlendirildim.

Adım 2: Bu aşamada,``` Ad (First Name)``` alanına ```{{7*7}}``` gibi basit bir şablon motoru ifadesi girdim. 
Bu, şablon motorunun bu girdiyi nasıl işlediğini görmek için kullanabileceğim temel bir SSTI testiydi.

Adım 3: Ardından, kayıt sayfasındaki diğer bilgileri doldurdum ve hesabımı başarıyla oluşturdum. 
Amacım, verdiğim girdinin arka planda nasıl işleneceğini görmekti.

Adım 4: Kayıt işlemi tamamlandıktan sonra, girdim olan ```{{7*7}}``` ifadesinin arka planda sunucu tarafında değerlendirildiğini doğrulamak için beklemeye başladım.

Adım 5: Bir süre sonra, hoş geldiniz/promosyon e-postasının gelen kutuma ulaşmasını bekledim. 
Ve işte, e-posta geldi! E-postanın konu başlığında ```"49, Glovo'ya hoş geldiniz!"``` yazıyordu. 
Bu, şablon motorunun girdiğim {{7*7}} ifadesini işlediğini ve sonucunu bana gönderdiğini kanıtlıyordu.

Adım 6: Bu noktada, saldırganın bu zafiyeti daha fazla kötüye kullanabileceğini fark ettim. 
Örneğin, Ad (First Name) alanına daha karmaşık ve kötü niyetli yükler enjekte ederek uygulamadan hassas bilgiler toplayabilir veya sunucu üzerinde komut çalıştırabilirdi.

Not: Bu saldırıyı gerçekleştirdikten sonra, diğer hesabım için hoş geldiniz e-postası almadım. 
Büyük olasılıkla, enjekte edilen kod nedeniyle sistemde bir hata oluşmuş olabilir.
Etki ve Sonuç:
Şablon motorları, web uygulamaları tarafından dinamik içerikleri sunmak için yaygın olarak kullanılıyor. 
Ancak, bu tür güvenlik açıkları, sunucunun iç yapısına doğrudan saldırı düzenlenmesine ve hatta Uzaktan Kod Yürütme``` (RCE) ```gibi ciddi sonuçlara yol açabilir. 
Bu zafiyet, her savunmasız uygulamayı potansiyel bir saldırı noktası haline getirir.

Bu zafiyeti bulduğumda, Glovoapp ekibiyle hemen iletişime geçtim ve sorunun çözülmesi için gereken adımları attılar. 
Ancak, bu zafiyetin ne kadar tehlikeli olabileceğini göstermek için buradaki adımları ve bulguları paylaşmak istedim.
_______________________________________
# SSTI 3

Lodash.js kütüphanesinin _.template fonksiyonunda bir Server-Side Template Injection (SSTI) zafiyeti tespit ettim. Bu zafiyet, sunucuda kod çalıştırılmasına izin veriyor.

Modül Bilgileri:

Modül Adı: Lodash
Sürüm: 4.17.15
npm Sayfası: Lodash npm
Modül Açıklaması:
Lodash kütüphanesi, Node.js modülleri olarak dışa aktarılmıştır. 
Bu modül, web uygulamalarında sıkça kullanılır ve haftada 26 milyondan fazla kez indirilir.

Zafiyetin Detayları:
Zafiyet Açıklaması:
Lodash kütüphanesinin _.template fonksiyonu, kullanıcı tarafından sağlanan girdiyi düzgün bir şekilde doğrulamaz. 
Bu fonksiyon, bir uygulamanın kullanıcıdan gelen girdiyi şablona yerleştirmesi durumunda saldırganlar tarafından suistimal edilebilir. 
Saldırgan, ${JSON.stringify(process.env)} gibi bir payload kullanarak sunucu üzerinde JavaScript kodu çalıştırabilir.

Zafiyeti Tekrar Üretme Adımları:
Adım 1: Lodash.js kütüphanesini gerektiren basit bir test uygulaması oluşturun. 
Aşağıdaki uygulama, 'name' parametresinde kullanıcıdan gelen girdiyi kabul eder ve bu girdi Lodash'ın _.template fonksiyonu tarafından işlenir.

```javascript
const express = require('express');
const _ = require('lodash');
const escapeHTML = require('escape-html');
const app = express();
app.get('/', (req, res) => {
  res.set('Content-Type', 'text/html');
  const name = req.query.name
  // Kullanıcı girdisinden bir şablon oluşturun
  const compiled = _.template("Hello " + escapeHTML(name) + ".");
  res.status(200).send(compiled());
});

app.listen(8000, () => {
  console.log('POC app listening on port 8000!')
});
```
Adım 2: Savunmasız uygulamayı şu URL ile ziyaret edin:

plaintext
Kodu kopyala
http://127.0.0.1:8000/?name=Test
Adım 3: Savunmasız uygulamayı tekrar ziyaret edin ve ${JSON.stringify(process.env)} gibi bir payload'ı 'name' parametresine girin. Örneğin:

plaintext
Kodu kopyala
http://127.0.0.1:8000/?name=Test${JSON.stringify(process.env)}
Bu payload, sunucuda çalıştırıldığında ortam değişkenlerini (environment variables) döndürecektir.

Destekleyici Materyal/Referanslar:
OS: OSX 10.15.5
Node.js Sürümü: v10.16.0
NPM Sürümü: v6.9.0
Sonuç:
Bu zafiyet, uzaktan kod yürütme (Remote Code Execution - RCE) riski taşıyor ve bu nedenle oldukça ciddi. 
Zafiyeti tespit ettikten sonra, kütüphane yöneticisine ulaşmadım ve ilgili depoya bir sorun bildirimi açmadım.

Ek Yorumlar:
Lodash belgelerini ve yöneticinin yorumlarını inceledikten sonra, _.template fonksiyonunun ${} ifadelerini ES (ECMAScript) desteği için şablon ayırıcılar olarak yorumladığını fark ettim. 
Bu durum, girdi içindeki ${} kalıbını kontrol eden bir regex ile eşleştiriliyor. Eşleşme gerçekleştiğinde, girdi gömülü bir ifade olarak değerlendirilir ve bu da bir saldırıya neden olabilir. 
Örneğin, ${JSON.stringify(process.env)} değeri _.template fonksiyonuna verildiğinde, sunucu ortam değişkenlerinin döndürülmesine yol açar veya name=Fred${7*7} değeri girildiğinde sonuç Fred49 olur.

Bu durumun aslında Lodash'ın beklenen işleyişi olduğunu anladım, ancak yine de bu işlevin güvensiz girdilerle kullanılmaması gerektiğini düşünüyorum.

Sonuç olarak, bu zafiyetin ciddi bir güvenlik açığı olduğunu doğrulayan bir CVE (CVE-2021-23337) daha sonra yayınlandı. 
Bu raporun geçerli olduğunu düşündüğüm için raporun açıklanmasını istedim ve kütüphane yöneticisi de bu raporu açıkladı. Rapor, uzaktan kod yürütme (RCE) zafiyeti nedeniyle önemlidir.

__________________________________________________________
# XXE 

Raporun Özeti:
@johnstone olarak, Starbucks'ın Çin'deki iş ilanları sayfasını (https://ecjobs.starbucks.com.cn) ziyaret ettiğimde, 
bir XML External Entities (XXE) zafiyeti tespit ettim. Bu zafiyet, 
bir saldırganın sunucuda zararlı dosyalar yüklemesine ve çeşitli saldırılar gerçekleştirmesine olanak tanıyordu.

Zafiyeti Bulma Süreci:
Adım 1: Öncelikle, kullanıcı fotoğrafı yükleme özelliğini incelemeye başladım. Burada, 
güvenlik kısıtlamalarını aşarak HTML, XHTML, XML, config gibi dosyaları yükleyebileceğimi fark ettim.

Adım 2: Burp Suite kullanarak, yükleme isteğini yakaladım ve allow_file_type_list parametresine html; 
gibi bir değer ekledim. Ayrıca, dosya adının sonunu .jpg yerine .html olarak değiştirdim. Bu şekilde, 
sunucunun yüklediğim dosyayı kabul etmesini sağladım. Yüklediğim dosyaya şu URL'den erişebildim:

```
https://ecjobs.starbucks.com.cn/retail/tempfiles/temp_uploaded_641dee35-5a62-478e-90d7-f5558a78c60e.html
```
Adım 3: Ardından, zararlı bir XML dosyası oluşturdum ve _hxpage parametresi ile bu dosyayı sunucuya yükledim. Örneğin:

```
POST /retail/hxpublic_v6/hxdynamicpage6.aspx?_hxpage=tempfiles/temp_uploaded_d4e4c8c5-c4ab-4743-a6fd-c2d779a29734.xml&max_file_size_kb=1024&allow_file_type_list=xml;jpg;jpeg;png;bmp;
```
Ya da HX_PAGE_NAME parametresini değiştirdim:

```
POST /retail/hxpublic_v6/hxxmlservice6.aspx HTTP/1.1
HX_PAGE_NAME="tempfiles/temp_uploaded_71cc275c-64fc-40fc-a9cc-52cce5a02858.xml"
```
Bu istekler sonucunda, Starbucks'ın sunucusunun saldırganın sunucusundan DTD dosyasını çektiğini doğruladım.


Bu zafiyet, saldırganın sunucuda zararlı dosyalar yüklemesine ve kullanıcıların bilgilerini çalmasına olanak tanıyordu. 
Ayrıca, XXE zafiyeti, sunucunun bazı bilgilerini açığa çıkartabilir, hizmet reddi saldırılarına yol açabilir ve hatta NTLMv2 hash saldırılarına neden olabilir. 
Starbucks'ın sunucu ortamı IIS 7.5 + ASP.NET + Windows olduğundan, bu zafiyet sunucunun tamamen ele geçirilmesine yol açabilir.

Eğer bu rapor geçerli sayılmayacaksa, lütfen raporu kendim kapatabilmem için izin verin. Teşekkür ederim.


24 Şubat 2019: Raporum Starbucks'a sundum.
26 Şubat 2019: Zafiyetin ciddiyeti "orta" seviyeden "yüksek" seviyeye çıkarıldı, ardından "kritik" olarak güncellendi.
27 Şubat 2019: Starbucks ekibi raporumu inceledi ve doğruladı. Rapor "Triaged" (işlemde) durumuna alındı.
3 Mart 2019: Starbucks ekibine zafiyetin hala var olup olmadığını sordum ve raporun ödüle uygun olup olmadığını öğrenmek istedim.
7 Mart 2019: Yeni bir zafiyet keşfettim ve ekibe bildirdim. Bu sefer, dosya türü kısıtlamalarını atlayarak sunucuya ASPX dosyaları yükleyebiliyordum. 
Bu, önceki XXE zafiyetinden bile daha ciddi bir sorundu, çünkü saldırgan sunucuda zararlı komutlar çalıştırabilirdi.
20 Mart 2019: Starbucks ekibi, raporumun başarıyla çözüldüğünü onayladı ve ödül verileceğini bildirdi.


Bu raporun sonunda Starbucks, raporumu doğruladı ve gerekli düzeltmeleri yaptı. Bu süreç boyunca ekiple iyi bir iletişim kurdum ve sonunda ödüllendirildim. 
Ayrıca, bu raporun açıklanmasını talep ettim ve bu talebim kabul edildi. Bu benim için şanslı bir gün oldu!


__________________________________________________________

