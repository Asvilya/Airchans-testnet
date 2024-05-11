<!-- wp:paragraph -->
<p>System Ubuntu 22.04</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<figure class="wp-block-table"><table><thead><tr><th>CPU</th><th>RAM</th><th>SSD</th></tr></thead><tbody><tr><td>4 vCPU</td><td>8 GB RAM</td><td>160 SSD</td></tr></tbody></table></figure>
<!-- /wp:table -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="install-dependencies-1"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#install-dependencies-1"></a>Install dependencies</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p><strong>UPDATE SYSTEM </strong></p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>sudo apt update &amp;&amp; sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make lz4 unzip ncdu -y</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p><strong>INSTALL GO 1.21.5</strong></p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>ver="1.21.5" 
cd $HOME 
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 

sudo rm -rf /usr/local/go 
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
rm "go$ver.linux-amd64.tar.gz"

echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" &gt;&gt; $HOME/.bash_profile
source $HOME/.bash_profile    </code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="download-and-build-binaries"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#download-and-build-binaries"></a>Download and build binaries</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x junctiond
sudo mv junctiond /usr/local/bin</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="set-vars"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#set-vars"></a>Set Vars</h4>
<!-- /wp:heading -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><!-- wp:paragraph -->
<p><code>Moniker</code> yerine validator adınızı ekliyoruz.</p>
<!-- /wp:paragraph --></blockquote>
<!-- /wp:quote -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond config chain-id junction
junctiond init "Moniker" --chain-id junction</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="config-init-app"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#config-init-app"></a>Config init app</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>wardend init $MONIKER
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${WARDEN_PORT}657\"|" $HOME/.warden/config/client.toml</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="download-genesis-and-addrbook"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#download-genesis-and-addrbook"></a>Download Genesis and Addrbook</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>sudo wget -O $HOME/.junction/config/genesis.json https://raw.githubusercontent.com/CoinHuntersTR/props/main/Airchains/genesis.json
sudo wget -O $HOME/.junction/config/addrbook.json https://raw.githubusercontent.com/CoinHuntersTR/props/main/Airchains/addrbook.json

sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"5000amf\"/;" ~/.junction/config/app.toml</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="config-pruning"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#config-pruning"></a>Config Pruning</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>pruning="custom" &amp;&amp; \
pruning_keep_recent="100" &amp;&amp; \
pruning_keep_every="0" &amp;&amp; \
pruning_interval="10" &amp;&amp; \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.junction/config/app.toml &amp;&amp; \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.junction/config/app.toml &amp;&amp; \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.junction/config/app.toml &amp;&amp; \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.junction/config/app.toml</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="set-seeds-and-peers"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#set-seeds-and-peers"></a>Set seeds and peers</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>PEERS=$(curl -sS https://junction-rpc.dymion.cloud/net_info | \
jq -r '.result.peers&#91;] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | \
awk -F ':' '{printf "%s:%s%s", $1, $(NF), NR==NF?"":","}')
echo "$PEERS"

sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.junction/config/config.toml</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="snapshot"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#snapshot"></a>Snapshot</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond tendermint unsafe-reset-all --home ~/.junction/ --keep-addr-book
curl https://files.dymion.cloud/junction/data.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.junction</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="create-service-file"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#create-service-file"></a>create service file</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>sudo tee /etc/systemd/system/junctiond.service &gt; /dev/null &lt;&lt;EOF
&#91;Unit]
Description=junction
After=network-online.target
&#91;Service]
User=$USER
ExecStart=$(which junctiond) start
Restart=always
RestartSec=3
LimitNOFILE=65535
&#91;Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable junctiond</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":4} -->
<h4 class="wp-block-heading" id="enable-and-start-service"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#enable-and-start-service"></a>enable and start service</h4>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>sudo systemctl daemon-reload
sudo systemctl enable junctiond
sudo systemctl restart junctiond &amp;&amp; sudo journalctl -u junctiond -f</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Log<a href="https://github.com/Core-Node-Team/Testnet-TR/tree/main/Airchains#log"></a></h3>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>sudo journalctl -u junctiond -f --no-hostname -o cat
</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Cüzdan olusturma<a href="https://github.com/Core-Node-Team/Testnet-TR/tree/main/Airchains#c%C3%BCzdan-olusturma"></a></h3>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond keys add cüzdan-adi-yaz
</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Cüzdan import<a href="https://github.com/Core-Node-Team/Testnet-TR/tree/main/Airchains#c%C3%BCzdan-import"></a></h3>
<!-- /wp:heading -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond keys add cüzdan-adi-yaz --recover
</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading">Validator oluştur<a href="https://github.com/Core-Node-Team/Testnet-TR/tree/main/Airchains#validator-olu%C5%9Ftur"></a></h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Not: pubkey alalım</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond comet show-validator
</code></pre>
<!-- /wp:code -->

<!-- wp:code -->
<pre class="wp-block-code"><code>nano $HOME/validator.json
</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Not: alttaki kodu düzenleyin sonra üsteki kodu yazıp düzenlediğinizi içine yapıstırın. eğer validator kurarken hata alırsanız. size önerdiği kodu tekrar içine yapıstırın</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>{
	"pubkey": &lt;validator-pub-key&gt;,
	"amount": "1000000amf",
	"moniker": "&lt;validator-name&gt;",
	"identity": "optional identity signature (ex. UPort or Keybase)",
	"website": "validator's (optional) website",
	"security": "validator's (optional) security contact email",
	"details": "validator's (optional) details",
	"commission-rate": "0.1",
	"commission-max-rate": "0.2",
	"commission-max-change-rate": "0.01",
	"min-self-delegation": "1"
}
</code></pre>
<!-- /wp:code -->

<!-- wp:code -->
<pre class="wp-block-code"><code>junctiond tx staking create-validator $HOME/validator.json --from cüzdan-adi --chain-id junction --fees 5000amf
</code></pre>
<!-- /wp:code -->

<!-- wp:heading {"level":3} -->
<h3 class="wp-block-heading" id="auto-installation"><a href="https://service.coinhunterstr.com/testnet/airchains/installation#auto-installation"></a></h3>
<!-- /wp:heading -->
