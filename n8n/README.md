# XRE Franchise Formu → Telegram + SMS + E-posta (n8n)

Form: https://form.xre.com.tr/franchise/ — bir **Bitrix24 CRM formu** (Form ID 160).
Form gönderilince Bitrix, "Franchise" hattında (kategori 4) `Potansiyel Franchise`
aşamasında bir anlaşma + kişi açar. Bu aşamadaki **Webhook robotu** n8n'i çağırır.

`xre-franchise-form.json`, n8n'e **Workflows → Import from File** ile içe aktarılır.

## Akış

```
Bitrix Robot Webhook ─► Bitrix İsteğini Oku (DEAL_123 → 123)
  ─► Bitrix: Anlaşma + Kişi + Sorumlu (tek batch çağrısı)
  ─► Mesajları Hazırla
       ├─► Telegram: Gruba Bildir
       ├─► Telefon Var mı? ─► Ercell: Müşteriye SMS
       └─► E-posta Var mı? ─► Müşteriye E-posta
```

Akış Bitrix'te **kayıt oluşturmaz**, yalnızca okur. Telegram mesajında ad soyad, telefon,
e-posta, ülke, sorumlu, aşama ve anlaşma/kişi kartı linkleri bulunur.

Veri kaynakları: ad soyad ve telefon/e-posta önce **kişiden**, yoksa anlaşmadaki
`Telefon` (`UF_CRM_1734685306767`) ve `E-posta` (`UF_CRM_1726650433488`) alanlarından alınır.

## Kurulum

1. `Bitrix İsteğini Oku` node'undaki `AYAR` bloğunu doldurun (Bitrix gelen webhook, Ercell
   kullanıcı/şifre). Repo herkese açık olduğu için buradaki değerler yer tutucudur.
2. Telegram credential (`@xre_rapor_bot`) ve SMTP credential'ını seçin.
3. Workflow'u **Active** yapın; **Production URL**'yi Bitrix'te Franchise hattı →
   Potansiyel Franchise aşamasındaki **Webhook** robotuna girin.
4. Elle test: `POST <production-url>?deal_id=149548`

## Önemli

- **SMS için formda telefon alanı olmalı.** Bitrix formuna bir telefon alanı ekleyip
  *Kişi → Telefon* alanına eşleyin; aksi halde SMS adımı atlanır.
