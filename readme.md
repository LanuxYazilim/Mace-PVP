# MacePvP

> Bu Dosyayı Tamamen Okumadan KESINLIKLE Modu Kullanmayınız!

Minecraft **1.21.11** (Fabric) için istemci taraflı mace PvP yardımcı modu.

Mod tamamen client-side çalışır, sunucuya hiçbir özel paket göndermez ve **hiç
mixin kullanmaz** — sadece Fabric API olaylarına bağlanır. Bu, sürüm geçişlerinde
kırılma yüzeyini küçük tutar.

---

## Özellikler

### 1. Kendi mace vuruşunun nereye ineceğini gösterir
Havadayken düşüşün vanilla fizik kurallarıyla tick tick simüle edilir ve yere
değeceğin nokta dünyada işaretlenir:

- iç içe halkalar + artı işareti (hasara göre yeşil → sarı → kırmızı renk geçişi)
- düşüş yörüngesi çizgisi
- oyuncudan çarpma noktasına dikey çizgi
- smash'in çevreye vurduğu **3.5 bloklık** etki alanı halkası
- çarpma noktası ekran dışındaysa ekran kenarında yön göstergesi

Simülasyon vanilla `LivingEntity.travel()` sırasını birebir taklit eder
(yerçekimi 0.08, dikey sönümleme 0.98, hava sürtünmesi 0.91). Terminal hız
-3.92 blok/tick olarak doğrulandı. Basılı W/A/S/D tuşları da hesaba katılır, yani
havada yön verirken işaret seninle birlikte kayar.

### 2. Rakip hitbox'ları
Menzildeki oyuncuların çarpışma kutuları çizilir. Ek olarak:

- zemindeki ayak izi halkası
- baş üstünde isim / can / mesafe etiketi
- mace tutan ve düşmekte olan rakipler ayrı renkte (tehdit rengi) gösterilir

> **Duvar arkası görünüm yok.** Minecraft 1.21.9 ile gelen Blaze3D yeniden
> yazımında derinlik testi kapalı hazır bir çizgi katmanı kalmadı (eski
> `getDebugLineStrip`). Hitbox'lar yalnızca göz hizanda görünen oyuncular için
> çizilir; `hitboxTranslucent` seçeneği yarı saydam çizgi kullanır, duvar
> arkasını göstermez.

### 3. Rüzgar topu + inci ile yükselme makrosu (Sol Alt)
Klavye/fare hareketini taklit ederek diziyi kendisi oynatır. Gerçek `use`
tuşuna basılmış gibi davranır (`KeyBinding.setKeyPressed` + `onKeyPressed`),
yani esyayı Minecraft'ın kendi girdi yolu kullanır. Bakış açısı anında zıplamaz,
birkaç tick içinde yumuşak bir eğriyle döner — fare hareketi taklidi.

Varsayılan dizi:

1. inciye geç → neredeyse dik yukarı bak → at
2. rüzgar topuna geç → ayağının dibine bak
3. rüzgar topunu patlat → yukarı fırla
4. (seçenek) tepe noktasında ikinci rüzgar topu ile ek yükseklik
5. mace'e geç → açıları geri al veya en yakın hedefe nişan al

> **Not — dizinin mantığı:** yukarı atılan inci, smash sonrası güvenli iniş /
> kaçış içindir; yüksekliği sağlayan rüzgar topudur. Önce fırlayıp inciyi
> havadayken atmayı tercih ediyorsan `macroPearlFirst = false`, inciyi hiç
> kullanmamak için `macroThrowPearl = false` yap. Tüm açılar, gecikmeler ve adım
> sırası config'ten ayarlanır.

Sadece rüzgar topuyla dikey fırlama için ayrı tuş: **V**.
Makro çalışırken sneak'e basmak (veya aynı tuşa tekrar basmak) iptal eder.

### 4. Rakibin mace'inin nereye ineceği
Düşmekte olan ve mace tutan rakipler için aynı simülasyon çalıştırılır:

- rakibin çarpma noktası kırmızı işaretle gösterilir
- çarpma noktasının etrafına smash etki alanı (tehlike halkası) çizilir
- **sen o alanın içindeysen** halka yanıp söner ve ekranın ortasında
  `DİKKAT: RAKİP SMASH!` uyarısı + sana gelecek tahmini hasar çıkar
- vuruş seni öldürecekse `ÖLDÜRÜCÜ` etiketi eklenir

Uzaktaki oyuncuların hızı sunucudan güvenilir gelmediği için hız ve düşüş
mesafesi konum farklarından ayrıca türetilir (`PlayerMotionTracker`).

---

## Ek özellikler (mace PvP performansı için)

| Özellik | Açıklama |
|---|---|
| **Hasar paneli** | Anlık düşüş, çarpmaya kalan süre, tahmini hasar (kalp cinsinden), hedefin vuruştan sonra kalan canı |
| **Zırh hesabı** | Zırh, zırh sertliği, Koruma büyüsü, Direnç efekti ve Breach büyüsü hesaba katılır |
| **Density / Sharpness** | Elindeki mace'in büyüleri hasar tahminine dahil edilir |
| **Öldürücü uyarısı** | Vuruş hedefi öldürüyorsa panelde `ÖLDÜRÜCÜ` yazar |
| **Hayalet kutu** | Hedefin **senin çarpma anındaki** tahmini konumu ayrı bir kutu olarak çizilir — hareketli hedefe önden nişan almayı sağlar |
| **Bekleme halkası** | Mace'in saldırı bekleme süresi nişangâhın etrafında halka olarak; dolunca yeşile döner |
| **Mermi önizleme** | Elinde rüzgar topu veya inci varken atışın nereye gideceği çizilir |
| **Su algılama** | İniş suya ise ayrı renk + "smash bonusu yok" uyarısı |
| **Çok alçak uyarısı** | Düşüş 1.5 bloğun altındaysa smash bonusu olmayacağını söyler |
| **Performans ölçümü** | Tahmin hesabının kaç µs sürdüğü ve kaç rakip izlendiği (isteğe bağlı) |

### Performans notları
- Tahminler kare başına değil **tick başına bir kez** hesaplanır; çizim ve HUD
  yalnızca hazır sonucu okur.
- `predictionIntervalTicks` ile hesap sıklığı seyreltilebilir (1 = her tick).
- Rakipler mesafeye göre sıralanır ve `maxTrackedPlayers` ile sınırlanır.
- Rakip tahminleri "hassas çarpışma" modu kapalı çalışır (5 ışın yerine 1).
- Menzil dışındaki oyuncular hiç işlenmez.

---

## Tuşlar

| Tuş | İşlev |
|---|---|
| **Sol Alt** | Rüzgar topu + inci yükselme makrosu |
| **V** | Sadece rüzgar topu ile fırlama |
| **Sağ Alt** | Modu tamamen aç/kapat |
| **H** | Hitbox'ları aç/kapat |
| **J** | Çarpma tahminlerini aç/kapat |
| **Sağ Shift** | Ayarlar ekranı |

Tuşlar oyun içi `Seçenekler → Kontroller` menüsünden değiştirilebilir.

---

## Ayarlar

Oyun içi ayar ekranı (**Sağ Shift**) 4 sayfadır: Çarpma Tahmini, Rakipler,
Hasar/HUD/Performans, Makro. Ayarlar `config/macepvp.json` dosyasına yazılır ve
renkler dahil her şey elle de düzenlenebilir (renk formatı `0xAARRGGBB`).

---

## Kurulum

### Yan Taraftaki Sürüm Numarasından İndirebilirsiniz.


---

## Ban riski

Kısa cevap: **mod sunucuya hiçbir özel paket göndermez, ama bazı özellikleri
çoğu rekabetçi sunucunun kurallarını ihlal eder.** Risk özelliğe göre değişir:

| Özellik | Sunucu paketlerden görebilir mi? | Kural riski |
|---|---|---|
| Çarpma tahmini, hasar paneli, HUD, bekleme halkası | **Hayır** — davranışın hiç değişmez, sadece zaten istemcinde olan bilgiyi çizer | Düşük. Çoğu sunucu HUD modlarına izin verir; katı sunucular "prediction" modlarını da yasaklayabilir |
| Rakip hitbox (duvar arkası **yok**, sadece görünen oyuncular) | **Hayır** | Orta. Gerçek wallhack değil ama birçok sunucu "ESP/hitbox" modlarını yine de yasaklar; screenshare kontrolünde mods klasöründen yakalanır |
| Mermi yörünge önizlemesi | **Hayır** | Orta-yüksek. Birçok büyük sunucu "trajectory" modlarını yasaklar |
| Makro (Sol Alt / V) | **Evet** — bakış açısı değişimini ve paket zamanlamasını sunucu görür | **En yüksek.** Tek tuşla çoklu eylem + kusursuz düzgün açı eğrisi, anticheat'lerin (Grim, Vulcan, Matrix, Polar) tam olarak baktığı şeydir |
| `macroAimAtTargetAfter` (otomatik nişan) | **Evet** | **En yüksek.** Bu fiilen aimbot sayılır |

Sunucu tipine göre:

- **Tek kişilik dünya / arkadaş sunucusu:** risk yok, rahatça kullan.
- **Vanilla anarchy tarzı, mod serbest sunucular:** genelde sorun yok.
- **Hypixel, MCCI ve benzeri büyük ağlar / ciddi Türk PvP sunucuları:** ESP,
  trajectory ve makro açıkça yasak. Yakalanırsan ban gelir.

**Riski düşürmek için** (`config/macepvp.json` veya Sağ Shift ayar ekranı):

```
enemyHitbox          = false     hitbox çizimini tamamen kapatır
projectilePreview    = false     yörünge önizlemesini kapatır
macroEnabled         = false     tuş simülasyonunu tamamen kapatır
macroAimAtTargetAfter= false     (zaten varsayılan olarak kapalı)
```

Bunlar kapalıyken geriye kalan (kendi çarpma tahminin + hasar paneli + bekleme
halkası) sunucuya görünmeyen, davranışını değiştirmeyen bir HUD modudur — en
düşük riskli profil budur. Yine de "izin verilir" garantisi değildir; oynadığın
sunucunun kural sayfasına bakmak sana düşer.

Bu modda anticheat atlatmaya yönelik hiçbir şey yok (rastgeleleştirilmiş makro
zamanlaması, gizleme, paket manipülasyonu vb.) ve bu tür özellikler eklenmeyecek.
Sorumluluk kullanıcıdadır.

## Lisans

MIT
