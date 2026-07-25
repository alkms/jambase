# PocketBase Geliştirme Önerileri

PocketBase projesi, tek bir dosyada çalışan, gömülü (embedded) veritabanı (SQLite) içeren oldukça yetenekli ve genişletilebilir bir Go arka uç (backend) framework'üdür. Proje üzerinde geliştirme yaparken dikkate almanız gereken bazı temel öneriler aşağıda listelenmiştir.

## 1. Kurulum ve Çalıştırma Ortamı
- **Go Sürümü:** Proje `go 1.25.0` gereksinimi ile çalışmaktadır. Ortamınızda Go 1.25 veya daha güncel bir sürümün kurulu olduğundan emin olun.
- **Derleme Parametreleri:** Uygulamayı bağımsız bir çalıştırılabilir dosya olarak derlerken, C bağımlılıklarından kurtulmak ve pure-Go SQLite sürücüsünü (modernc.org/sqlite) kullanmak için `CGO_ENABLED=0` bayrağını kullanın (Örn. `CGO_ENABLED=0 go build`). Bu, özellikle çapraz derleme (cross-compilation) yaparken (örn. farklı mimari veya işletim sistemleri için) işinizi çok kolaylaştıracaktır.

## 2. Geliştirme Modelleri
- **Standalone Uygulama (Bağımsız Uygulama):** Sadece hazır özellikleri kullanmak istiyorsanız, projenin `examples/base/main.go` dizininden derleyip, hazır JavaScript (JS VM) desteği ile birlikte genişletebilirsiniz.
- **Go Framework Olarak Kullanım:** PocketBase'i kendi projenizde bir kütüphane (`github.com/pocketbase/pocketbase`) olarak içe aktararak (import), kendi iş mantığınızı (business logic) çok rahat ekleyebilirsiniz. `app.OnServe().BindFunc(...)` ve benzeri Event Hook'larını kullanarak özel API rotaları oluşturabilir ve veritabanı olaylarına müdahale edebilirsiniz.

## 3. Test ve Doğrulama
- **Birim ve Entegrasyon Testleri:** Projenin içerisindeki var olan testleri periyodik olarak çalıştırın. Yeni bir özellik eklediğinizde veya hata düzeltmesi yaptığınızda her zaman standart `go test ./...` komutuyla herhangi bir regresyon olup olmadığını kontrol edin.

## 4. Kod Standartları ve Linter Kullanımı
- **golangci-lint:** Proje klasöründeki `golangci.yml` dosyasında kapsamlı bir konfigürasyon mevcuttur. Kod yapısında tutarlılığı sağlamak için, `govet`, `staticcheck`, `gofmt`, `goimports`, `misspell` vb. araçlar kullanılmaktadır. Geliştirme yaparken yerel ortamınızda sık sık linter çalıştırarak kod kalitesinden emin olmanız tavsiye edilir.

## 5. Genişletilebilirlik ve Eklentiler (JavaScript vs Go)
- **JavaScript Desteği:** Projenin JS eklentisi varsayılan olarak devrededir (`goja` bağımlılıkları vasıtasıyla). Bu sayede hiç Go kodu yazmadan, doğrudan JavaScript kullanarak uygulamanıza özel hook'lar yazabilirsiniz. Ancak, performansa kritik derecede ihtiyaç duyduğunuz ağır işlemlerde Go ile eklenti yazmanız (framework yaklaşımı) her zaman daha performanslı ve güvenilir olacaktır.