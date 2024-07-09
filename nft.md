#Introduction to XRPL

## getting started

Before anything else, you are going to want to setup your coding environment.

Any javascript editor will do and feel free to use your own node environement to follow along with javascript. Most of the code we will use can be run client side as well. 

If you want a hastle free setup, use replit (https://replit.com/). Accounts are free and then create a new replit with the typescript defaults or the node defaults if you would rather avoid typescript.

If you use another language, you can follow along using a script of your own, but you will have to use the appropriate SDK which may differ from xrpl.js at times.

You will often want to refer to the main API documentation for transactions here: https://xrpl.org/docs/references/protocol/transactions/types/


### faucets

While faucets are accessible programatically, you can also create a test account via this page: https://xrpl.org/resources/dev-tools/xrp-faucets/


### wallets

This tutorial makes use of the Xaman wallet, which you can download from your mobile phone at https://xumm.app. 
Once installed you can import a wallet via "family seed" (the address secret) for quicker setup. 

Most examples in this tutorial sign transactions programmatically however. 


## creating the main structure of your script 

create a new file or edit `index.ts`

    import {  Client,  Wallet }  from "xrpl" 
    
    const client = new Client("wss://s.altnet.rippletest.net:51233")


    const main = async () => {
      console.log("lets get started...");
      await client.connect();

      // do something interesting here

  
      await client.disconnect();
      console.log("all done!");
    };

    main();

Running this to make sure everything works should console log a few things.
If you are running this from your own node environment, don't forget to `npm i xrpl` to add it to your project.


## creating nft 

Lets add some nft creation logic, you might have one wallet available for that.

    import { convertStringToHex, NFTokenMint, NFTokenMintFlags, NFTokenCreateOffer } from 'xrpl';
    
    const { wallet: wallet1, balance: balance1 } = await client.fundWallet()    
    console.log('wallet1', wallet1)

    const uri = "My super NFT URI"

    const nftMintTx: NFTokenMint = {
        TransactionType: "NFTokenMint",
        Account: wallet.address,
        URI: convertStringToHex(uri),
        Flags: NFTokenMintFlags.tfBurnable + NFTokenMintFlags.tfTransferable, // Burnable in case no one is buying it
        NFTokenTaxon: 0, // Unique identifier for the NFT type
    }

    const prepared = await client.autofill(nftMintTx)
    const signed = wallet.sign(prepared)
    const result = await client.submitAndWait(signed.tx_blob)
    const nftId = nftId: (result.result.meta as NFTokenMintMetadata)?.nftoken_id as string

    console.log("NFT ID " + nftId + " created")

At this point, you should have one account with an NFT created.

## putting your NFT on sale

Hence you have your NFT, and you would like to be able to put it on sale which would create an offer.

    import { NFTokenCreateOffer } from 'xrpl';
    
    const nftCreateOfferTx: NFTokenCreateOffer = {
        TransactionType: "NFTokenCreateOffer",
        Account: wallet.address,
        Destination: destination, // Optional if precised it would precised the offer for a specific address
        NFTokenID: nftId,
        Amount: "0", // 0 would represent a gift to someone
        Flags: 1 // Sell offer
    };

    const prepared = await client.autofill(nftCreateOfferTx)
    const signed = wallet.sign(prepared)
    const result = await client.submitAndWait(signed.tx_blob)
    const offerId = (result.result.meta as NFTokenCreateOfferMetadata)?.offer_id as string

    console.log("Offer ID " + offerId + " created")

At this point, your NFT should be on sale and would be able to someone specific or anyone to buy it.

## Interact with others NFT

Connect and add your public address to the list using the training app. 
When you click view tokens next to an address you can see that account's available nfts.

You then should be able to buy other NFTs using your Xaman wallet.
