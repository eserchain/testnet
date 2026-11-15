# Eser Chain White Paper

Sürüm 1.0, 13 Kasım 2026. **TBD** olarak işaretli değerler mainnet launch'tan önce belirlenecek. Bu metin İngilizce
sürümün çevirisidir; iki sürüm arasında fark olursa İngilizce sürüm esas alınır.

## Özet

Eser Chain bağımsız, Ethereum uyumlu bir Proof-of-Work blockchain'dir. Block'lar orijinal Ethash algoritmasıyla mine
edilir; yaygın bir GPU'su olan herkes katılabilir. Chain, Ethereum Virtual Machine'i çalıştırır: Solidity contract'lar,
MetaMask ve bilinen Ethereum araçları değişiklik gerekmeden çalışır. Native coin ESER'dir. Premine ve satış yoktur:
var olan her ESER, genesis block'ta sabitlenmiş kurallarla mining yoluyla oluşur. Block reward 2 ESER ile başlar ve her
9.700.000 block'ta (yaklaşık dört yıl) yarıya iner; böylece block reward'lardan gelen arz yaklaşık 38,8 milyon ESER ile
sınırlı kalır.

## 1. Motivasyon

Ethereum'un Proof of Stake'e geçmesinden bu yana sıradan donanımın hâlâ block ürettiği chain sayısı az; yeni
chain'lerin çoğu da kuruculara ya da yatırımcılara ayrılmış büyük paylarla başlıyor. Eser Chain bunun tersini hedefler:

- **Açık mining.** Ethash, insanların zaten sahip olduğu GPU'larda çalışır. 4 GB'lık bir kart bugün mine edebilir;
  proje Windows ve Linux için miner build'lerini kaynak koduyla birlikte yayımlar.
- **Adil başlangıç.** İlk block mine edilmeden önce hiç coin yoktur. Kurucu da herkesle aynı kurallarla mine eder.
- **Bilinen araçlar.** Developer'lar Solidity, Remix, MetaMask ve mevcut Ethereum kütüphanelerini değiştirmeden kullanır.
- **Açık kurallar.** Client, Core-Geth'in değiştirilmemiş bir sürümüdür; chain'in kuralları herkese açık bir genesis
  dosyasıdır.

## 2. Consensus

| | |
|---|---|
| Algoritma | Ethash, orijinal (Etchash değil): 30.000 block'luk DAG epoch'ları; DAG yaklaşık 1 GB ile başlar ve her epoch'ta yaklaşık 8 MB büyür |
| Block süresi | yaklaşık 13 saniye |
| Difficulty ayarı | EIP-100 ile Homestead kuralı (uncle block'lar hesaba katılır); difficulty her block'ta en fazla 1/2048 değişir |
| Fork seçimi | En yüksek total difficulty; derin reorganization'lara karşı MESS (ECBP-1100) ile (bölüm 6) |
| Uncle block'lar | Block başına en fazla iki, en fazla altı nesil eski |

**Block süresi.** Difficulty kuralı her block'un süresini sabit 9 saniyelik bir bölenle karşılaştırır. Denge
durumunda süre 9 / ln 2 ≈ 13 saniye civarına oturur. Hedef süre bir parametre değildir: chain değiştirilmemiş bir
client'ta çalışsın diye standart kural korunmuştur.

**Uncle block'lar.** Kazanan block'la neredeyse aynı anda bulunan geçerli bir block, uncle block olarak eklenebilir.
Miner'ı block reward'ın (8 − d) / 8'ini alır; d, block'un kaç block sonra eklendiğidir (1 ile 6 arası). Uncle block'u
ekleyen miner, her uncle block için block reward'ın 1/32'si kadar ek ödül alır. Uncle reward'lar, küçük ve uzaktaki
miner'lar için ağ gecikmesinin maliyetini düşürür.

## 3. Execution layer

| | |
|---|---|
| Virtual machine | Ethereum'un Shanghai güncellemesindeki opcode'larla EVM (`PUSH0` dahil), block 0'dan itibaren aktif |
| Transaction'lar | Legacy (type 0) ve access-list (type 1) transaction'lar, EIP-155 replay koruması |
| Fee'ler | Gas price açık artırması; fee'nin tamamı miner'a gider. EIP-1559 base fee yok, hiçbir şey yakılmaz (burn yok) |
| Desteklenmeyenler | Cancun opcode'ları (`MCOPY`, `TLOAD`/`TSTORE`, `BLOBHASH`) ve `BASEFEE` |
| Gas limit | Genesis'te 30.000.000; miner'lar kademeli olarak artırıp azaltabilir |
| Chain ID | Mainnet `19811113`, testnet `20261113` |

Contract'lar EVM sürümü `shanghai` için derlenir. Deployment, çağrılar, event'ler ve kaynak kodu doğrulama testnet'te
Solidity 0.8.24, Remix ve MetaMask ile denendi. Block explorer contract kaynak kodlarını doğrular.

## 4. Para politikası

Block reward 2 ESER ile başlar ve her 9.700.000 block'ta yarıya iner. 13 saniyelik block'larla bir dönem yaklaşık dört
yıl sürer.

| Dönem | Block'lar | Block reward | Yaklaşık başlangıç | Dönem sonunda block reward'lardan gelen arz |
|---|---|---|---|---|
| 0 | 0 – 9.699.999 | 2 ESER | 0. yıl | 19,40 M |
| 1 | 9.700.000 – 19.399.999 | 1 ESER | 4. yıl | 29,10 M |
| 2 | 19.400.000 – 29.099.999 | 0,5 ESER | 8. yıl | 33,95 M |
| 3 | 29.100.000 – 38.799.999 | 0,25 ESER | 12. yıl | 36,38 M |
| 4 | 38.800.000 – 48.499.999 | 0,125 ESER | 16. yıl | 37,59 M |
| 5 | 48.500.000 – 58.199.999 | 0,0625 ESER | 20. yıl | 38,19 M |

Reward 61. halving'de sıfıra iner. Block reward'lar toplamda yaklaşık **38,8 milyon ESER** üretir. Uncle ve nephew
reward'lar kullanıma bağlı küçük bir miktar ekler; transaction fee'leri mevcut ESER'i miner'lara aktarır, yeni ESER
üretmez. "Yıllar" ortalama 13 saniyelik block süresini varsayar; gerçek takvim difficulty ayarına bağlıdır.

Takvim genesis dosyasına 62 aktivasyon noktası (`blockReward`) olarak yazılmıştır. Kısa aralıklı ayrı bir chain'de
test edildi: client beklenen yüksekliklerde 2, 1, 0,5, 0,25 ve 0,125 ESER ödedi. Mining pool da reward'ları aynı
takvimle hesaplar.

## 5. Adil başlangıç (fair launch)

- **Premine yok.** Genesis block kimseye coin ayırmaz.
- **Satış yok.** ICO, ön satış ya da token satışı yoktur ve proje ESER satmaz.
- **Herkese aynı kurallar.** Kurucu diğer miner'lar gibi mine eder.
- **Genesis mesajı.** Genesis block şu cümleyi taşır:
  *"Peace at home, peace in the world. M.K. Ataturk"* (Yurtta sulh, cihanda sulh. M.K. Atatürk).
- **Genesis dosyası.** Launch'taki genesis dosyası ve SHA-256'sı yayımlanır. Node operatörleri bu SHA-256'yı kontrol
  etmelidir: reward takvimi dosyanın bir parçasıdır ama genesis block hash'inin parçası değildir. Yanlış dosyayla
  çalışan bir node dört yıl boyunca aynı chain'i izler, ilk halving'de ise chain'den ayrılır.

## 6. Güvenlik

**Çoğunluk saldırıları.** Her Proof-of-Work chain'de hash power'ın çoğunu kontrol eden kişi yakın geçmişi yeniden
yazabilir; örneğin bir ödemeyi geri alabilir (51 % saldırısı). Az hash power'a sahip genç bir chain, daha fazlasını
kiralayan ya da elinde bulunduran herkese açıktır. Kalıcı savunma çok sayıda bağımsız miner'dır; aşağıdaki önlemler o
zamana kadar zararı azaltır.

**MESS (ECBP-1100).** Core-Geth'in Modified Exponential Subjective Scoring'i block 0'dan itibaren aktiftir. Bir node,
geç gelen rakip bir chain parçasından kendi chain'inden çok daha fazla total difficulty ister; yaklaşık yedi saatten
eski parçalar için bu oran yaklaşık 31 kata kadar çıkar. Kısa, doğal reorganization'lar etkilenmez. Private bir ağdaki
testte, dürüst chain'in 1,56 katı total difficulty'ye sahip 45 dakikalık private bir chain yayımlandı: beş MESS node'u
onu reddetti, MESS'i kapalı bir node ise saldırganı izlemek için 154 block'luk reorganization yaptı.

MESS'in sınırları: sadece en az beş peer'ı olan, tamamen sync olmuş node'larda devreye girer; izole kalmış ya da
saldırı sırasında ağa katılan node'lar yanıltılabilir; çok daha fazla hash power'a sahip kararlı bir saldırgan yine de
kazanabilir; MESS'li node'larla diğer node'lar anlaşamazsa ağ, taraflardan biri geride kalana kadar ikiye bölünebilir.
Exchange'ler, pool'lar ve servisler yeterli peer'ı olan kendi node'larını çalıştırmalı ve MESS'i açık tutmalıdır.

**Confirmation'lar.** Servisler tutar büyüdükçe daha fazla, genç bir ağda ise daha da fazla confirmation beklemelidir.
Mining pool reward'ları ancak 32 block sonra hesaplara yazar.

**İşletim.** Wallet key'leri asla paylaşılmaz; public RPC endpoint'leri hiçbir hesap fonksiyonu sunmaz; pool ve faucet
wallet'ları sadece işletme için gereken miktarı tutan hot wallet'lardır.

## 7. Ağ ve yazılım

| Bileşen | Yazılım | Notlar |
|---|---|---|
| Node | Core-Geth v1.13.0, resmi sürüm, değiştirilmemiş | Linux x86-64 ve arm64 test edildi |
| Bootnode'lar | Proje tarafından çalıştırılan üç bootnode, iki ayrı ağda | Bağımsız bootnode'lar memnuniyetle karşılanır |
| Mining pool | Proje düzeltmeleriyle open-etc-pool-friends | PPLNS, 1 % fee, 0,2 ESER'den itibaren otomatik payout |
| Explorer | Blockscout | Contract kaynak kodu doğrulama |
| Faucet | Proje servisi | Sadece testnet |
| Public RPC | Rate-limit'li endpoint | Yoğun kullananlar kendi node'unu çalıştırmalı |

**Test edilen miner'lar.** ethminer 0.19 (proje build'leri, OpenCL, 4 GB kartlar için split DAG); lolMiner 1.98a;
RTX 30 serisi kartlarda GMiner 3.44 ve T-Rex 0.26.8 (bu ikisi RTX 50 serisi kartlarda açılmıyor). AMD kartlar henüz
test edilmedi.

Yapılandırmanın, rehberlerin ve yukarıdaki testlerin sonuçlarının tamamı herkese açıktır: https://eserchain.org ve
https://github.com/eserchain/testnet.

## 8. Yönetişim

Eser Chain, kurucu tarafından yönetilen bir projedir. Protokol kurallarını ve yol haritasını kurucu belirler.

- Kural değişiklikleri hard fork'tur. Aktivasyon block numarası, gerekçesi ve kod ya da yapılandırma değişikliğiyle
  birlikte önceden duyurulur ve public repository'de yayımlanır.
- Bir değişiklik, ağda ancak node operatörleri ve miner'lar yeni sürümü çalıştırdığında geçerli olur. Güncelleme
  yapmayan node'lara kimse bir değişikliği zorla kabul ettiremez.
- Değişiklikler mainnet'e gelmeden önce testnet'te denenir.

## 9. Yol haritası

| Aşama | Durum |
|---|---|
| Lokal test ağı, pool, explorer, miner build'leri | Tamamlandı |
| Public testnet (chain ID 20261113), üç bootnode, pool, explorer, faucet, public RPC | 13 Kasım 2026'dan beri yayında |
| Testnet'in gerçek kullanımı: dışarıdan miner'lar ve node'lar, birkaç hafta sorunsuz çalışma | Devam ediyor |
| Mainnet hazırlığı: ayrı altyapı, bağımsız bootnode lokasyonları, pool wallet, başlangıç difficulty'si, güvenlik incelemesi | Sırada |
| Mainnet launch (chain ID 19811113) | **TBD** |

## 10. Sorumluluk reddi

Bu belge yazılımı ve ağ kurallarını anlatır. Herhangi bir şeyin alım ya da satım teklifi, yatırım tavsiyesi ya da
değer vaadi değildir. ESER'in proje tarafından belirlenmiş bir fiyatı yoktur. Testnet ESER'in değeri yoktur. Mining
elektrik harcar ve donanımı yıpratır; blockchain'ler arızalanabilir, bölünebilir ya da saldırıya uğrayabilir; yazılım
hata içerebilir. Mine eden, node çalıştıran ya da ESER tutan herkes bunu kendi sorumluluğunda yapar. Proje hiçbir zaman
private key, recovery phrase ya da parola istemez.
