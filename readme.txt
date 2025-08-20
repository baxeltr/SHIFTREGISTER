74HC165 TABANLI PARALEL-SERİ SHIFT REGISTER DEVRESİ
==================================================

Bu proje, 8-bit'lik paralel veriyi seri veriye dönüştürmek için tasarlanmış bir elektronik devredir. Devrenin kalbinde 74HC165 (PISO - Parallel-In, Serial-Out) shift register entegresi bulunmaktadır. Bu devre, bir mikrodenetleyicinin sadece 3 pini kullanılarak 8 farklı dijital girişin (buton, switch, sensör vb.) durumunu okumasına olanak tanır.

(Bu belgenin yanında bulunan devre şeması: image_b108a4.png)

Genel Bakış
-----------
Devre, üzerindeki 8'li DIP switch ile manuel olarak 8-bit'lik bir veri girişi yapılmasına olanak tanır. Her bir girişin durumu, yanındaki yeşil LED ile görsel olarak takip edilebilir. Devreye güç verildiğini gösteren bir adet kırmızı güç LED'i de bulunmaktadır. Okunan veri, mikrodenetleyiciye seri olarak (tek bir data pini üzerinden) gönderilir.

Özellikler
----------
- Entegre: 74HC165 PISO Shift Register
- Giriş Biti: 8-bit
- Besleme Gerilimi: +3.3V
- Giriş Yöntemleri:
  - Dahili 8'li DIP Switch (SW2)
  - Harici bağlantı için 9-pinli header (J5)
- Görsel Göstergeler:
  - 1 adet Kırmızı Güç LED'i (D1)
  - 8 adet Yeşil Giriş Durum LED'i (D10-D17)
- Çıkış Arayüzü: 4-pinli header (J2) üzerinden seri data, saat (clock) ve yükleme (load) bağlantısı.

Çalışma Prensibi
----------------
1. Veri Girişi: Devre üzerindeki SW2 DIP switch'leri kullanılarak 8-bit'lik paralel veri ayarlanır. Bir switch ON (kapalı) konumuna getirildiğinde ilgili data hattını +3V3'e çeker (Mantıksal '1') ve bağlı olduğu yeşil LED yanar. OFF (açık) konumunda ise hat GND'ye çekilir (Mantıksal '0') ve LED söner. Alternatif olarak J5 konnektörü üzerinden de dışarıdan 8-bit'lik veri girilebilir.

2. Veriyi Yükleme (Latching): Mikrodenetleyici, HC165_LOAD pinini kısa bir süreliğine LOW (0) yapıp tekrar HIGH (1) yaptığında, DIP switch'lerin o anki durumu (8 bit'lik veri) 74HC165'in dahili paralel yazmaçlarına (register) yüklenir.

3. Veriyi Okuma (Shifting): Veri yüklendikten sonra, mikrodenetleyici HC165_CLK pinine her bir saat darbesi (clock pulse) gönderdiğinde, dahili yazmaçtaki veri bir bit sağa kaydırılır ve en anlamlı bit (MSB) HC165_DATA pininden dışarıya seri olarak aktarılır. Bu işlem 8 kez tekrarlandığında tüm 8 bit'lik veri okunmuş olur.

ÖNEMLİ NOT: Şematikte seri veri çıkışı olarak entegrenin 7 numaralı pini (Q) kullanılmıştır. Bu pin, standart seri çıkış olan Q7'nin (9 numaralı pin) TERSLENMİŞ (inverted) halidir. Bu nedenle, mikrodenetleyici tarafında okunan verinin bit'lerini yazılımsal olarak terslemeyi (NOT işlemi) unutmayın.

Devre Bileşenleri (BOM)
-----------------------
Referans      Değer/Parça              Adet
-----------   ----------------------   ----
U2            74HC165                  1
SW2           DIP Switch 8-Pozisyon    1
D1            Kırmızı LED              1
D10-D17       Yeşil LED                8
R1, R10-R17   330R                     9
C2            100nF                    1
J2            1x04 Pin Header          1
J5            1x09 Pin Header          1

Bağlantı Arayüzü (J2 Pinout)
----------------------------
Mikrodenetleyici bağlantısı bu konnektör üzerinden yapılır:

- Pin 1 (GND): Toprak (Ground) bağlantısı.
- Pin 2 (HC165_LOAD): Yükleme pini (PL - Parallel Load). Genellikle 'Chip Select (CS)' veya 'Latch' olarak da adlandırılır.
- Pin 3 (HC165_CLK): Saat pini (CP - Clock Pulse). Genellikle 'SCK' olarak adlandırılır.
- Pin 4 (HC165_DATA): Seri veri çıkışı (Q - Inverted Output). Genellikle 'MISO' veya 'Data' olarak adlandırılır.

Arduino/Mikrodenetleyici Örnek Kodu
-----------------------------------
Aşağıda devreyi bir mikrodenetleyici ile nasıl okuyabileceğinize dair basit bir Arduino benzeri kod örneği bulunmaktadır.

// Pin tanımlamaları
const int loadPin = 2;   // HC165_LOAD (PL) pinine bağlı
const int clockPin = 3;  // HC165_CLK (CP) pinine bağlı
const int dataPin = 4;   // HC165_DATA (Q) pinine bağlı

void setup() {
  Serial.begin(9600);

  // Pin modlarını ayarla
  pinMode(loadPin, OUTPUT);
  pinMode(clockPin, OUTPUT);
  pinMode(dataPin, INPUT);

  // Başlangıç durumları
  digitalWrite(clockPin, LOW);
  digitalWrite(loadPin, HIGH);
}

void loop() {
  // Shift register'dan 8-bit veri oku
  byte data = readShiftRegister();

  // Okunan veriyi seri port'a yazdır
  Serial.print("Okunan Veri (Binary): ");
  Serial.println(data, BIN);

  delay(500); // Her yarım saniyede bir oku
}

// 74HC165'ten veri okuyan fonksiyon
byte readShiftRegister() {
  byte incomingData = 0;

  // 1. Veriyi register'a yükle (load pulse)
  digitalWrite(loadPin, LOW);
  delayMicroseconds(5);
  digitalWrite(loadPin, HIGH);
  delayMicroseconds(5);

  // 2. 8 bit'i seri olarak oku
  // Önce MSB (En Anlamlı Bit) gelir
  for (int i = 0; i < 8; i++) {
    // Mevcut veriyi bir bit sola kaydır
    incomingData = incomingData << 1;

    // Data pinini oku ve en sağdaki bite ekle
    if (digitalRead(dataPin) == HIGH) {
      incomingData |= 1;
    }

    // Bir sonraki biti okumak için clock palsi gönder
    digitalWrite(clockPin, HIGH);
    delayMicroseconds(5);
    digitalWrite(clockPin, LOW);
    delayMicroseconds(5);
  }

  // ÖNEMLİ: Devre şemasında çıkış olarak terslenmiş (Q) pin kullanıldığı için,
  // okunan verinin bit'lerini tersle (invert et).
  incomingData = ~incomingData;

  return incomingData;
}