<h1 align="center">Stratos Validator kurulum rehberi :atom_symbol:</h1>

![image](https://user-images.githubusercontent.com/101149671/181047640-425da52b-6e1b-40d1-97b5-981de691937d.png)

# Stratos Türkiye Telegram grubu: [Stratos Türkiye](https://t.me/StratosTurkish)

## Sistem gereksinimleri:

```
8GB RAM
160 GB SSD
4 vCPU
```

<h1 align="center">Validator kurulumu:</h1>

## root yetkisi:
```
sudo su
```

## root dizinine gidiyoruz:
```
cd /root
```

## Sistem güncellemesi:
```
sudo apt update && sudo apt upgrade -y
```

## Kütüphane kurulumu:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y < "/dev/null"
```

## Binary dosya kurulumu:
```
cd $HOME
wget https://github.com/stratosnet/stratos-chain/releases/download/v0.8.0/stchaind
```

## stchaind

```
md5sum stchain*
```

### :point_right: Komutu girdikten sonra görselde ki gibi çıktı alıcaksınız

![image](https://user-images.githubusercontent.com/101149671/181049516-5dcdc3cf-b3f0-4a06-815b-e63aa10a0ee3.png)

## Binary dosyalarına execute yetkisi verelim
```
chmod +x stchaind
```

## Altta ki komutu girdikten sonra görselde ki gibi çıktı alıyoruz.
```
ls
```

### Sizde sadece schaind yazması yeterli, sui ve stratos-chain yazmayacak.

![image](https://user-images.githubusercontent.com/101149671/181049915-ed70387d-5f13-480e-9dfe-968a00426923.png)

## Go kurulumu 

* go version yazdığınızda 1.16+ üstü sürüm olmalı, genelde 1.18 çıkar.
```
wget -O go1.18.2.linux-amd64.tar.gz https://golang.org/dl/go1.18.2.linux-amd64.tar.gz 
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.2.linux-amd64.tar.gz && rm go1.18.2.linux-amd64.tar.gz 
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile 
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile 
echo 'export GO111MODULE=on' >> $HOME/.bash_profile 
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile 
go version 
go mod tidy
```
## Source code ile binary dosyayı derliyoruz. 

```
git clone https://github.com/stratosnet/stratos-chain.git
cd stratos-chain
git checkout v0.8.0
make build
```
#### :point_right: Hata alırsanız 

* Hata almazsanız bunları girmenıze gerek yok. (yüksek ihtimal hata almayacaksınız)
```
go mod tidy
sudo apt update
make build
```

## Binary dosyalarını $GOPATH/bin dizinine yüklüyoruz:
```
cd ~/stratos-chain
make install
```

## initialize işlemi yapıyoruz. 

* :point_right: Node Name kısmını kendi validator isminizi girin :point_left:

```
cd $HOME
./stchaind init NodeName
```

## genesis.json ve config.toml dosyalarını indiriyoruz:
```
wget https://raw.githubusercontent.com/stratosnet/stratos-chain-testnet/main/genesis.json
wget https://raw.githubusercontent.com/stratosnet/stratos-chain-testnet/main/config.toml
```

## addrbook.json dosyasını indiriyoruz:
```
wget -O $HOME/.stchaind/config/addrbook.json "https://github.com/mmc6185/node-testnets/blob/main/stratos/stratos-tropos-4/addrbook.json?raw=true"
```

## genesis.json ve config.toml dosyalarını .stchaind/config/ dizini altına taşıma işlemi:
```
mv config.toml $HOME/.stchaind/config/
mv genesis.json  $HOME/.stchaind/config/
```

## servis dosyası :
```
echo "[Unit]
Description=Stratos Node
After=network.target
[Service]
User=$USER
Type=simple
ExecStart=$(which stchaind) start
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target" > $HOME/stratosd.service
sudo mv $HOME/stratosd.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```

## servisimizi aktifleştiriyoruz:
```
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable stratosd
sudo systemctl restart stratosd
```

## Node'umuzun loglarına bakıyoruz.

* Görselde ki gibi loglar akıcak, 1-2-3 diye başlıyacak sizde.

```
journalctl -u stratosd -f
```
![image](https://user-images.githubusercontent.com/101149671/181054052-5f6415e1-b513-4429-86b3-c5cc83b77807.png)

## Şimdi explorere gidiyoruz: [explorer linki](https://big-dipper-tropos.thestratos.org/)

* Sol üstte Latest Block yazıcak, güncel blok orası.
* Birazdan false çıktısı komutunu gireceğiz
* False çıktınız true yazacak, false olması için güncel bloğa gelmesi gerekiyor, örnek:
* Eşleşmesi max 30-60 dakika arası sürer.

```
stchaind status 2>&1 | jq .SyncInfo
```

![image](https://user-images.githubusercontent.com/101149671/181054593-3a2eab44-1aa5-4efa-917f-d161e87130c0.png)

## Şimdi o eşleşirken biz cüzdan oluşturalım:

* WalletName kısmını kendı cüzdan adınız yapın!
* Çıkan bilgileri not edin en altta 12 kelımenız olacak o da dahil.

```
stchaind keys add --hd-path "m/44'/606'/0'/0/0" --keyring-backend test  WalletName
```

## Faucetten token alalım

* WalletAdres kısmını kaldırın ve cüzdan adresinizi yazın, tırnakları kaldırmayın!

```
curl --header "Content-Type: application/json" --request POST --data '{"denom":"ustos","address":"walletAddress"} ' https://faucet-tropos.thestratos.org/credit
```

## Bakalım tokenler gelmişmi 

* st1400.. kısmına kendi cüzdan adresinizi yazın

```
stchaind query bank balances st1400f6e4kes5sk0ltfz8ms74ga9wzd9dulchh5q
```

![image](https://user-images.githubusercontent.com/101149671/181055881-b35c2aa7-d751-44ab-aaed-9a92381a2e62.png)

## Validator oluşturma

* NodeName kısmına validator ismimizi girin!
* WalletAddres kısmına cüzdan adınızı girin!

```
stchaind tx staking create-validator \
--amount=100000000ustos \
--pubkey=$(stchaind tendermint show-validator) \
--moniker="NodeName" \
--chain-id=tropos-4  --keyring-backend=test --gas=auto -y \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=walletAddress \
--gas=auto -y
```

## Explorerde kendimizi kontrol edelim:

![image](https://user-images.githubusercontent.com/101149671/181056792-6b9c4fcf-df8d-4d13-94dd-4ae37b882550.png)

# Stratos Türkiye Telegram grubu: [Stratos Türkiye](https://t.me/StratosTurkish)
