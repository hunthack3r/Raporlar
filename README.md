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
