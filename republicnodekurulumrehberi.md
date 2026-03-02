# 🚀 Republic AI Validatör Node Kurulum Rehberi (Windows WSL İçin)

Bu rehber, **Windows** işletim sisteminde **WSL (Windows Subsystem for Linux)** kullanarak sıfırdan, hiçbir kodlama geçmişi olmayan birinin bile anlayabileceği şekilde **Republic AI Validatör Node** kurulumunu aşama aşama anlatmaktadır.

---

## 🟢 Ön Hazırlık: Windows WSL Kurulumu ve Systemd Ayarı

Windows üzerinde bir Linux ortamı (Ubuntu) çalıştırmamız gerekiyor.

1. **Windows PowerShell'i Yönetici Olarak Çalıştırın.** (Başlat menüsüne PowerShell yazıp sağ tıklayın ve "Yönetici Olarak Çalıştır" deyin).
2. Aşağıdaki komutu kopyalayıp PowerShell'e sağ tıkla yapıştırın ve **Enter**'a basın:
   ```powershell
   wsl --install
   ```
   *(Bu komut Ubuntu'yu bilgisayarınıza kuracaktır. Bilgisayarınızı yeniden başlatmanız gerekebilir.)*
3. Bilgisayar açıldıktan sonra Başlat menüsünden **Ubuntu** uygulamasını açın. Sizden bir kullanıcı adı (username) ve şifre (password) belirlemenizi isteyecek. Bunları belirleyin (şifre yazarken güvenlik gereği ekranda görünmez, yazıp Enter'a basın).
4. **Systemd (Servis Yöneticisi) Aktifleştirme:**
   Node'un arka planda güvenle otomatik çalışması için `systemd` özelliğini açmamız şart. Ubuntu terminaline şu komutu yazın:
   ```bash
   sudo nano /etc/wsl.conf
   ```
   Açılan boş sayfaya şunları yapıştırın:
   ```ini
   [boot]
   systemd=true
   ```
   Kaydetmek için klavyeden sırasıyla **CTRL + O** yapıp **Enter**'a basın, sonrasında çıkmak için **CTRL + X** yapın.
5. Windows PowerShell'e (Yönetici) geri dönüp Ubuntu'yu yeniden başlatın:
   ```powershell
   wsl --shutdown
   ```
6. **Ubuntu** uygulamasını tekrar açın. Artık Node kurulumuna başlayabiliriz.

---

## 🟢 Aşama 1: Sistemi Güncelleme ve Gerekli Paketleri Kurma

Ubuntu terminalinize aşağıdaki komutu kopyalayıp yapıştırın. Bu işlem bilgisayarınızdaki Linux paketlerini güncelleyecek ve kurulum için gerekli programları indirecektir.

```bash
sudo apt update -y && sudo apt upgrade -y && sudo apt install curl wget tar jq build-essential -y
```

---

## 🟢 Aşama 2: Go (Programlama Dili) Kurulumu

Republic ağı `Go` diliyle yazılmıştır. Kurmak için sırasıyla bu komutları girin (hepsini birden kopyalayıp sağ tıkayarak yapıştırabilirsiniz):

```bash
cd $HOME
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

---

## 🟢 Aşama 3: Cosmovisor Kurulumu

**Cosmovisor**, ağda bir güncelleme geldiğinde (örneğin v3.0 gibi) node'unuzu doğru zamanda **otomatik olarak** yeni sürüme geçiren çok önemli bir programdır. Kurmak için:

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

---

## 🟢 Aşama 4: Değişkenlerin Tanımlanması

Cüzdan adınızı ve ağın kullanacağı portu belirliyoruz (`wallet` adını isterseniz değiştirebilirsiniz ama böyle kalması ileride kodları kullanırken daha kolaylık sağlar).

```bash
echo "export REPUBLIC_WALLET='wallet'" >> $HOME/.bash_profile
echo "export REPUBLIC_PORT='43'" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

---

## 🟢 Aşama 5: Node Dosyalarını İndirme ve Başlatma (Init)

İlk versiyon olan `v0.1.0`'ı indirip Cosmovisor dizinine yerleştiriyoruz:

```bash
VERSION="v0.1.0" && \
mkdir -p $HOME/.republic/cosmovisor/genesis/bin && \
curl -L "https://media.githubusercontent.com/media/RepublicAI/networks/main/testnet/releases/${VERSION}/republicd-linux-amd64" -o republicd && \
chmod +x republicd && \
mv republicd $HOME/.republic/cosmovisor/genesis/bin/ && \
ln -sfn $HOME/.republic/cosmovisor/genesis $HOME/.republic/cosmovisor/current && \
sudo cp $HOME/.republic/cosmovisor/genesis/bin/republicd /usr/local/bin/
```

**Node İsminizi Belirleyin:**
Aşağıdaki komutta `Nodeİsminiz` yazan yere ağda görünmesini istediğiniz takma adı yazın (boşluk veya Türkçe karakter kullanmadan).

```bash
republicd init Nodeİsminiz --chain-id raitestnet_77701-1 --home $HOME/.republic
```

Ağın doğuş dosyasını (`genesis.json`) indirin:
```bash
curl -s https://raw.githubusercontent.com/RepublicAI/networks/main/testnet/genesis.json > $HOME/.republic/config/genesis.json
```

---

## 🟢 Aşama 6: Port, Seed, Peer ve Pruning Ayarları

Node'unuzun diğer bilgisayarlarla haberleşebilmesi ve çakışma yapmaması için aşağıdaki **tüm kod bloğunu tümüyle kopyalayıp Ubuntu terminaline** yapıştırın:

**Port Ayarları:**
```bash
sed -i.bak -e "s%:26658%:${REPUBLIC_PORT}658%g;
s%:26657%:${REPUBLIC_PORT}657%g;
s%:6060%:${REPUBLIC_PORT}060%g;
s%:26656%:${REPUBLIC_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${REPUBLIC_PORT}656\"%;
s%:26660%:${REPUBLIC_PORT}660%g" $HOME/.republic/config/config.toml

sed -i.bak -e "s%:1317%:${REPUBLIC_PORT}317%g;
s%:8080%:${REPUBLIC_PORT}080%g;
s%:9090%:${REPUBLIC_PORT}090%g;
s%:9091%:${REPUBLIC_PORT}091%g;
s%:8545%:${REPUBLIC_PORT}545%g;
s%:8546%:${REPUBLIC_PORT}546%g;
s%:6065%:${REPUBLIC_PORT}065%g" $HOME/.republic/config/app.toml
```

**Peer ve Seed Ayarları (Diğer node'lara bağlanmak için):**
```bash
SEEDS=""
PEERS="4e14a1edc972ed3f4c03eae8434cb3997b342029@46.224.213.11:43656,c5f9653155d9095901c8044dc01fadf49212f350@45.143.198.6:26656,bb8dd41fc4731fd1b99bb054103c5c9526433bdc@149.5.246.217:43656,89f3b98f9428ce7c7bb6d48294dcceeb14446302@38.49.214.43:26656,87b1a77039b469eac7e3441ee14008cbed732ed9@38.49.214.87:26656,fb1f134e0dcd5c6c5719b318211598133fab46fb@154.12.118.199:26656"

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.republic/config/config.toml
```

**Depolama Alanından Tasarruf Etmek için (Pruning) ve İşlem Ücreti (Gas) Ayarları:**
```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.republic/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.republic/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.republic/config/app.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.republic/config/config.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "250000000arai"|g' $HOME/.republic/config/app.toml
```

---

## 🟢 Aşama 7: Node Servisini Oluşturma ve Başlatma

Aşağıdaki komutu tamamen kopyalayıp yapıştırın. Bu, Node'un arka planda güvenle bir sistem servisi olarak çalışmasını sağlar.

```bash
sudo tee /etc/systemd/system/republicd.service > /dev/null <<EOF
[Unit]
Description=Republic AI Node
After=network-online.target

[Service]
User=$USER
Environment="DAEMON_NAME=republicd"
Environment="DAEMON_HOME=$HOME/.republic"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
ExecStart=$HOME/go/bin/cosmovisor run start --home $HOME/.republic --chain-id raitestnet_77701-1
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Servisi Ubuntu'ya tanıtıp başlatalım:
```bash
sudo systemctl daemon-reload
sudo systemctl enable republicd
sudo systemctl start republicd
```

---

## 🟢 Aşama 8: Gelecek Güncellemelerin (v0.2.1 ve v0.3.0) Hazırlanması [ÇOK ÖNEMLİ]

Ağ belli blok yüksekliklerine (örneğin 326250. bloka) ulaştığında yapısını ve versiyonunu değiştirir. `Cosmovisor` kurduğumuz için bu dosyaları **şimdiden** klasörlerine koyarsak, o bloka geldiğinde **Node'unuz otomatik olarak kendi kendini günceller.** Eğer bunu yapmazsanız Node o blokta kilitlenir kalır!

**v0.2.1 Güncelleme Hazırlığı:**
```bash
mkdir -p $HOME/.republic/cosmovisor/upgrades/v0.2.1/bin
wget -O $HOME/.republic/cosmovisor/upgrades/v0.2.1/bin/republicd https://github.com/RepublicAI/networks/releases/download/v0.2.1/republicd-linux-amd64
chmod +x $HOME/.republic/cosmovisor/upgrades/v0.2.1/bin/republicd
```

**v0.3.0 Güncelleme Hazırlığı (Blok 326250'ye ulaşıldığında otomatik geçilir):**
```bash
mkdir -p $HOME/.republic/cosmovisor/upgrades/v0.3.0/bin
wget -O $HOME/.republic/cosmovisor/upgrades/v0.3.0/bin/republicd https://github.com/RepublicAI/networks/releases/download/v0.3.0/republicd-linux-amd64
chmod +x $HOME/.republic/cosmovisor/upgrades/v0.3.0/bin/republicd
```
*(Node'unuz arka planda çalışıp blokları sıfırdan indirirken, doğru yüksekliğe geldiğinde bu dosyaları kullanarak siz müdahele etmeden kendini güncelleyecektir.)*

---

## 🟢 Aşama 9: Cüzdan Oluşturma ve Faucet'ten Token Alma

Ağ üzerinde işlem yapabilmek için bir cüzdana ihtiyacınız var. Aşağıdaki komutu yazın:
```bash
republicd keys add $REPUBLIC_WALLET
```
> **⚠️ KRİTİK UYARI:** Bu komutu girdikten sonra ekranda size **12 veya 24 kelimelik bir gizli kelime öbeği (mnemonic/seed phrase)** ve bir cüzdan adresi (`rai1...` ile başlayan) verecektir. **BU GİZLİ KELİMELERİ BİR KAĞIDA NOT ALIN VE GÜVENLİ BİR YERDE SAKLAYIN.** Kaybederseniz cüzdanınıza bir daha asla erişemezsiniz.

**Token Alma:**
1. Yukarıda size verilen `rai1...` adresini kopyalayın.
2. [Republic AI Faucet](https://faucet.republicai.io) adresine gidin.
3. Cüzdan adresinizi yapıştırarak cüzdanınıza test tokeni (RAI) gönderin. Validatör kurmak için bu tokenlere ihtiyacımız var. (Günde birkaç kez token talep ederek biriktirebilirsiniz.)

---

## 🟢 Aşama 10: Node'un Senkronize Olmasını Bekleme

Validatör (doğrulayıcı) yetkisi almadan önce bilgisayarınızın ağın güncel bloklarına yetişmesini beklemeniz şarttır! Bu işlem duruma göre birkaç saat sürebilir.

Senkronizasyon (Eşitlenme) durumunu kontrol etmek için:
```bash
republicd status --node tcp://localhost:${REPUBLIC_PORT}657 2>&1 | jq '{catching_up: .sync_info.catching_up, latest_block_height: .sync_info.latest_block_height}'
```
* Çıkan yazıda `"catching_up": true` yazıyorsa **HALA EŞİTLENİYOR DEKMEKTİR, BEKLEMEYE DEVAM EDİN.**
* Ekranda `"catching_up": false` yazdığı an eşitlenmiş demektir. Artık bir sonraki aşamaya (Aşama 11) geçebilirsiniz.

*(Arka planda nelerin indirildiğini canlı izlemek isterseniz `journalctl -u republicd -f` komutunu yazabilir, izlemeyi durdurmak için klavyeden `CTRL + C` yapabilirsiniz.)*

---

## 🟢 Aşama 11: Validatör Oluşturma

Artık tüm blokları indirdiğinize (`catching_up: false` olduğuna) ve cüzdanınızda Faucet'ten aldığınız test tokenleri bulunduğuna göre validatörünüzü ayağa kaldırabilirsiniz.

Aşağıdaki metni kopyalayın, ancak terminale yapıştırmadan önce Not Defteri'ne alarak **KENDİ BİLGİLERİNİZE GÖRE DÜZENLEYİN** (örneğin Node ismini değiştirin):

```bash
PUBKEY=$(jq -r '.pub_key.value' $HOME/.republic/config/priv_validator_key.json)

cat > validator.json << EOF
{
  "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"$PUBKEY"},
  "amount": "1000000000000000000arai",
  "moniker": "BURAYA NODE İSMİNİZİ YAZIN",
  "identity": "Varsa Keybase IDnizi yazin yoksa bos birakin",
  "website": "",
  "security": "mailadresiniz@deneme.com",
  "details": "Windows WSL Republic Node Rehberi",
  "commission-rate": "0.05",
  "commission-max-rate": "0.15",
  "commission-max-change-rate": "0.02",
  "min-self-delegation": "1"
}
EOF
```

Şimdi bu dosyayı okuyup işlemleri ağa kaydetmesi için aşağıdaki komutu girin:
*(Sizden onay isterse `y` yazıp klavyeden Enter'a basın)*
```bash
republicd tx staking create-validator validator.json \
--from $REPUBLIC_WALLET \
--chain-id raitestnet_77701-1 \
--gas auto \
--gas-adjustment 1.5 \
--gas-prices "1000000000arai" \
--node tcp://localhost:${REPUBLIC_PORT}657 \
-y
```

**Tebrikler! Validatör kurulumunu başarıyla tamamladınız.** 🥳

---

## 🟢 Ekstra: İndeks Cüzdana Çevirme (Wallet Bilgileri)

Republic sisteminde bu cüzdanınızın aktifliğine veya ödüllerine bakmak ve puan kazanmak için ana cüzdan (Örn: Metamask gibi EVM adresi) olarak da bir karşılığı vardır:
1. `republicd keys show $REPUBLIC_WALLET -a` yazıp güncel rai... adresinizi görebilirsiniz.
2. Cüzdan adresi ile Explorer üzerinden Node'nunuzun `Bonded` (Aktif ve onaylı) yoksa `Unbonded` (Aktif olmayan yedek) durumunda mı olduğunu takibe alabilirsiniz. Aktife (BONDED setine) girmek için faucetten token alıp node'unuza Delegate etmeniz (yatırmanız) gerekir.

---

## 🟢 Aşama 12: Günlük ve Faydalı Komutlar

Aşağıdaki komutları Node'unuzun ne durumda olduğunu görmek veya işlem yapmak için istediğiniz zaman kullanabilirsiniz.

**1. Validatör Durumunu Kontrol Etme:**
*(Eğer "status": "BOND_STATUS_BONDED" derse her şey mükemmel. "UNBONDED" derse daha fazla delege etmeniz gerekir.)*
```bash
republicd q staking validator $(republicd keys show $REPUBLIC_WALLET --bech val -a) --node tcp://localhost:${REPUBLIC_PORT}657
```

**2. Cüzdandaki Bakiyenizi Görme:**
```bash
republicd q bank balances $(republicd keys show $REPUBLIC_WALLET -a) --node tcp://localhost:${REPUBLIC_PORT}657
```

**3. Bağlı Olduğunuz Peer (Düğüm) Sayısını Görme (En az 10 ve üzeri olmalı):**
```bash
curl -s http://127.0.0.1:${REPUBLIC_PORT}657/net_info | jq '.result.n_peers'
```

**4. Validatöre Ekstra Token Stake Etme (Delegate Yapma):**
*(Daha çok block yakalamak için cüzdanınızdaki tokenleri kendi validatörünüze yatırma işlemidir. Şu anki kodda yaklaşık 1 RAI ekler, miktar kısmını bakiyenize göre artırabilirsiniz. 1 RAI = 1000000000000000000arai)*
```bash
republicd tx staking delegate $(republicd keys show $REPUBLIC_WALLET --bech val -a) \
1000000000000000000arai \
--from $REPUBLIC_WALLET \
--chain-id raitestnet_77701-1 \
--gas auto \
--gas-adjustment 1.5 \
--gas-prices 1000000000arai \
--node tcp://localhost:${REPUBLIC_PORT}657 \
-y
```

**5. Hapis (Jail) Durumundan Çıkma:**
*(Eğer bilgisayarınız kapalı kalırsa, internetiniz kesilirse veya node çakılırsa ağ ceza kesip validatörünüzü hapse atar. Unjail komutu ile geri dönebilirsiniz)*
```bash
republicd tx slashing unjail \
--from $REPUBLIC_WALLET \
--chain-id raitestnet_77701-1 \
--gas auto \
--gas-adjustment 1.5 \
--gas-prices 1000000000arai \
--node tcp://localhost:${REPUBLIC_PORT}657 \
-y
```

**6. Kazanılan Validatör Ödüllerini Çekme:**
```bash
republicd tx distribution withdraw-rewards $(republicd keys show $REPUBLIC_WALLET --bech val -a) \
--from $REPUBLIC_WALLET \
--commission \
--chain-id raitestnet_77701-1 \
--gas auto \
--gas-adjustment 1.5 \
--gas-prices "1000000000arai" \
--node tcp://localhost:${REPUBLIC_PORT}657 \
-y
```
