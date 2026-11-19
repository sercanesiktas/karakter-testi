# karakter-testi

Görsel üretim modellerinin Türkçe harfleri (ç, ğ, ı, İ, ö, ş, ü) doğru yazıp yazmadığını ölçen küçük bir test seti ve puanlama aracı.

Difüzyon modelleri okunabilir yazı basmakta zorlanır. Türkçede bu sorun harf aksanlarında belirginleşir: tabelaya "Şişli" yazdırıldığında "Sisli" çıkabilir, `ı` ile `i` birbirine karışabilir. Bu proje aynı 20 prompt ile farklı modelleri karşılaştırmayı ve sonucu sayıya dökmeyi sağlar.

## Yöntem

1. `testler.json` içinde 20 test vardır: harf dizileri, kelimeler, büyük harfli kelimeler, `i`/`ı` çiftleri ve cümleler. Hepsi aynı tabela şablonuyla istenir.
2. Her test için bir görsel üretilir. Görseller API ile üretilebilir ya da bir arayüzden elle indirilebilir.
3. `puanla.py` görseli Tesseract ile okur (Türkçe dil verisi) ve okunan metni beklenenle karşılaştırır.

Her görsel için üç ölçüt hesaplanır:

| Ölçüt | Açıklama |
|---|---|
| Tam doğru | Okunan metin beklenenle harfi harfine aynıdır. Büyük/küçük harf farkı sayılmaz. |
| Özel harf doğru | Beklenen metindeki `ç ğ ı ö ş ü İ I` harflerinden doğru okunanların oranı. |
| Aksan düştü | Harf okunmuş ama aksanı yok: `ş` yerine `s`, `İ` yerine `I` gibi. |

Sonuçlar `sonuclar/<model>.json` dosyasına yazılır. `rapor.py` hepsini [SONUCLAR.md](SONUCLAR.md) tablosunda toplar.

## Kurulum

Python 3.9 veya üzeri ile Tesseract ve Türkçe dil verisi gerekir. Başka bir Python paketi gerekmez.

```bash
sudo apt install tesseract-ocr tesseract-ocr-tur
git clone https://github.com/sercanesiktas/karakter-testi.git
cd karakter-testi
```

## Kullanım

Görselleri API ile üretmek için:

```bash
CLOUDFLARE_ACCOUNT_ID=... CLOUDFLARE_API_TOKEN=... python3 uret.py cloudflare
GEMINI_API_KEY=... python3 uret.py gemini
```

Çıktılar `ciktilar/<saglayici>-<model>/` altına yazılır. Bir model adı belirtmek için komuta ikinci argüman olarak eklenir. Kota ya da bakiye bittiğinde (HTTP 402 ve 429) betik durur, yeniden çalıştırıldığında üretilmiş görselleri atlayıp kaldığı yerden devam eder.

Görseller elle üretilecekse, `testler.json` içindeki promptlar kullanılarak `ciktilar/<model-adi>/<test-id>.png` biçiminde kaydedilir (`harf-01.png`, `kelime-05.png` gibi).

Puanlama ve rapor:

```bash
python3 puanla.py ciktilar/<model-adi>
python3 rapor.py
```

## Ölçümün güvenilirliği

OCR'ın kendisi de hata yapabilir, bu yüzden puanlayıcı sentetik görsellerle doğrulanmıştır. Aynı 20 metin DejaVu Sans Bold ile kusursuz çizilmiş, bir kopyasında aksanlar elle düşürülmüştür:

| Girdi | Tam doğru | Özel harf doğru | Aksan düştü |
|---|---:|---:|---:|
| Doğru metin | %100 | %100 | %0 |
| Aksanı düşürülmüş metin | %0 | %6 | %94 |

Bu değerler temiz, düz ve yüksek kontrastlı bir yazı için geçerlidir. Gerçek model çıktılarında yazı eğik, dokulu ya da stilize olabilir ve OCR bu durumda tek başına hata yapabilir. Düşük bir puan her zaman modelin hatası anlamına gelmez, şüpheli durumlarda `sonuclar/<model>.json` içindeki `okunan` alanı ile görselin kendisi karşılaştırılmalıdır. Tek başına duran `ı` ve `İ` harfleri OCR tarafından okunamadığı için tek-harf testleri harf dizisi olarak yazılmıştır.

## Sınırlamalar

- Henüz gerçek bir modelle çalıştırılmış sonuç yoktur. `uret.py` içindeki Cloudflare ve Gemini istekleri API anahtarı olmadan denenemediği için yalnızca sahte yanıtlarla test edilmiştir.
- Gemini için varsayılan model `gemini-3.1-flash-image-preview` olarak ayarlıdır. Model adı değişirse `python3 uret.py gemini <model-adi>` ile verilebilir.
- 20 test küçük bir örneklemdir. Sonuçlar tek bir koşuya bağlıdır, aynı testi birkaç kez çalıştırıp ortalamak daha güvenilir olur.

Görsel modellerin yazı basma sorununa dair ölçümler için [HayalAI istatistikleri](https://github.com/sercanesiktas/hayalai/blob/main/ISTATISTIK.md) ve [sağlayıcı notları](https://github.com/sercanesiktas/hayalai/blob/main/SAGLAYICI-NOTLARI.md) sayfalarına bakılabilir.

## Test

```bash
pip install pillow
python3 -m unittest discover -s test
```

Pillow yalnızca testlerde sentetik görsel çizmek için kullanılır.

## Lisans

MIT, bkz. [LICENSE](LICENSE).
