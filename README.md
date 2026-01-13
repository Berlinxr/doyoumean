# Proje Ana Alanı
**Yazılım**

# Proje Tematik Alanı
**Algoritma Tasarımı ve Uygulamaları**

# Proje Adı (Başlığı)
**Kullanıcı Etkileşimine Dayalı, Güvenlik Odaklı Otonom  
Domain Doğrulama ve Yönlendirme**

---

## ÖZET

İnternet kullanıcılarının web adreslerini (URL) hatalı yazması, siber güvenlik dünyasında **Typosquatting** olarak bilinen ciddi bir riske yol açmaktadır. Mevcut domain eşleştirme sistemleri, ya statik listelerle çalıştıkları için güncel tehditlere adapte olamamakta ya da sadece sayaç temelli çalıştıkları için öğrenme sürecinde zararlı (phishing) siteleri dâhil etme riski taşımaktadır.

Bu proje, bu eksiklikleri gidermek amacıyla **otonom öğrenme yeteneğine sahip web tabanlı bir uygulama** sunmaktadır. Sistem, kullanıcı girdilerini **Levenshtein Mesafesi temelli benzerlik algoritması (fuzzywuzzy)** ile hızlıca tarar. Eğer yerel listelerde eşleşme bulunamazsa, dış kaynak (**Google API**) üzerinden öğrenme sürecini başlatır.

Projenin temel özgünlüğü, **Kural Tabanlı Puanlama Algoritması** kullanmasıdır. Bu algoritma, domainin güvenlik katsayısı (TLD, Subdomain) ve benzerlik oranı gibi çoklu parametreleri değerlendirir. Böylece kötü niyetli domainlerin sisteme dâhil olması engellenir.

Flask ile geliştirilen bu algoritma, performanstan ödün vermeden güvenliği önceliklendiren, sürekli gelişen bir çözüm sunmaktadır.

**Anahtar Kelimeler:** Platform, Bağımsızlık, Kullanıcı Etkileşimli Gelişim, Sürdürülebilirlik

---

## AMAÇ

1. **Güvenlik Odaklı Karar Mekanizması Geliştirmek**  
   Geleneksel mantıktan kaçınarak, öğrenme listesindeki domainleri **Kural Tabanlı Puanlama Algoritması** ile değerlendirmektir. Bu algoritma, bir domaini sadece popülaritesine göre değil; aynı zamanda **TLD uzantısı** ve **subdomain yapısı** gibi güvenlik kriterlerine göre filtreleyerek kötü niyetli (phishing) sitelerin ana listeye dâhil olma riskini minimize etmeyi amaçlar.

2. **Otonom ve Adaptif Öğrenmeyi Sağlamak**  
   Projenin statik bir veritabanına bağlı kalmamasını sağlamaktır. Kullanıcıların arama etkileşimleri sayesinde sistem, kendi veritabanını (`domains.json`) sürekli güncelleyerek değişen internet ortamına ve yeni çıkan domainlere karşı adaptasyon yeteneği kazanmayı hedefler.

3. **Optimum Performans ve Erişim Sunmak**  
   Dış kaynak (API) kullanımını yalnızca zorunlu hallerde tutarak sistem maliyetini düşürmek ve Flask tabanlı web mimarisi sayesinde projenin tüm cihazlardan erişilebilir, platform bağımsız bir servis haline gelmesini sağlamaktır.

---

## 1. GİRİŞ

İnternet güvenliği alanında yapılan güncel araştırmalar, alan adı benzerliklerine dayalı **Typosquatting (Klavye sürçmesi)** saldırılarının kullanıcılar ve kurumlar için en büyük siber tehditlerden biri olmaya devam ettiğini göstermektedir. Truong (2023) tarafından yapılan kapsamlı analizde, mevcut tarayıcı tabanlı önlemlerin statik kara listelere dayandığı ve dinamik olarak üretilen yeni saldırı türlerine karşı yetersiz kaldığı vurgulanmıştır.

Literatürde bu sorunu çözmek için çeşitli metin benzerlik algoritmaları geliştirilmiştir. Spaulding ve Upadhyaya (2016), **Levenshtein Mesafesi** gibi yaklaşık metin eşleştirme yöntemlerinin typosquatting tespitinde temel standart olduğunu belirtmiştir. Ancak bu yöntemler çoğunlukla yalnızca kelime benzerliğine odaklanmakta, alan adının güvenilirliğini (TLD, SSL durumu vb.) göz ardı etmektedir.

Son dönemde Welch (2025), tespit sistemlerinde öğrenme tabanlı modellerin (LLM ve makine öğrenmesi) kullanımının önemini vurgulamıştır. Ancak literatürde, hem Levenshtein benzerliğini hem de **alan adı uzantısı güvenilirliğini** puanlayan ve **kullanıcı etkileşimiyle kendi kendini güncelleyen hafif mimarili sistemler** konusunda bir boşluk bulunmaktadır. Bu proje, söz konusu boşluğu doldurmayı amaçlamaktadır.

---

## 2. YÖNTEM

Bu projede, hatalı alan adlarını tespit edip doğrularını öneren web tabanlı bir prototip geliştirmek için **Deneysel Yazılım Geliştirme Metodolojisi** kullanılmıştır.

### 2.1. Veri Toplama ve Kullanılan Araçlar

Proje, **Python** programlama dili ve **Flask** web çatısı üzerinde inşa edilmiştir. Veri yönetimi için statik veritabanları yerine dinamik olarak güncellenen **JSON tabanlı yapı** (`domains.json`, `domains1.json`) tercih edilmiştir.

Yerel veritabanında bulunmayan sorgular için **Google Custom Search API** kullanılarak gerçek zamanlı öğrenme süreci desteklenmiştir.

Geliştirilen sistem, tarafımca yazılan Flask uygulaması üzerinde çalışmakta olup aşağıdaki adres üzerinden erişilebilmektedir:

```
https://system.onurepbaygun.icu
```

---

### 2.2. Geliştirilen Algoritmalar

#### 2.2.1. Benzerlik Tespiti (Levenshtein Mesafesi)

Kullanıcı girdisi ile veritabanındaki kayıtlar arasındaki yapısal benzerlik, **fuzzywuzzy** kütüphanesi kullanılarak ölçülmüştür.

- Ana Liste Eşiği: **%50**
- Öğrenme Listesi Eşiği: **%40**

#### 2.2.2. Kural Tabanlı Puanlama Algoritması

Aday domainlerin güvenli listeye alınıp alınmayacağına karar vermek için çok kriterli bir puanlama fonksiyonu (`hesapla_puan`) geliştirilmiştir.

**Puanlama Kriterleri:**

- **TLD Puanı**
  - `.com`, `.org`, `.net` → +1
  - `.edu`, `.gov` → +2
- **Subdomain Kontrolü**
  - `www` dışındaki şüpheli alt alan adları → −1
- **Kullanıcı Teyidi**
  - Domainin **5 farklı kullanıcı** tarafından sorgulanması → tetikleyici

#### 2.2.3. Karar Süreci

- Önce yerel önbellek kontrol edilir  
- Eşleşme yoksa API’den veri çekilir ve **Aday Liste**ye eklenir  
- Tekrar sayısı **5** olan domainler puanlanır  
- **Toplam puan ≥ 3** → Güvenli  
- **Toplam puan < 3** → Sistemden temizlenir  

---

## 4. BULGULAR

### 4.1. Benzerlik Eşiği Doğruluk Analizi

Belirlenen **%50 Ana Liste Eşiği**, 486 yazım hatası senaryosu ile test edilmiştir.

| Platform   | Deneme | Başarılı | Başarı (%) |
|-----------|--------|----------|------------|
| Instagram | 100    | 100      | 100.00     |
| LinkedIn  | 86     | 86       | 100.00     |
| TikTok    | 100    | 99       | 99.00      |
| YouTube   | 100    | 99       | 99.00      |
| Facebook  | 100    | 98       | 98.00      |

---

### 4.2. Otonom Öğrenme ve API Bağımlılığı

Sistem, bir domain için **6. sorgudan itibaren** harici API çağrılarını tamamen sonlandırmış ve API bağımlılığı **%0** seviyesine düşmüştür. Bu durum, akıllı önbellekleme yaklaşımının başarıyla uygulandığını göstermektedir.

---

## 5. SONUÇ VE TARTIŞMA

Bu proje, typosquatting saldırılarını yüksek doğrulukla tespit eden ve kullanıcı etkileşimiyle kendini güncelleyen **Otonom Öğrenen Domain Doğrulama Sistemi** geliştirilmesini kapsamaktadır.

- **%98+ doğruluk**
- **Harici API bağımlılığının kaldırılması**
- **Hafif mimari ile yüksek güvenlik**

Literatürde önerilen yaklaşımlarla uyumlu ve düşük kaynak tüketimli bir alternatif sunulmuştur.

---

## 6. ÖNERİLER

### 6.1. Tarayıcı Eklentisi

Sistemin bir **Browser Extension** olarak geliştirilmesiyle, kullanıcı siteye girmeden önce phishing saldırıları engellenebilir.

### 6.2. Derin Öğrenme Entegrasyonu

Gelecek çalışmalarda **LSTM** veya **BERT** tabanlı modellerle anlamsal domain sahteciliği tespiti yapılabilir.