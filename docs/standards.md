# Proje standartları

Bu kurallar projenin ilk gününden itibaren geçerlidir. Bir kural değişirse önce bu dosya güncellenir.

## Board

GitHub Projects üzerinde tek bir board: **ShopLab**. Her task bir issue'dur ve başlığı task id'si ile başlar (`T-101 customer-service: müşteri kaydı ve profil`). Her faz bir milestone'dur.

| Kolon | Anlamı |
|---|---|
| Backlog | Henüz sırası gelmemiş task'lar |
| Ready | Kabul kriterleri net, başlanabilir |
| In Progress | Üzerinde çalışılıyor (aynı anda en fazla 1) |
| Review | PR açık, review bekliyor |
| Done | Definition of Done'ı karşılıyor ve `main`'e girdi |

## Branch

- `main` her zaman derlenir ve testleri geçer. `main`'e doğrudan commit atılmaz.
- Her task için kısa ömürlü bir branch açılır. `main`'e PR ile girer.
- Branch adı **task id'si ve kısa bir İngilizce açıklamadan** oluşur, kebab-case yazılır:
  - `T-001-repo-skeleton`
  - `T-101-customer-registration`

## Commit

[Conventional Commits](https://www.conventionalcommits.org/) kullanılır. Mesajlar **İngilizce** yazılır.

```
<type>(<scope>): <kısa açıklama>
```

| Tip | Ne zaman? |
|---|---|
| `feat` | Yeni davranış |
| `fix` | Hata düzeltme |
| `refactor` | Davranışı değiştirmeyen yapı değişikliği |
| `test` | Sadece test ekleme veya düzeltme |
| `docs` | Sadece doküman |
| `chore` | Derleme, bağımlılık, araç ayarı |

`scope` servis adıdır (`customer`, `inventory`, `order` …). Birden fazla servisi etkileyen ya da kök düzeyindeki değişikliklerde yazılmaz.

Örnekler:

```
feat(customer): add address book
fix(inventory): prevent negative stock
chore: set up maven multi-module skeleton
```

## Definition of Done

Bir task ancak aşağıdakilerin hepsi doğruysa Done kolonuna geçer:

- [ ] Tüm kabul kriterleri karşılanıyor.
- [ ] Task'ta istenen testler yazıldı ve `./mvnw verify` geçiyor.
- [ ] API, proje dokümanındaki kontrata uyuyor. Kontrat değiştiyse doküman da güncellendi.
- [ ] Verilen tasarım kararları Karar Rehberi'ne (gerekirse ADR olarak) yazıldı.
- [ ] README'deki çalıştırma adımları hâlâ doğru.
- [ ] Review yapıldı ve yorumlar kapatıldı.
- [ ] Proje dokümanında task "Bitti" olarak işaretlendi ve geliştirme günlüğüne bir satır eklendi.

## Test stratejisi

| Seviye | Neyi test eder? | Ne zaman? |
|---|---|---|
| Unit | İş kuralları, hesaplamalar, durum geçişleri | Her task |
| Slice (web, JPA, Mongo) | HTTP kontratı, sorgular, eşlemeler | Her task |
| Entegrasyon (Testcontainers) | Gerçek veritabanı ve Kafka ile akış | Faz 2'den itibaren |
| Kontrat | Servisler arası API uyumu | Faz 4'ten itibaren |
| Uçtan uca | Sepetten teslimata tüm akış | Faz 5 sonunda |

## Yerel portlar

| Bileşen | Port |
|---|---|
| api-gateway | 8080 |
| customer · catalog · inventory · cart · order · payment · notification | 8081 – 8087 |
| PostgreSQL · MongoDB · Redis | 5432 · 27017 · 6379 |
| Kafka · Keycloak · Elasticsearch | 9092 · 8180 · 9200 |
