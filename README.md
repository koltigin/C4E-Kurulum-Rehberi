# C4E Kurulum Rehberi

![C4E-GitHub](https://github.com/koltigin/C4E-Kurulum-Rehberi/assets/102043225/7037175e-f6bf-4f9f-a298-dd534c89f8fd)

## Bağlantılar
 ✔️ [Website](https://c4e.io/)<br>
 ✔️ [Blockchain Explorer](https://explorer-testnet.c4e.io/)<br>
 ✔️ [Doküman](https://docs.c4e.io/validatorsGuide/Starttestnet/validator-setup.html#)<br>
 ✔️ [GitHub](https://github.com/chain4energy)<br>
 ✔️ [Discord](https://discord.gg/chain4energy)<br>
 ✔️ [Zealy](https://zealy.io/c/c4e/invite/KiZnxeaimKSv6lTcNAILI)

## Gereksinimler 
| Bileşenler | Minimum Gereksinimler | **Tavsiye Edilen Gereksinimler** | 
| ------------ | ------------ | ------------ |
| CPU |	4 | 4 |
| RAM	| 16 GB | 16 GB |
| Storage	| 120 GB SSD | 250 GB SSD |

## Sistemi Güncelleme
```shell
apt update && apt upgrade -y
```

## Gerekli Kütüphanelerin Kurulması
```shell
apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen gcc lz4 -y < "/dev/null"
```

## Go Kurulumu
```shell
ver="1.20.3"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
rm -rf /usr/local/go
tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm -rf "go$ver.linux-amd64.tar.gz"
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

## Değişkenleri Yükleme
aşağıda değiştirmeniz gereken yerleri yazıyorum.
* `$C4E_NODENAME` validator adınız
* `$C4E_WALLET` cüzdan adınız
*  Eğer portu başka bir node kullanıyorsa aşağıdan değiştirebilirsiniz.
```shell
echo "export C4E_NODENAME=$C4E_NODENAME"  >> $HOME/.bash_profile
echo "export C4E_WALLET=$C4E_WALLET" >> $HOME/.bash_profile
echo "export C4E_PORT=11" >> $HOME/.bash_profile
echo "export C4E_CHAIN_ID=babajaga-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Örnek
Node ve Cüzdan adımızın `Mehmet` olduğunu varsayalım. Kod aşağıdaki şekilde düzenlenecektir. 
```shell
echo "export C4E_NODENAME=Mehmet"  >> $HOME/.bash_profile
echo "export C4E_WALLET=Mehmet" >> $HOME/.bash_profile
echo "export C4E_PORT=11" >> $HOME/.bash_profile
echo "export C4E_CHAIN_ID=babajaga-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## C4E'nin Kurulması

```shell
cd $HOME
git clone https://github.com/chain4energy/c4e-chain.git
cd c4e-chain
git checkout tags/v1.3.0
make install
c4ed version
```
Versiyon çıktısı `1.3.0` olacak.



## Uygulamayı Yapılandırma ve Başlatma
```shell
c4ed config keyring-backend test
c4ed config chain-id $C4E_CHAIN_ID
c4ed init --chain-id $C4E_CHAIN_ID $C4E_NODENAME
```

## Genesis ve addrbook Dosyasının Kopyalanması
```shell
wget https://raw.githubusercontent.com/chain4energy/c4e-chains/main/babajaga-1/genesis.json -O $HOME/.c4e-chain/config/genesis.json
wget https://raw.githubusercontent.com/koltigin/C4E-Kurulum-Rehberi/main/addrbook.json -O $HOME/.c4e-chain/config/addrbook.json
```

## Minimum GAS Ücretinin Ayarlanması
```shell
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0uc4e"|g' $HOME/.c4e-chain/config/app.toml
```

## Indexer'i Kapatma (Opsiyonel)
```shell
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.c4e-chain/config/config.toml
```

## SEED ve PEERS Ayarlanması
```shell
peers="de18fc6b4a5a76bd30f65ebb28f880095b5dd58b@66.70.177.76:36656,33f90a0ac7e8f48305ea7e64610b789bbbb33224@151.80.19.186:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
```

## Prometheus'u Aktif Etme
```shell
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.c4e-chain/config/config.toml
```

## Pruning'i Ayarlama
```shell
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.c4e-chain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.c4e-chain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.c4e-chain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.c4e-chain/config/app.toml
```

## Portları Ayarlama
```shell
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${C4E_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${C4E_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${C4E_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${C4E_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${C4E_PORT}660\"%" $HOME/.c4e-chain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${C4E_PORT}317\"%; s%^address = \":8080\"%address = \":${C4E_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${C4E_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${C4E_PORT}091\"%" $HOME/.c4e-chain/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${C4E_PORT}657\"%" $HOME/.c4e-chain/config/client.toml
```

External Adres Ekleme
```shell
PUB_IP=`curl -s -4 icanhazip.com`
sed -e "s|external_address = \".*\"|external_address = \"$PUB_IP:${C4E_PORT}656\"|g" ~/.c4e-chain/config/config.toml > ~/.c4e-chain/config/config.toml.tmp
mv ~/.c4e-chain/config/config.toml.tmp  ~/.c4e-chain/config/config.toml
```

## Servis Dosyası Oluşturma
```shell
tee /etc/systemd/system/c4ed.service > /dev/null << EOF
[Unit]
Description=C4E Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which c4ed) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
## Servisi Aktive Etme
```shell
systemctl daemon-reload
systemctl enable c4ed
```

## StateSync Yükleme ([Stake Town](https://services.stake-town.com/home/testnet/c4e/sync#snapshot))
```shell
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
c4ed tendermint unsafe-reset-all --home $HOME/.c4e-chain --keep-addr-book

SNAP_RPC="https://c4e-testnet-rpc.stake-town.com:443"

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

PEERS="94a46ce2a5c5b835b84e121676847c5ee4eabf3f@65.109.65.248:35656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.c4e-chain/config/config.toml

CONFIG_TOML=$HOME/.c4e-chain/config/config.toml
sed -i 's|^enable *=.*|enable = true|' $CONFIG_TOML
sed -i 's|^rpc_servers *=.*|rpc_servers = "'$SNAP_RPC,$SNAP_RPC'"|' $CONFIG_TOML
sed -i 's|^trust_height *=.*|trust_height = '$BLOCK_HEIGHT'|' $CONFIG_TOML
sed -i 's|^trust_hash *=.*|trust_hash = "'$TRUST_HASH'"|' $CONFIG_TOML

mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json
```

## Node Başlatma ve Logları Kontrol Etme
```shell
systemctl start c4ed
journalctl -u c4ed -f -o cat
```  

## Cüzdan Oluşturma

### Yeni Cüzdan Oluşturma
`$C4E_WALLET` bölümünü değiştirmiyoruz kurulumun başında cüzdanımıza değişkenler ile isim belirledik.
```shell 
c4ed keys add $C4E_WALLET
```  

### Var Olan Cüzdanı İçeri Aktarma
```shell
c4ed keys add $C4E_WALLET --recover
```

## Cüzdan ve Valoper Bilgileri
Burada cüzdan ve valoper bilgilerimizi değişkene ekliyoruz.

```shell
C4E_WALLET_ADDRESS=$(c4ed keys show $C4E_WALLET -a)
C4E_VALOPER_ADDRESS=$(c4ed keys show $C4E_WALLET --bech val -a)
```

```shell
echo 'export C4E_WALLET_ADDRESS='${C4E_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export C4E_VALOPER_ADDRESS='${C4E_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Faucet
https://wallet-testnet.c4e.io/faucet adresine giderek token istiyoruz. 

### Cüzdan Bakiyesine Bakma
```
c4ed query bank balances $C4E_WALLET_ADDRESS
```

🔴 **BU AŞAMADAN SONRA NODE'UMUZUN EŞLEŞMESİNİ BEKLİYORUZ.**

## Senkronizasyonu Kontrol Etme
`false` çıktısı almadıkça bir sonraki yani validator oluşturma adımına geçmiyoruz.
```shell
c4ed status 2>&1 | jq .SyncInfo
```

🔴 **Eşleşme tamamlandıysa aşağıdaki adıma geçiyoruz.**


## Validator Oluşturma
 Aşağıdaki komutta aşağıda berlirttiğim yerler dışında bir değişiklik yapmanız gerekmez;
   - `identity`  burada `XXXX1111XXXX1111` yazan yere [keybase](https://keybase.io/) sitesine üye olarak size verilen kimlik numaranızı yazıyorsunuz.
   - `details` `Always forward with the Anatolian Team 🚀` yazan yere kendiniz hakkında bilgiler yazabilirsiniz.
   - `website`  `https://anatolianteam.com` yazan yere varsa bir siteniz ya da twitter vb. adresinizi yazabilirsiniz.
   - `security-contact`  E-posta adresiniz.
 ```shell 
c4ed tx staking create-validator \
--amount=1000000uc4e \
--pubkey=$(c4ed tendermint show-validator) \
--moniker=$C4E_NODENAME \
--chain-id=$C4E_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.05 \
--min-self-delegation="1" \
--gas-prices=0.1uc4e \
--gas-adjustment=1.5 \
--gas=auto \
--from=$C4E_WALLET \
--details="Always forward with the Anatolian Team 🚀" \
--security-contact="xxxxxxx@gmail.com" \
--website="https://anatolianteam.com" \
--identity="XXXX1111XXXX1111" \
--yes
```

## YARARLI KOMUTLAR

## Logları Kontrol Etme 
```
journalctl -fu c4ed -o cat
```

### Sistemi Başlatma

```
systemctl start c4ed
```

### Sistemi Durdurma
```
systemctl stop c4ed
```

### Sistemi Yeniden Başlatma
```
systemctl restart c4ed
```

### Node Senkronizasyon Durumu
```
c4ed status 2>&1 | jq .SyncInfo
```
```
curl -s localhost:26657/status | jq .result.sync_info
```

### Validator Bilgileri
```
c4ed status 2>&1 | jq .ValidatorInfo
```

### Node Bilgileri

```
c4ed status 2>&1 | jq .NodeInfo
```

### Node ID Öğrenme

```
c4ed tendermint show-node-id
```


### Node IP Adresini Öğrenme

```
curl icanhazip.com
```

### Cüzdanların Listesine Bakma

```
c4ed keys list
```

### Cüzdan Adresini Görme

```
c4ed keys show $C4E_WALLET --bech val -a
```

### Cüzdanı İçeri Aktarma

```
c4ed keys add $C4E_WALLET --recover
```

### Cüzdanı Silme

```
c4ed keys delete $C4E_WALLET
```

### Cüzdan Bakiyesine Bakma
```
c4ed query bank balances $C4E_WALLET_ADDRESS
```

### Bir Cüzdandan Diğer Bir Cüzdana Transfer Yapma

```
c4ed tx bank send $C4E_WALLET_ADDRESS GONDERILECEK_CUZDAN_ADRESI 100000000uc4e
```

### Proposal Oylamasına Katılma
```
c4ed tx gov vote 1 yes --from $C4E_WALLET --chain-id=$C4E_CHAIN_ID
```


### Validatore Stake Etme / Delegate Etme

```
c4ed tx staking delegate $VALOPER_ADDRESS 100000000uc4e --from=$C4E_WALLET --chain-id=$C4E_CHAIN_ID --gas=auto --fees 5000uc4e
```

### Mevcut Validatorden Diğer Validatore Stake Etme / Redelegate Etme
<srcValidatorAddress>: Mevcut Stake edilen validatorün adresi
<destValidatorAddress>: Yeni stake edilecek validatorün adresi 
```
c4ed tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 100000000uc4e --from=$C4E_WALLET --chain-id=$C4E_CHAIN_ID --gas=auto
```

### Ödülleri Çekme

```
c4ed tx distribution withdraw-all-rewards --from=$C4E_WALLET --chain-id=$C4E_CHAIN_ID --gas=auto
```

### Komisyon Ödüllerini Çekme

```
c4ed tx distribution withdraw-rewards $VALOPER_ADDRESS --from=$C4E_WALLET --commission --chain-id=$C4E_CHAIN_ID
```

### Validator İsmini Değiştirme
NEWNODENAME yazan yere yeni validator/moniker isminizi yazınız. TR karakçer içermemelidir.

```
c4ed tx staking edit-validator \
--moniker=NEWNODENAME \
--chain-id=$C4E_CHAIN_ID \
--from=$C4E_WALLET
```

### Validator Komisyon Oranını Degiştirme
```
c4ed tx staking edit-validator --commission-rate "0.02" --moniker=$C4E_NODENAME --chain-id=$C4E_CHAIN_ID --from=$C4E_WALLET
```

### Validator Bilgilerinizi Düzenleme
Bu bilgileri değiştirmeden önce https://keybase.io/ adresine kayıt olarak aşağıdaki kodda görüldüğü gibi 16 haneli (XXXX0000XXXX0000) kodunuzu almalısınız. Ayrıca profil resmi vs. ayarları da yapabilirsiniz. 
$NODENAME: Yeni node adınız (moniker)
$C4E_WALLET: Cüzdan adınız, değiştirmeniz gerekmez. Değişkenlere ekledik çünkü.
```
c4ed tx staking edit-validator \
--moniker=$NODENAME \
--identity=XXXX0000XXXX0000 \
--website="VARSA WEBSITENIZI YAZABILIRSINIZ" \
--details="BU BOLUME KENDINIZI TANITAN BIR CUMLE YAZABILIRSINIZ" \
--chain-id=$C4E_CHAIN_ID \
--from=$C4E_WALLET
```

### Validatoru Jail Durumundan Kurtarma 

```
c4ed tx slashing unjail --from $C4E_WALLET --chain-id $C4E_CHAIN_ID --gas-prices 0.00001uc4e --gas-adjustment 1.5 --gas auto -y
```

### Node'u Tamamen Silme 

```
systemctl stop c4ed && \
systemctl disable c4ed && \
rm /etc/systemd/system/c4ed.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .c4e-chain c4e-chain && \
rm -rf $(which c4ed)
sed -i '/C4E_/d' ~/.bash_profile
```


# Hesaplar:

[Anatolian Team](https://anatolianteam.com)

[Twitter](https://twitter.commehmetkoltigin)

[Medium](https://medium.com/@mehmetkoltigin)

[YouTube](https://www.youtube.com/channel/UCmLgaftx5e38BE0E7gpY2dA)

[Discord](https://discordapp.com/users/837933958280904737)

[Telegram](https://t.me/mehmetkoltigin)
