####This is a tutorial to use the IOTA Core Client library. It will alllow you to use the IOTA protocol to send and recieve transactions.

***With the Core Client library, you can:***

- Build transactions and package them into bundles before sending them to a node
- Connect to a node and send it transactions to attach to the Tangle
- Connect to a node and request data from the Tangle such as transactions, and the balances of addresses



.
####A) In this tutorial, you send a "hello world" message in a zero-value transaction. These transactions are useful for storing messages in the Tangle without having to send any IOTA tokens.


***For all tutorials A) through D), we use the [IOTA Devnet]("https://docs.iota.org/docs/getting-started/1.1/networks/overview)***

First, make sure you have the software you need:


1. Install the latest long-term support (LTS) version of [Node.js]("https://nodejs.org/en/download/ "Node.js download")
2. Install a code editor. We recommend [Visual Studio Code]("https://code.visualstudio.com/Download" "Visual Studio Code download"), but many more are available.
3. Open a command-line interface
Depending on your operating system, a command-line interface could be [PowerShell in Windows]("https://docs.microsoft.com/en-us/powershell/?view=powershell-7.1 "Powershell for Windows download"), the [Linux Terminal]("https://www.howtogeek.com/140679/beginner-geek-how-to-start-using-the-linux-terminal/" "Linux Terminal download") or [Terminal for macOS]("https://macpaw.com/how-to/use-terminal-on-mac" "Download").
4. Create a new directory to use for your project in for example PowerShell:

>mkdir myNewProject
cd myNewProject

.
Then, initialize a new project. Make sure you are in the root of your project-folder. This will install a Package.JSON for your project. It will ask you a few questions about the meta-data for your project. 
In this and all following examples, we have used NPM. NPM is included in the download of Powershell, otherwise you can download it at [NPM.JS](https://www.npmjs.com/get-npm). If you prefer, You can use [Yarn]("https://yarnpkg.com/"), another package manager.

>npm init

.
Now you install the needed packages.


>npm install @iota/core @iota/converter

After the download has finished, you will find these two libraries in a folder called node_modules.

Then it is time to connect to a node in the IOTA network.


**1.** Require the packages

>const Iota = require('@iota/core');
const Converter = require('@iota/converter');

.
**2.** Connect to a node:

>const iota = Iota.composeAPI({
provider: 'https://nodes.devnet.iota.org:443'
});

.
**3.** Define the depth and the minimum weight magnitude

>const depth = 3;
const minimumWeightMagnitude = 9;

.
**4.** Define an address to which you want to send a message:

>const address =
'HEQLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWORLDHELLOWOR99D';


This address does not have to belong to anyone. To be valid, the address just needs to consist of 81 trytes.

.
**5.** Define a seed
>const seed =
'PUEOTSEITFEVEWCWBTSIZM9NKRGJEIMXTULBACGFRQK9IMGICLBKW9TTEVSDQMGWKBXPVCBMMCXWMNPDX';


Because this is a zero-value transaction, the seed is not used. However, the library expects a valid seed, so we use a random string of 81 characters. If you enter a seed that consists of less than 81 characters, the library will append 9s to the end of it to make 81 characters.

**6.** Create a JSON message that you want to send to the address and convert it to trytes

>const message = JSON.stringify({"message": "Hello world"});
const messageInTrytes = Converter.asciiToTrytes(message);

We encode the message in JSON to make it easier to read the message when we get the transaction from the Tangle in the next guide.

.
**7.** Define a zero-value transaction that sends the message to the address

>const transfers = [
{
    value: 0,
    address: address,
    message: messageInTrytes
}
];

.
**8.** To create a bundle from your transfers object, pass it to the prepareTransfers() method. Then, pass the returned bundle trytes to the sendTrytes() method, which handles tip selection, remote proof of work, and sending the bundle to the node. For details about this process, see Sending transactions.

>iota.prepareTransfers(seed, transfers)
    .then(trytes => {
        return iota.sendTrytes(trytes, depth, minimumWeightMagnitude);
    })
    .then(bundle => {
        console.log(bundle[0].hash)
    })
    .catch(err => {
        console.error(err)
    });


.
In the console, you should see the tail transaction hash of the bundle you just sent.

>Congratulations 🎉
You've just sent your first zero-value transaction. Your transaction is attached to the Tangle, and will be forwarded to the rest of the network.

You can use this tail transaction hash to read the transaction from the Tangle.


.
####B) In this tutorial, you read your "hello world" transaction from the Tangle by giving a node your tail transaction hash.


.
***Packages***


To complete this tutorial, you need to install the following packages:

>npm install @iota/core @iota/extract-json

.
***Code walkthrough***

**1.** Require the packages
>const Iota = require('@iota/core');
const Extract = require('@iota/extract-json');

**2.** Connect to a node

>const iota = Iota.composeAPI({
provider: 'https://nodes.devnet.iota.org:443'
});

**3.** Define the tail transaction hash of the bundle

>const tailTransactionHash =
'ZFICKFQXASUESAWLSFFIWHVOAJCSJHJNXMRC9AJSIOTNGNKEWOFLECHPULLJSNRCNJPYNZEC9VGOSV999';


We use the tail transaction hash because the signatureMessageFragmentfield is part of the hash. Therefore, the message in the transaction is immutable.

If you were to use the bundle hash, you may see a different message because anyone canchange the message in the tail transactionand attach a copy of the bundle to the Tangle.


**4.** Use the getBundle() method to get all transactions in the tail transaction's bundle. Then, use the extractJSON() method to decode the JSON messages in the signatureMessageFragment fields of the bundle's transactions and print them to the console

>iota.getBundle(tailTransactionHash)
.then(bundle => {
    console.log(JSON.parse(Extract.extractJson(bundle)));
})
.catch(err => {
    console.error(err);
});

In the console, you should see your JSON message:

>{"message": "Hello world"}


.
>Congratulations 🎉
You've just found and read a transaction from the Tangle.

.
####C) Change the messages in a bundle

**You will need to install these packages:**

>npm install @iota/core @iota/converter @iota/bundle @iota/transaction @iota/transaction-converter

.
**Code walkthrough**

**1.** Require the packages
>const Iota = require('@iota/core');
const Converter = require('@iota/converter');
const Bundle = require('@iota/bundle');
const { TRANSACTION_LENGTH, SIGNATURE_OR_MESSAGE_LENGTH } = require('@iota/transaction');
const { asTransactionTrytes } = require('@iota/transaction-converter');

**2.** Connect to a node
const iota = Iota.composeAPI({
provider: 'https://nodes.devnet.iota.org:443'
});

**3.** Define the hash of the tail transaction whose message you want to change
>const tail = 'UZSQCOKPEDTIZWLFNJWTPDNYZCYYHAMJAJVVHOHAHSQLPYOYYN9PT9DN9OOCESNS9RPYFIESTOCGCL999'


We use a tail transaction because it is more likely to be an input, which means that it won't contain a signature.

If we were to change a signature, the bundle would be invalid. But, if we change a message, the bundle will still be valid after we do proof of work to calculate the new transaction hash.

**4.** Use the [getBundle()](https://github.com/iotaledger/iota.js/blob/next/api_reference.md#module_core.getBundle) method to get all transactions in the tail transaction's bundle
>iota.getBundle(tail).then(bundle => {
>
>});

**5.** Create a copy of the returned bundle
>// Create new bundle array with a length equal to the number of transactions in the bundle
let newBundle = new Int8Array(bundle.length * TRANSACTION_LENGTH);
for (let i = 0; i < bundle.length; i++) {
    // Fill the bundle with the transaction trits from the bundle
    newBundle.set(Converter.trytesToTrits(asTransactionTrytes(bundle[i])), i * TRANSACTION_LENGTH);
}

**6** Use the [addSignatureOrMessage()](https://github.com/iotaledger/iota.js/tree/next/packages/bundle#bundleaddsignatureormessagebundle-signatureormessage-index) method to add a new message to the tail transaction of your copy

>// Define an array to hold your new message
const message = new Int8Array(SIGNATURE_OR_MESSAGE_LENGTH)
// Set the new message
message.set(Converter.trytesToTrits("World \n Hello"))
// Add the new message to the tail transaction in the bundle
newBundle = Bundle.addSignatureOrMessage(newBundle,message,0);

**7.** Use the finalizeBundle() method to calculate the bundle hash of your copy

>newBundle = Bundle.finalizeBundle(newBundle);

This bundle hash is the same as the original bundle.

**8.** Convert the transactions in your copy from trits to trytes

>const newTrytes = [];
for (let i = 0; i < bundle.length; i++) {
    // Convert the transaction trits to trytes and add them to a new array
    newTrytes.push(Converter.tritsToTrytes(newBundle.slice(i * TRANSACTION_LENGTH, (i + 1) * TRANSACTION_LENGTH)))
}

**9.** Send your copy to a node

>iota.sendTrytes(newTrytes.reverse(),3,9).then(transactions => {
// Print your new tail transaction hash to the console
console.log(transactions[0].hash);
})


You reverse the bundle array because the library expects bundles to be sent head first.

Now, you can search for your new tail transaction in the Tangle and see that it's in a bundle with the same bundle hash as the original.

.
>Congratulations 🎉
You've just changed the message of a tail transaction in a bundle and reattached a copy of that bundle to the Tangle.



If your original tail transaction belongs to a transfer bundle, nodes will mark either your copy or the original bundle as a double spend. Therefore, only one of them will be confirmed.


.
####D) Check if a transaction has been confirmed

***Before IOTA tokens can be transferred, the transactions must be confirmed. Transactions in a bundle remain in a pending state until the tail transaction is referenced and approved by a milestone.***


.
**Packages**

To complete this tutorial, you need to install the following package:

>npm install @iota/core

.
**Code walkthrough**
1. Go to [explorer.iota.org]("https://explorer.iota.org/mainnet" "The IOTA Explorer") and find a confirmed transaction.

>Can't find a confirmed transaction?
Click a transaction hash in the Latest milestones box, then click the branch transaction hash. This transaction is referenced and approved by the milestone, so it is in a confirmed state.

2. Pass the transaction hash to the getInclusionStates() method to check if the IOTA node's latest solid subtangle milestone approves it

>iota.getInclusionStates(['TRANSACTION HASH'])
.then(states => console.log(states));

When you execute the file, you should see an array that contains the true boolean, meaning that the transaction is confirmed.

>You could also use the getInclusionStates()method to check if a transaction is approved by an array of your own chosen transactions.

3. Go to [explorer.iota.org]("https://explorer.iota.org/mainnet" "The IOTA Explorer") and find a pending transaction.


>Can't find a pending transaction?
Click a transaction hash in the Latest transactions box. This transaction is a tip, so it is in a pending state.

4. Pass the transaction hash to the getInclusionStates() method to check if the IOTA node's latest solid subtangle milestone approves it

>iota.getInclusionStates(['TRANSACTION HASH'])
.then(states => console.log(states));

When you execute the file, you should see an array that contains the false boolean, meaning that the transaction is not yet confirmed.

