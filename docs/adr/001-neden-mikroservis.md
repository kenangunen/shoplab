# ADR-001: Neden mikroservis?

| | |
|---|---|
| **Durum** | Kabul edildi |
| **Tarih** | 2026-11-27 |
| **Task** | T-002 |
| **İlgili kararlar** | K1 (7 servis), K10 (monorepo, Maven multi-module) |

## Bağlam

ShopLab tek bir mağazanın e-ticaret backend'i. Tek bir geliştirici yazıyor, gerçek kullanıcısı ve gerçek trafiği yok. Projenin asıl amacı çalışan bir ürün değil, **öğrenmek**: servisler arası iletişim, olay tabanlı mimari, dağıtık transaction (saga, outbox), servis bazında veri sahipliği, gözlemlenebilirlik ve Kubernetes.

Bu ölçekte mikroservis seçmek gerçek hayatta tartışmalıdır. Çoğu deneyimli ekip aynı problemi bir monolit ya da modüler monolitle çözer. Bu ADR'ın amacı bu tartışmayı dürüstçe kayda geçirmek: neyi bilerek seçtiğimizi ve bunun bedelini.

## Sorulan sorular

1. Projenin başarı ölçütü ne: ürünün kendisi mi, yoksa kurulurken öğrenilenler mi?
2. Öğrenmek istediğimiz konuların kaçı tek bir süreç içinde **gerçekten** ortaya çıkar?
3. Bir ekip ya da bağımsız deploy ihtiyacı var mı?
4. Mikroservisin operasyonel yükünü (çok süreç, ağ hataları, dağıtık izleme) tek kişi taşıyabilir mi?
5. Yanlış seçtiysek geri dönmek ne kadar pahalı?

## Seçenekler

### (a) Monolit

Tek uygulama, tek veritabanı, paketlerle ayrılmış katmanlar.

- **Artı:** En hızlı başlangıç. Tek transaction ile tutarlılık bedava, debug kolay, tek deploy.
- **Eksi:** Modül sınırlarını koruyan bir mekanizma yok, sınırlar zamanla bulanıklaşır. Öğrenme hedeflerinin çoğu (saga, outbox, servisler arası hata yönetimi, Kafka) **hiç ortaya çıkmaz**, çünkü problem hiç doğmaz.

### (b) Modüler monolit

Tek deploy edilebilir uygulama, ama içi net sınırlı modüllerden oluşur. Her modül kendi tablolarına sahiptir, modüller birbirine sadece public API ve olaylarla konuşur. Spring Modulith bu sınırları test ile doğrulayabilir.

- **Artı:** Gerçek hayatta bu ölçek için en doğru seçim. Servis sınırlarını ve veri sahipliğini öğretir, dağıtık sistemin bedelini ödetmez. Sınırlar doğru çizildiyse ileride mikroservise bölmek kolaydır.
- **Eksi:** Ağ hatası, kısmi başarısızlık, eventual consistency ve servisler arası saga gerçek anlamda yaşanmaz. Kubernetes'te tek pod olur, servis bazında ölçekleme ve dayanıklılık konuları öğrenilmez.

### (c) Mikroservis

Her iş yeteneği ayrı bir süreç, ayrı bir veritabanı. Servisler HTTP ve Kafka ile konuşur.

- **Artı:** Öğrenme hedeflerinin tamamı **zorunlu olarak** ortaya çıkar. Sipariş akışı tek transaction'a sığmadığı için saga yazmak zorunda kalırız. Servis düşebildiği için timeout, retry ve circuit breaker gerekir. Veri ayrı olduğu için olay tabanlı senkronizasyon gerekir.
- **Eksi:** Aşağıdaki "Bedeller" bölümündeki her şey.

## Karar

**(c) Mikroservis.** Başlangıçta 7 servis (K1), tek repoda Maven multi-module olarak (K10).

Gerekçe tek cümle: **Bu projenin ürünü yazılım değil, öğrenmedir.** Öğrenmek istediğimiz problemler ancak sistem gerçekten dağıtık olduğunda ortaya çıkar. Modüler monolit bu problemleri "nasıl olurdu" düzeyinde anlatır, mikroservis ise yaşatır.

Bilerek aşırı mühendislik yapıyoruz. Bunu bir hata olarak değil, bir eğitim aracı olarak seçiyoruz ve bedelini aşağıda açıkça kabul ediyoruz.

## Gerçek bir şirket olsaydık ne seçerdik?

**(b) Modüler monolit.** Tek mağaza, tek geliştirici ve kanıtlanmamış bir ürün için mikroservis, var olmayan problemlere bugünden ödenen bir faturadır. Gerçek bir ekip:

- Modüler monolitle başlar, sınırları netleştirir.
- Bir modülü ancak **somut bir sebep** doğduğunda ayırır: o modül diğerlerinden farklı ölçeklenmesi gerekiyorsa, ayrı bir ekip ona sahip olacaksa, farklı bir deploy ritmine ihtiyacı varsa ya da farklı bir teknoloji gerektiriyorsa.

Bu projede de servis sınırlarını, sanki ileride bir modüler monolite geri birleştirilebilecekmiş gibi temiz tutmaya çalışacağız.

## Sonuçlar

### Kabul ettiğimiz bedeller

1. **Dağıtık tutarlılık:** Sipariş, stok ve ödeme tek bir transaction'da değil. Tutarlılığı saga ve outbox ile kendimiz sağlamak zorundayız. Monolitte bu tek bir `@Transactional`'dı.
2. **Ağ artık bir hata kaynağı:** Her servis çağrısı zaman aşımına uğrayabilir, yavaşlayabilir ya da yarıda kesilebilir. Timeout, retry, idempotency ve circuit breaker gerekiyor.
3. **Yerel ortam ağırlaşıyor:** Faz 7'de 7 servis, 2 veritabanı türü, Kafka, Keycloak, Redis ve Elasticsearch aynı anda çalışacak. 8 GB RAM'de zorlanılır, profil bazında kısmi ayağa kaldırma gerekecek.
4. **Debug zorlaşıyor:** Tek bir isteğin hikâyesi birden fazla servisin loguna dağılıyor. Correlation id ve dağıtık izleme (tracing) olmadan hata bulmak çok zor.
5. **Test zorlaşıyor:** Unit ve slice testler yetmiyor. Servisler arası kontrat testleri ve Testcontainers ile entegrasyon testleri gerekiyor. Uçtan uca test kurmak pahalı.
6. **Veri kopyalanıyor:** Servisler birbirinin veritabanına erişemediği için bazı veriler (ürün adı, fiyat, adres) olaylarla kopyalanıyor ve bir süre tutarsız kalabiliyor (eventual consistency).
7. **Tekrar eden altyapı kodu:** Her serviste aynı yapılandırma, hata formatı ve güvenlik ayarı. Paylaşılan domain kütüphanesi kullanmama kararı (K10) bunu bilerek kabul ediyor.
8. **Yavaş ilerleme:** Bir özelliği eklemek, monolite göre daha fazla dosya, daha fazla servis ve daha fazla koordinasyon demek.

### Kazandıklarımız

- Projenin öğrenme hedeflerinin tamamı gerçek problemler olarak karşımıza çıkacak.
- Servis başına farklı mimari (layered ve hexagonal, K8) ve farklı veritabanı (K6) denenebilecek.
- Kubernetes'te servis bazında deploy, ölçekleme ve dayanıklılık gerçekten anlamlı olacak.

## Bu kararı ne zaman yeniden düşünmeliyiz?

- **Projenin amacı değişirse:** ShopLab gerçek kullanıcıya açılacak bir ürüne dönüşürse, modüler monolite geri birleştirmeyi ciddi bir seçenek olarak değerlendir.
- **Bedeller öğrenmeyi engellemeye başlarsa:** Zamanın çoğu öğrenme yerine yerel ortamı ayakta tutmaya gidiyorsa, sık birlikte değişen iki servisi birleştirmeyi düşün.
- **İki servis sürekli birlikte değişiyorsa:** Bir özellik için hep aynı iki servise dokunuluyorsa, sınır yanlış çizilmiş olabilir. Bu, birleştirme ya da sınırı yeniden çizme sinyalidir.
- **Faz 5 sonunda (sipariş akışı tamamlandığında):** Bu ADR'ı yeniden oku. "Mikroservis bana ne öğretti, hangi bedeli gereksiz ödedim?" sorularını cevapla ve bir ek not düş.
