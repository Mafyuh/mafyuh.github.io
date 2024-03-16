+++
date = 2024-03-15T00:13:40Z
description = ""
draft = false
slug = "spl-token-cli"
title = "How to create a Solana Token (SPL) from CLI with metadata"
tags = ["Homelab"]
comments = true
+++

I wanted to create an SPL token and after looking online I couldn't find an updated guide. So I thought I would learn and share. There are much easier ways to create these tokens but they cost $ and spending more $ than needed is no fun. This guide costs as little SOL as possible as everything is transacted directly on-chain. Everything is done from the CLI.

This guide just covers the basics, the tools used are way more powerful than what I use them for, this is just creating a basic token with no taxes or locked supply or anything complex, but these tools do support those options. If you are interested in doing more I would read the proper documentation.
- https://docs.solanalabs.com/cli/install
- https://metaboss.rs/overview.html
- https://spl.solana.com/token

[NetworkChuck has a video](https://www.youtube.com/watch?v=befUVytFC80) from late 2021 on doing this, but some commands are a bit outdated, and Solana updated their entire metadata process in 2022. 

I am using an Ubuntu 22.04 VM with 60GB storage to run these commands.

- Starting balance: 0.079975 SOL
- Ending balance: 0.05731652 SOL
- Total SOL cost: 0.02265848 SOL ($4.22 on 3/15/2024)

## Installing Solana Tools

First we need to download Solana tools to our system:
```
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```
then run the export path command that is given to you: 

```
export PATH="/home/mafyuh/.local/share/solana/install/active_release/bin:$PATH"
```
Restart your terminal session.

## Creating Wallet
We will create a new SOL wallet to fund our token. To do this run:
```
solana-keygen new
```
You don't have to put a passphrase if you don't want to. I would backup your recovery seed phrase and take note of the public address. I would fund this wallet with some SOL as well at this time.

Keep note of the keypair directory for later step.

Check your SOL balance with:
```
solana balance
```

## Install Rust
We need Rust in order to create the token, to install Rust run:
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Press enter for default installation. Once completed, restart your session again.

Then we need to install some needed packages:
```
sudo apt install libudev-dev llvm libclang-dev libssl-dev pkg-config build-essential protobuf-compiler -y
```

## Install spl-token-cli
Now using Rust we are gonna install Solana's CLI tools, this will take a few minutes.

```
cargo install spl-token-cli
```
## Create Token
Creating a new token is simple, make sure your wallet is funded with SOL and just run:
```
spl-token create-token
```
Your token's address will be printed on screen. You will use this address in pretty much all the rest of the steps so keep handy. You do need to have your wallet funded first.

Now we need to create a token account for this token:
```
spl-token create-account <TOKEN_ADDRESS>
```
Example:
```
spl-token create-account 7njsg9BA1xvXX9DNpe5fERHK4zb7MbCHKZ6zsx5k3adr
```
If you get errors like:

"unable to confirm transaction. This can happen in situations such as transaction expiration and insufficient fee-payer funds"

You just need to retry a few times, it will eventually go thru but sometimes takes 3-4 runs.

## Minting Tokens

Now that you have a token and an account for the token, you can actually mint some tokens. To do this run:
```
spl-token mint <TOKEN_ADDRESS> <# of tokens> <ACCOUNT_ADDRESS>
```
Example:
```
spl-token mint 7njsg9BA1xvXX9DNpe5fERHK4zb7MbCHKZ6zsx5k3adr 1000000 CkaGbdriXVMHtzFBPtnpDjQvZ9gM9bwd8XdTTKR2Wx32
```
To see your tokens you can run:
```
spl-token accounts
```
Now you will want to send these tokens to a new address, so make a new wallet and get its pubkey, then to send these tokens run:
```
spl-token transfer --fund-recipient --allow-unfunded-recipient <TOKEN_ADDRESS> <# of tokens> <NEW_ADDRESS>
```
Example:
```
spl-token transfer --fund-recipient --allow-unfunded-recipient 7njsg9BA1xvXX9DNpe5fERHK4zb7MbCHKZ6zsx5k3adr 1000000 2DDyEt5N4y77ETWhhUmkZiympQbpjkfrt8FcMKhB1iWU
```

## Installing Metaboss
Once this completes you can install metaboss which is needed to upload metadata. Again, this takes some time:
```
cargo install metaboss
```

## Arweave/Github
While we wait on metaboss to install, we should start uploading our tokens Logo to a cloud provider, I use Arweave in this example but you can use anything really. There are also many ways to upload to arweave so this is just a friendly example thats free.

First create an account at https://akord.com/use-arweave
Upload your image to a new vault. (PNG)
Click on the information icon next to your image and copy the arweave.net URL. (Not under Share) We need this for our JSON file we will create next. 

Now you can create a json file, and in it paste the following:
```
{
  "name": "TOKEN_NAME",
  "symbol": "SYM",
  "description": "Description of token",
  "image": "https://arweave.net/image-url-from-above"
}
```
Now save this file with .json extension and upload it to Arweave just like the image. Now we need this JSON file's Arweave link. Copy it from akord and create a new json file in your Solana server's working directory. Fill in the following:
```
{
  "name": "TOKEN_NAME",
  "symbol": "SYM",
  "uri": "https://arweave.net/json-file-arweave-url"
}
```
Using the JSON file's Arweave link as the URI. Name this file whatever.json.

## Creating Metadata
First we need to update our RPC URL, to set to mainnet run:
```
solana config set --url https://api.mainnet-beta.solana.com  --keypair /home/mafyuh/.config/solana/id.json
```
Filling in your keypair directory from earlier.

Now that metaboss is installed, we just need to run 1 command to create our tokens metadata, again it may take a few tries:
```
metaboss create metadata -a <TOKEN_ADDRESS> -m whatever.json
```
You should be able to go to solscan and see your updated metadata! It should appear in the SOL wallets soon after.
## Updating Metadata

If you ever need to update your metadata, you can do so by running:
```
metaboss update uri --keypair /home/mafyuh/.config/solana/id.json --account <TOKEN_ADDRESS> --new-uri https://arweave.net/new-arweave-json-url
```

Hope this guide has helped you save some $ when creating your Solana tokens! If you appreciate the post and wanna send some tokens as thanks, my SOL wallet address is: 3RYPrKxC6BNv3XUMf8Cyjg36pw6Qu1txRvqq6LNq9Psj
