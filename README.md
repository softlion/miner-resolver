# Infos
Scripts to help Helium witness miner to contact the PoC challenger

This repo contains:

Balena scripts for:
- nebra indoor & outdoor & Rock PI
- sensecap M1

Docker scripts for:
- pisces outdoor

Note: you need full root access to your miner.

## Root access
Balena based miners:
- You will need to extract the SD card from the miner, plug it in a computer, and update a file on it.
- add your ssh key to the /config.json file

Pisces:
- ssh is already available. Use the same credentials as for the dashboard.

## Concept
This tool is checking from an Helium full miner if it got issues contacting a challenger.  
If yes, it gets the p2p adress of this challenger from the Helium api, and ping this challenger with this p2p address.  
If that is successful, this address is added to local peer table.  And the miner will be able to send the witness report to the PoC challenger.

## Installation
Choose the correct version for your miner:
- Balena: https://github.com/snip/miner-resolver/releases/latest/download/miner-resolver_balena_arm64
- Pisces: https://github.com/snip/miner-resolver/releases/latest/download/miner-resolver_pisces_arm64

From a root ssh session on the miner:
```
wget https://github.com/snip/miner-resolver/releases/latest/download/<your correct version> -O /tmp/miner-resolver
chmod +x /tmp/miner-resolver
```

On Pisces, add the user to the docker group so it does not need sudo:
```
sudo usermod -a -G docker admin
```
Logout form ssh and login again so it takes effect.

## Usage
One time:
```
/tmp/miner-resolver
```

Forever every 30mn (Do not spam the public api! You could be blacklisted. No need to test more often.):
```
while true; do /tmp/miner-resolver_arm64; sleep 1800; done
```

## Rebuilding

Install dependencies:
```
go install github.com/hpcloud/tail@latest
```
install UPX (https://upx.github.io/ or `choco install upx` on windows)

Set the enviroment variable
```
#linux / macos
env GOOS=linux GOARCH=arm64 <go build ...>

#powershell
$Env:GOOS="linux";$Env:GOARCH="arm64";
<go build ...>
```

Release build:
```
go build -o out/miner-resolver_balena -ldflags '-s -w' ./balena
upx --best --lzma out/miner-resolver_balena

go build -o out/miner-resolver_pisces -ldflags '-s -w' ./pisces
upx --best --lzma out/miner-resolver_pisces
```

Ex: copy result on pisces miner:
```
scp  .\out\miner-resolver_pisces scp://admin@<miner ip>:<ssh port>
chmod +x miner-resolver_pisces
```


## Example of success execution
```
2022-02-27 12:36:05.459 25 [warning] <0.28578.3>@miner_onion_server:send_witness:{243,37} failed to dial challenger "/p2p/112NTMRPGUYdiYMyVGwX1QWNJspjLmGwZLzJDRpQdLjNoEEMLZCS": not_found
New witness
2022-02-27 12:36:35.591 25 [warning] <0.28578.3>@miner_onion_server:send_witness:{243,37} failed to dial challenger "/p2p/112NTMRPGUYdiYMyVGwX1QWNJspjLmGwZLzJDRpQdLjNoEEMLZCS": not_found
Already existing. Count: 1
Do action with: 112NTMRPGUYdiYMyVGwX1QWNJspjLmGwZLzJDRpQdLjNoEEMLZCS
p2p addr from API: /ip4/173.176.240.170/tcp/44158
exec command: balena exec --interactive $(balena ps --filter name=^helium-miner --format "{{.ID}}") miner peer ping /ip4/173.176.240.170/tcp/44158 2>&1
exec command output: Pinged "/ip4/173.176.240.170/tcp/44158" successfully with roundtrip time: 144 ms

2022-02-27 12:37:06.186 25 [info] <0.28578.3>@miner_onion_server:send_witness:{251,37} successfully sent witness to challenger "/p2p/112NTMRPGUYdiYMyVGwX1QWNJspjLmGwZLzJDRpQdLjNoEEMLZCS" with RSSI: -92, Frequency: 867.1, SNR: -15.2
Deleting entry
```
