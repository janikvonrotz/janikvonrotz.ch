---
id: 4739
title: Create a soft wallet and transfer your Ether coins from an exchange
date: 2018-01-10T14:58:27+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4739
permalink: /2018/01/10/create-a-soft-wallet-and-transfer-your-ether-coins-from-an-exchange/
specific_page_layout:
  - default-sidebar
image: /wp-content/uploads/2018/01/ethereum-logo.png
categories:
  - Ethereum
tags:
  - block
  - blockchain
  - encryption
  - ether
  - ethereum
  - exchange
  - explorer
  - geth
  - kraken
  - parity
  - platform
  - soft wallet
  - transaction
  - wallet
---
Intrigued by the title you might ask your self: What is the reason to store ether in a software wallet? Well, If you have cryptocoins on an exchange platform there is always the risk of the account getting hacked or the platform goes offline (see MtGox). Exchange platforms for cryptocoins are not as regulated and institutionalized as banks and trading centers are. The risk is in favor of the provider. To assert full control of your coins aka your money it is recommended to store them in a wallet.
<!--more-->

In the following tutorial I will show you how I have transferred my coins into a soft wallet.

This tutorial assumes that the os environment is MacOS. Nonetheless, the platform does not matter to go through the tutorial, it also works for other platforms such as Windows or Linux.

## Install Geth

Lets get started by installing the go command line tool to interact with the Ethereum network.

Use Homebrew to install the tool, make sure it is up-to-date.

    brew update
    brew upgrade

Install the **g**o **eth**erum cli.

    brew tap ethereum/ethereum
    brew install ethereum

For other platforms use the according package manager.

## Account Management

To receive Ether we need an account.

Create an account and set a password.

    geth account new

Make sure to store the password somewhere safe. There is no way to retrieve or reset the password for the account once lost.

List all available accounts.

    geth account list

Backup the keystore file of the new account.

    cp ~/Library/Ethereum/keystore/UTC--yyyy-mm-ddT12-59-07.353801000Z--927b07ac62ee6c10861b5024710a997937b20e31 /path/to/encrypted/storage/Etherum/_ADDRESS_.prv

## Sync

Transactions are stored on the block chain. In order to see transactions and execute them you must sync the blockchain.

Sync the Etherum blockchain in fast mode.

    geth --syncmode fast --cache 2048 

This mode allows us to sync the blockchain without processing and verifying each block.

The second parameter sets the cache according to the computers available memory. Make sure to adjust the number.

To see the progress of the sync an attached geth console must be started. An attached geth console is connected to the currently running geth process. Open a new terminal and run the command below.

    geth attached

Now check the syncing state.

    eth.syncing

As long as the current and highest block number do not match, further syncing is required.

## Send Ether

Once the sync is finished, we are ready to retrieve Ether.

From your exchange platform (I am using Kraken) send the fewest  amount of Ether possible to the accounts address. **Do not send all of your Ether.** We have to make sure everything works correctly.

To see the transaction a sync of the chain is required.

To check the accounts balance start the geth console.

    geth console

Now run this JavaScript code.

    function checkAllBalances() {
        var totalBal = 0;
        for (var acctNum in eth.accounts) {
            var acct = eth.accounts[acctNum];
            var acctBal = web3.fromWei(eth.getBalance(acct), "ether");
            totalBal += parseFloat(acctBal);
            console.log("  eth.accounts[" + acctNum + "]: \t" + acct + " \tbalance: " + acctBal + " ether");
        }
        console.log("  Total balance: " + totalBal + " ether");
    };
    checkAllBalances();

The JavaScript function return the Ether balance for each registered account.

If you see a positive Ether balance for your account then everything works fine and you are ready to send more Ether to the soft wallet.

## Secure Storage

In case you want to know how I store my password and key files.

I am using the [keybase file system](https://keybase.io/) to store the key file of the geth account and [KeePass](https://keepassxc.org/) to store the password. The KeePass database is encrypted with a password and a key file. All data can be accessed from my Windows and from my Mac computer. These tools are all open source. Further the hard disk of my Windows and Mac computer are encrypted.

## Geth Alternative

In case geth did not work you, try [parity](https://www.parity.io/).

## Source

[Etherum Wiki - Managing your accounts](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts)  
[Etherscan - Blockchain Explorer](https://etherscan.io)  
