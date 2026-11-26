# ShopLab

Java 21 ve Spring Boot 4 ile yazılan, çok servisli bir e-ticaret backend'i. Amaç, gerçek bir sistemin parçalarını (servisler arası iletişim, olay tabanlı mimari, saga, güvenlik, gözlemlenebilirlik, Kubernetes) adım adım kurarak öğrenmek.

## Modüller

| Modül | Port | Durum |
|---|---|---|
| `customer-service` | 8081 | Faz 1 · iskelet |

Yeni servisler eklendikçe bu tablo güncellenir. Planlanan servisler: catalog, inventory, cart, order, payment, notification.

## Gereksinimler

- JDK 21
- Git

Maven kurmanız gerekmez: repo, kendi Maven sürümünü getiren Maven Wrapper (`./mvnw`) ile gelir.

## Derleme ve test

```bash
./mvnw clean verify
```

## Çalıştırma

```bash
./mvnw -pl customer-service spring-boot:run
```

Servisin ayakta olduğunu kontrol etmek için:

```bash
curl localhost:8081/actuator/health
# {"status":"UP", ...}
```

## Çalışma kuralları

Branch, commit ve "bitti" tanımı için: [docs/standards.md](docs/standards.md)
