# decentr
![img](https://img.shields.io/docker/cloud/build/decentr/decentr.svg)

Decentr blockchain


## Testnet Full Node Quick Start
This assumes that you're running Linux or MacOS and have installed [Go 1.15+](https://golang.org/dl/).  This guide helps you:

* build and install Decentr
* allow you to name your node
* add seeds to your config file
* download genesis state
* start your node
* use decentrdcli to check the status of your node.

Build, Install, and Name your Node:

```bash
# Clone Decentr from the latest release found here: https://github.com/Decentr-net/decentr/releases
git clone -b v0.6.1 https://github.com/Decentr-net/decentr
# Enter the folder Decentr was cloned into
cd decentr
# Compile and install Decentr
make install
# Initialize decentrd in ~/.decentrd and name your node
decentrd init yournodenamehere
```

Add Seeds:

```bash
# Edit config.toml
nano ~/.decentrd/config/config.toml
```

Scroll down to `seeds` in `config.toml`, and add some of these seeds as a comma-separated list:

```c
fba90c20ade62a2c9564a03eff93c0603ccdf238@ares.testnet.decentr.xyz:26656
9479eef715892a18249350fcb1eeca0efc4c9354@hera.testnet.decentr.xyz:26656
d44010452fec5cb0b9b314aa089f1082fbaed185@hermes.testnet.decentr.xyz:26656
28d55428c88a870c7c1737a31369f1c43595e4cf@poseidon.testnet.decentr.xyz:26656
8e14add2e93231a41fb3736a12203ee22ad0069c@zeus.testnet.decentr.xyz:26656
```

Download Genesis, Start your Node, Check your Node Status:

```bash
# Download genesis.json
wget -O $HOME/.decentrd/config/genesis.json https://raw.githubusercontent.com/Decentr-net/testnets/master/1.0/genesis.json
# Start Decentrd
decentrd start --cerberus-addr https://cerberus.testnet.decentr.xyz --community-moderator-addr decentr1nt5k6eg9zq5t2v66pr6zgyt5hh5tu8sk30re3a
# Check your node's status with decentrcli
decentrcli status
```

Welcome to the Decentr!

To start LCD (light-client daemon), a local REST server
```bash
decentrcli rest-server
# > I[2020-07-31|13:50:22.088] Starting application REST service (chain-id: "testnet")... module=rest-server 
# > I[2020-07-31|13:50:22.088] Starting RPC HTTP server on 127.0.0.1:1317   module=rest-server 
``` 
The server is available at `127.0.0.1:1317`

### CLI
```bash
decentrcli help
decentrcli config chain-id testnet
decentrcli config keyring-backend test 

decentrcli keys add megaherz
# > 
# {
#   "name": "megaherz",
#   "type": "local",
#   "address": "decentr1m8k9dy3962v8km0d5jwsqanwvf0h5fmj6f5zyp",
#   "pubkey": "decentrpub1addwnpepq2yrdqzcnleu2gr69c5zkw7laa4el7mtj8ala97s648wzlvegk7vcpsh6kg",
#   "mnemonic": "crouch goddess pass cigar conduct odor beach coil hole enroll fringe crane witness squeeze mention pioneer inmate wink concert laugh segment abuse tomorrow amused"
#  }
```

### REST transactions
If you want to use REST to create tx, you should get it from rest service, then sign and broadcast it.

#### Example
Get tx body
```bash
curl -XPOST -s http://localhost:1317/profile/public/$(decentrcli keys show jack -a) \
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"},"public": { "firstName": "foo","lastName": "bar","avatar": "https://avatars3.githubusercontent.com/u/1526177","gender": "female","birthday": "2001-02-01"} }' > unsignedTx.json
```
  
unsignedTx.json will contain
```json
{"type":"cosmos-sdk/StdTx","value":{"msg":[{"type":"profile/SetPublic","value":{"owner":"decentr1z4z94y4lf33tdk4qvwh237ly8ngyjv5my6xqrw","public":{ "firstName": "foo","lastName": "bar","avatar": "https://avatars3.githubusercontent.com/u/1526177","gender": "female","birthday": "2001-02-01"} }}],"fee":{"amount":[],"gas":"200000"},"signatures":null,"memo":""}}
```
  
Then sign this transaction
```bash
decentrcli tx sign unsignedTx.json --from jack --chain-id testnet > signedTx.json
```
  
And finally broadcast the signed transaction
```bash
decentrcli tx broadcast signedTx.json
```

## PDV (Personal Data Value) Data

#### CLI
```bash
# Query pdv owner by its address
decentrcli query pdv owner <address>

# Query pdv full
decentrcli query pdv show <address>

# List account's pdv
decentrcli query pdv list <owner> [from] [limit]

# Get cerberus address
decentrcli query pdv cerberus

# Create pdv
decentrcli tx pdv create [type] [pdv] --from [account]
```

#### REST
```bash
# Query pdv owner by its address
curl -s http://localhost:1317/pdv/{address}/owner

# Query pdv full
curl -s http://localhost:1317/pdv/{address}/show

# List account's pdv
curl -s http://localhost:1317/pdv/{owner}/list?from={from}&limit={limit}

# Get cerberus address
curl -s http://localhost:1317/pdv/cerberus-addr

# Create PDV
curl -XPOST -s http://localhost:1317/pdv \ 
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"}, "type": 2, "address": "address from cerberus"}' > unsignedTx.json
```

## PDV Token
PDV tokens are assigned to the user as soon as they reveal their personal data. 
There are no transactions, only query to get PDV token balance of the specific user.

#### CLI
```bash
# Query pdv token balance
decentrcli query token balance [address]

# Query pdv token stats
decentrcli query token stats [address]
```

#### REST
```bash
# Query pdv token balance
curl -s http://localhost:1317/token/balance/{address}

# Query pdv token stats
curl -s http://localhost:1317/token/stats/{address}
```

### Profile
User profile consists of two parts: private and public. Private data is encrypted with user's private key.
Public one includes first name, last name, avatar, gender and birthday.

#### CLI
```bash
# Query private profile. Returns base64 encode string.
decentrcli query profile private [address]

# Query public profile
decentrcli query profile public [address] 

# Set private profile data that you own. The data should be encrypted with your private key beforehead.
decentrcli tx profile set-private [data] --from [account]

# Set public profile data that you own. Public profile are attributes: gender, birth date.
# Birthday date format is yyyy-mm-dd. Gender: male, female
decentrcli tx profile set-public '{ "firstName": "foo", "lastName": "bar", "avatar": "https://avatars3.githubusercontent.com/u/1526177", "gender": "female", "birthday": "2001-02-01"}' --from [account]
```

#### REST
To execute REST command decentrcli has to be run as a REST server `decentrcli rest-server` 

```bash
### Get account info
curl -s http://localhost:1317/auth/accounts/$(decentrcli keys show jack -a)
# > {"value": { "address": "decentr1d7narytgsy5lj2agt0t8sntaq3p8ucjhermqjj","coins": [], "public_key": "decentrpub1addwnpepq2jqxxu853rh0pa0agnkaxwaz6qdz6kpd4esqpw33sz3mp3a6mwh5eejl8q", "account_number": 3,"sequence": 6 }}

# Query private profile. Returns base64 encode string.
curl -s http://localhost:1317/profile/private/$(decentrcli keys show jack -a)
# > {"height": "0", "result": "YldWbllXaGxjbm9L"}

# Query public profile.
curl -s http://localhost:1317/profile/public/$(decentrcli keys show jack -a)
# > { "height": "0", "result": { "firstName": "foo", "lastName": "bar", "avatar": "https://avatars3.githubusercontent.com/u/1526177", "gender": "female", "birthday": "2001-02-01", "registeredAt:"1607972947"}}

# Set private profile
curl -XPOST -s http://localhost:1317/profile/private/$(decentrcli keys show jack -a) \ 
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"},"private": "YldWbllXaGxjbm9L"}' > unsignedTx.json

# Set public profile
curl -XPOST -s http://localhost:1317/profile/public/$(decentrcli keys show jack -a) \
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"},"public": { "firstName": "foo","lastName": "bar","avatar": "https://avatars3.githubusercontent.com/u/1526177","gender": "female","birthday": "2001-02-01"} }' > unsignedTx.json
```
## Bank

#### CLI
```bash
# Create and sign a send tx
decentrcli tx send [from_key_or_address] [to_address] [amount]
```

#### REST
```bash
# Get balance
curl http://localhost:1317/bank/balances/{address} 

# Transfer coins (send coins to a address)
curl -X POST http://localhost:1317/bank/accounts/{address}/transfers \ 
    -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"}, "amount": [{"denom": "udec", "amount": "15"}]}' > unsignedTx.json
```

## Community

### Categories
| Value | Description |
| --- | --- |
| 1 | World News |
| 2 | Travel & Tourism |
| 3 | Science & Technology |
| 4 | Strange World |
| 5 | Arts & Entertainment |
| 6 | Writers& Writing |
| 7 | Health & Fitness |
| 8 | Crypto & Blockchain |
| 9 | Sports |


### Likes weight
|Value | Description |
| --- | --- |
| 1 | Like |
| 0 | Delete |
| -1 | Dislike |

#### CLI
```bash
# Create post
decentrcli tx community create-post [text] --title [title] --preview-image [url to preview] --category 2 --from [account]

# Delete post
decentrcli tx community delete-post [postOwner] [postUUID] --from [account]

# Like post
decentrcli tx community like-post [postOwner] [postUUID] --weight [weight] --from [account]

# Get moderator account address
decentrcli query community moderator-addr

# Get user's posts
decentrcli query community user-posts <account> [--from-uuid uuid] [--limit int]

# Get a single post
decentrcli query community post <owner> <uuid>

# Get the recent posts
decentrcli query community posts [--from-owner account --from-uuid uuid] [--category int] [--limit int]

# Get the most popular posts
decentrcli query community popular-posts [--from-owner account --from-uuid uuid] [--category int] [--limit int] [--interval day/week/month]

# Get user's likes
decentrcli query community user-liked-posts [owner]
```

#### REST
```bash
# Create post, note category is a quoted number.
curl -XPOST -s http://localhost:1317/community/posts \
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"},"text": "my brand new text", "title":"my first post title", "imagePreview": "https://localhost/mypicture.jpg", "category": "2"}' > unsignedTx.json

# Delete post
curl -XPOST -s http://localhost:1317/community/posts/{postOwner}/{postUUID}/delete \
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"}}' > unsignedTx.json

# Get moderator account address
curl -s http://localhost:1317/community/moderator-addr

# Like post
curl -XPOST -s http://localhost:1317/community/posts/{postOwner}/{postUUID}/like\
     -d '{"base_req":{"chain_id":"testnet", "from": "'$(decentrcli keys show jack -a)'"}, "weight": 1}' > unsignedTx.json

# Get a single post
curl -s "http://localhost:1317/community/post/{owner}/{uuid}"

# Get the latest posts
curl -s "http://localhost:1317/community/posts?category={category}&limit={limit}&fromOwner={account}&fromUUID={post's uuid}"

# Get the most popular posts by day
curl -s "http://localhost:1317/community/posts/popular/byDay?category={category}&limit={limit}&fromOwner={account}&fromUUID={post's uuid}"

# Get the most popular posts by week
curl -s "http://localhost:1317/community/posts/popular/byWeek?category={category}&limit={limit}&fromOwner={account}&fromUUID={post's uuid}"

# Get the most popular posts by month
curl -s "http://localhost:1317/community/posts/popular/byMonth?category={category}&limit={limit}&fromOwner={account}&fromUUID={post's uuid}"

# Get user's posts
curl -s "http://localhost:1317/community/posts/{account}?from={postUUID}&limit={limit}"

# Get user's likes
curl -s "http://localhost:1317/community/likedPosts/{account}"
```

## Build
```bash
make install
```
creates two binaries: decentrd (node) and decentrcli (cli)

#### Build local image image
```
docker build -t decentr-local -f scripts/Dockerfile .
```
#### Start local testnet
```
cd scripts/test && docker-compose up
```

## Follow us!
Your data is value. Decentr makes your data payable and tradeable online.
* [Medium](https://medium.com/@DecentrNet)
* [Twitter](https://twitter.com/DecentrNet)
* [Telegram](https://t.me/DecentrNet)
