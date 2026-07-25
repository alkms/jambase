# Kargo Takip Uygulaması 7 Günlük Geliştirme Planı

Bu döküman, PocketBase (Backend), Flutter (Mobil Frontend), Leaflet/OpenStreetMap (`flutter_map`) teknolojilerini kullanarak geliştirilecek gerçek zamanlı (real-time) kargo takip uygulaması için hazırlanmış 7 günlük iş planını içermektedir.

---

### **1. Gün: Sistem Tasarımı ve Backend (PocketBase) Kurulumu**
**Hedef:** Veritabanı mimarisini kurmak ve PocketBase'i ayağa kaldırmak.
*   **PocketBase Kurulumu:** PocketBase'i yerel ortamda (localhost) çalıştırın.
*   **Koleksiyonların (Tabloların) Oluşturulması:**
    *   `users`: (Hazır gelir). Rol ekleyin: `driver` (kurye) ve `customer` (müşteri).
    *   `shipments`: Kargo bilgileri. Alanlar: `title` (isim), `status` (pending, in_transit, delivered), `driver_id` (ilişki), `customer_id` (ilişki).
    *   `tracking`: Konum güncellemeleri. Alanlar: `shipment_id` (ilişki), `latitude` (ondalık), `longitude` (ondalık), `timestamp`.
*   **API Kuralları (API Rules):**
    *   Sadece ilgili kurye kendi konumunu güncelleyebilsin.
    *   Sadece ilgili müşteri kendi kargosunun konumunu okuyabilsin (Güvenlik kurallarını yazın).

### **2. Gün: Flutter Projesi Kurulumu ve Kimlik Doğrulama**
**Hedef:** Mobil uygulamanın iskeletini oluşturmak ve giriş sistemini yapmak.
*   **Proje Başlangıcı:** `flutter create cargo_tracker` ile Android odaklı projeyi oluşturun.
*   **Bağımlılıklar (pubspec.yaml):** `pocketbase` (Dart SDK), `provider` veya `bloc` (State management için).
*   **Giriş/Kayıt Ekranları:**
    *   Kurye ve Müşteri için ayrı giriş (Login) arayüzleri yapın.
    *   PocketBase SDK kullanarak e-posta/şifre ile kimlik doğrulama işlemini bağlayın.
    *   Token'ı cihazda güvenli şekilde saklayın (örn: `flutter_secure_storage`).

### **3. Gün: Konum Servisleri (Kurye Uygulaması)**
**Hedef:** Kuryenin GPS verisini almak ve PocketBase'e yazmak.
*   **Bağımlılıklar:** `geolocator` (konum almak için) ve `permission_handler`.
*   **İzinler:** Android `AndroidManifest.xml` içine `ACCESS_FINE_LOCATION` ve arka plan konum izinlerini ekleyin.
*   **Konum Dinleme (Location Stream):** Kurye "Teslimata Başla" dediğinde `geolocator` ile konum değişimlerini dinlemeye (listen) başlayın.
*   **PocketBase'e Veri Gönderme:** Konum her değiştiğinde (örneğin her 10 metrede veya 5 saniyede bir), PocketBase'deki `tracking` koleksiyonuna veya doğrudan `shipments` tablosundaki bir `current_location` alanına POST/PUT isteği atın.

### **4. Gün: Harita Entegrasyonu (Müşteri Uygulaması)**
**Hedef:** OpenStreetMap alt yapısını Flutter'da göstermek.
*   **Bağımlılıklar:** `flutter_map` (Leaflet'in Flutter karşılığıdır, OSM destekler) ve `latlong2`.
*   **Harita Arayüzü:** Müşteri paneline bir harita widget'ı ekleyin.
*   **Tile Server Ayarı:** OpenStreetMap'in ücretsiz Tile sunucularını (`https://tile.openstreetmap.org/{z}/{x}/{y}.png`) bağlayın.
*   **Statik Gösterim:** Kargonun en son konumunu PocketBase'den `GET` isteğiyle çekip haritaya statik bir "Kamyon" ikonu (Marker) yerleştirin.

### **5. Gün: Gerçek Zamanlı (Realtime) Takip Entegrasyonu**
**Hedef:** Kurye hareket ettikçe müşteri haritasındaki pin'in kayarak ilerlemesini sağlamak.
*   **PocketBase Realtime Subscribe:** Müşteri uygulamasında ilgili `tracking` veya `shipments` kaydına PocketBase'in `pb.collection('shipments').subscribe(...)` metodu ile abone olun.
*   **Harita Güncellemesi:** SSE (Server-Sent Events) üzerinden yeni bir koordinat geldiğinde, Flutter state'ini güncelleyerek Marker'ın (pin) yeni koordinata geçmesini sağlayın.
*   **Rota (Polyline) Çizimi (Opsiyonel):** Kuryenin geçtiği yolları geçmiş koordinatlardan alarak harita üzerine çizgi (`Polyline`) olarak ekleyin.

### **6. Gün: Kargo Durum Yönetimi ve UI Geliştirmeleri**
**Hedef:** Teslimat sürecini uçtan uca tamamlamak.
*   **Kurye Paneli:** "Teslim Edildi" butonu ekleyin. Kurye buna bastığında `shipments` tablosundaki durumu `delivered` yapın.
*   **Fotoğraf Yükleme:** Teslimat kanıtı için `image_picker` ile fotoğraf çekip PocketBase'e (File alanı) yükleyin.
*   **Müşteri Bildirimleri:** Durum `delivered` olduğunda harita ekranını kapatıp "Kargonuz Teslim Edildi" ekranını gösterin. UI/UX cilalamalarını (renkler, paddingler, loading animasyonları) tamamlayın.

### **7. Gün: Test, Hata Yakalama ve Canlıya Alma**
**Hedef:** Uygulamayı stabil hale getirip sunucu kurulumunu yapmak.
*   **Gerçek Cihaz Testi:** Kurye uygulamasını bir Android telefona kurup dışarıda yürüyerek/arabayla haritada anlık değişimi test edin. (Emülatörde mock location kullanabilirsiniz ama gerçek cihaz testi şarttır).
*   **Hata Yönetimi (Error Handling):** İnternet kopması durumunda kurye konumlarını yerelde (lokalde) biriktirip, internet gelince PocketBase'e toplu gönderme (Offline-first yaklaşımı) senaryolarını test edin.
*   **Canlı (Production) Ortamı:** PocketBase'i bir VPS sunucuya (DigitalOcean, Hetzner, AWS vb.) taşıyın. Domain bağlayın ve SSL sertifikasını aktif edin. (Mobil uygulamanın bağlanacağı URL'i bu canlı sunucu URL'i ile değiştirip Android APK/AAB çıktısını alın).

---

**Önemli Geliştirici Notu (Leaflet vs Flutter Map):**
Leaflet bir JavaScript kütüphanesidir. Uygulamayı native (Flutter) yapacağınız için, Leaflet'i bir *WebView* içine koymak yerine, Flutter'ın native olarak Leaflet mimarisini kullanan `flutter_map` paketini kullanmalısınız. `flutter_map`, tıpkı Leaflet gibi çalışır ve OpenStreetMap haritalarını sıfır lisans ücretiyle mobil uygulamanızda göstermenizi sağlar. Performansı WebView'a göre çok daha yüksektir.
