# XRE Franchise Form → Bitrix24 + Telegram + SMS + E-posta (n8n)

`xre-franchise-form.json`, n8n'e **Workflows → Import from File** ile içe aktarılır.

## Akış

```
Form Webhook ─► Form Verisini Hazırla ─► Bitrix: Mükerrer Kontrol (telefon)
     ├─ lead varsa ───────────────────────┐
     └─ yoksa ─► Bitrix: Lead Oluştur ────┴─► Bitrix: Lead Getir ─► Bitrix: Sorumlu Getir ─► Mesajları Hazırla
                                                                                             ├─► Telegram: Gruba Bildir
                                                                                             ├─► Ercell: Müşteriye SMS
                                                                                             └─► (e-posta varsa) Müşteriye E-posta
```

Telegram grubuna giden mesajda ad soyad, telefon, e-posta, şehir, formdaki diğer tüm alanlar,
sorumlu kişi, lead durumu ve Bitrix kartı linki bulunur. Aynı telefonla ikinci başvuru gelirse
yeni lead açılmaz, mevcut kart "(mükerrer)" etiketiyle bildirilir.

## Kurulum

1. **Ayarlar**: `Form Verisini Hazırla` node'unun en üstündeki `AYAR` bloğunu doldurun
   (Bitrix gelen webhook URL'si — `crm` ve `user` izinli, portal adresi, SMS metni).
2. **Telegram**: n8n → Credentials → *Telegram API* → bot token'ını (`@xre_rapor_bot`) girin,
   adını `XRE Rapor Bot` koyun. Token'ı JSON'a / repoya yazmayın.
   - Chat ID `-5591560509` olarak ayarlı. Mesaj gitmezse grup "süper grup" olabilir;
     o zaman `-1005591560509` deneyin (doğru ID'yi görmek için gruba bir mesaj yazıp
     `https://api.telegram.org/bot<TOKEN>/getUpdates` açın).
3. **Ercell SMS**: `AYAR.ercell` içine XML API adresi, kullanıcı adı ve şifreyi girin.
   SMS, Ercell `CORPORATESMS` XML formatında (başlık `XRE`, `UC`, `1:N`) gönderilir;
   numara `905XXXXXXXXX` formatındadır.
   > Repo herkese açık olduğu için buradaki JSON'da Bitrix webhook kodu ve Ercell şifresi
   > yer tutucudur; gerçek değerler yalnızca n8n'deki workflow'da durmalı.
4. **E-posta**: *SMTP* credential'ı (`XRE SMTP`) oluşturun, `fromEmail`'i kontrol edin.
5. Workflow'u **Active** yapın ve **Production URL**'yi alın:
   `https://<n8n-adresiniz>/webhook/xre-franchise-form`
6. **Formu bağlayın** (form.xre.com.tr/franchise-form):
   - *Elementor Pro Form*: Actions After Submit → **Webhook** → URL'yi yapıştırın
     (Advanced Data açık olabilir, ikisi de desteklenir).
   - *Contact Form 7*: "CF7 to Webhook" eklentisi ile URL'yi girin.
   - *WPForms / Gravity / Fluent*: ilgili Webhooks eklentisi ile POST gönderin.

Form alan adları otomatik eşlenir (`ad`, `soyad`, `ad_soyad`, `name`, `telefon`, `phone`,
`email`, `e_posta`, `sehir`, `mesaj` vb.); tanınmayan alanlar Telegram'da ve Bitrix
yorumunda "ek alan" olarak gösterilir.

Telegram, SMS ve e-posta adımları birbirinden bağımsız: biri hata verirse diğerleri yine çalışır.
