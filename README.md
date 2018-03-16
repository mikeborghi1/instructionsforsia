1. First download the siad daemon from here or download my zipped copy from [here]()

	  [https://sia.tech/get-started](https://sia.tech/get-started)

2. Extract the folder and run the siad.exe to begin syncing.  You should end up with the following folder structure   
```
	>consensus
	>> consensus.db <-- 10gb, drag from zip
	>> consensus.log
	>doc
	>gateway
	>host
	>renter
	>transactionpool <-- has a 1gb transactionpool.db file, drag from 
	>wallet 
	siad
    siac
    otherstuff
	
```
3. Delete the contents of the wallet folder so we know we're starting fresh
4. then download the sia.js git repo 
```
	git clone https://github.com/NebulousLabs/Nodejs-Sia
	cd Nodejs-Sia
	npm install sia.js
```
The daemon runs on localhost:9980 

## The following is from the readme in the sia.js repo:


### Example Usage

```js
import { connect } from 'sia.js'

// Using promises...
// connect to an already running Sia daemon on localhost:9980 and print its version
connect('localhost:9980')
  .then((siad) => {
    siad.call('/daemon/version').then((version) => console.log(version))
  })
  .catch((err) => {
    console.error(err)
  })

// Or ES7 async/await
async function getVersion() {
  try {
    const siad = await connect('localhost:9980')
    const version = await siad.call('/daemon/version')
    console.log('Siad has version: ' + version)
  } catch (e) {
    console.error(e)
  }
}

```
You can also forgo using `connect` and use `call` directly by providing an API address as the first parameter:

```js
import { call } from 'sia.js'

async function getVersion(address) {
  try {
    const version = await call(address, '/daemon/version')
    return version
  } catch (e) {
    console.error('error getting ' + address + ' version: ' + e.toString())
  }
}

console.log(getVersion('10.0.0.1:9980'))
```

`sia.js` can also launch a siad instance given a path on disk to the `siad` binary.  `launch` takes an object defining the flags to use as its second argument, and returns the `child_process` object.  You are responsible for keeping track of the state of this `child_process` object, and catching any errors `launch` may throw.

```js
import { launch } from 'sia.js'

try {
  // Flags are passed in as an object in the second argument to `launch`.
  // if no flags are passed, the default flags will be used.
  const siadProcess = launch('/path/to/your/siad', {
    'modules': 'cghmrtw',
    'profile': true,
  })
  // siadProcess is a ChildProcess class.  See https://nodejs.org/api/child_process.html#child_process_class_childprocess for more information on what you can do with it.
  siadProcess.on('error', (err) => console.log('siad encountered an error ' + err))
} catch (e) {
  console.error('error launching siad: ' + e.toString())
}
```

The call object passed as the first argument into call() are funneled directly
into the [`request`](https://github.com/request/request) library, so checkout
[their options](https://github.com/request/request#requestoptions-callback) to
see how to access the full functionality of [Sia's
API](https://github.com/NebulousLabs/Sia/blob/master/doc/API.md)

```js
Siad.call({
  url: '/consensus/block',
  method: 'GET',
  qs: {
    height: 0
  }
})
```

Should log something like:

```bash
null { block:
 { parentid: '0000000000000000000000000000000000000000000000000000000000000000',
   nonce: [ 0, 0, 0, 0, 0, 0, 0, 0 ],
   timestamp: 1433600000,
   minerpayouts: null,
   transactions: [ [Object] ] } }
```
##

 Decide how you want to launch the daemon either through a shell script (batch script for win) or using the sia.js api and specifying the path as shown above. 
 
4. In the sia.js repo is a folder 'test' which contains a file sia.js which uses nock to test the various daemon functions to make sure its operational.  do this by running the following command
	```
    npm test
    ```
	There are other scripts in the package.json but the one for test is 
    ```
        "test": "npm run lint && mocha --compilers js:babel-register --recursive ./test",
     ```
     Not really familiar with all that shit but if the tests all pass then you're gucci.  hit me up if the tests fail.
     
5. this file contains all the api info, read through the top part [https://github.com/NebulousLabs/Sia/blob/master/doc/API.md](https://github.com/NebulousLabs/Sia/blob/master/doc/API.md) and [this](https://github.com/NebulousLabs/Sia/tree/master/doc/api) folder contains detailed responses for each of the api calls.  You really only need [wallet.md](https://github.com/NebulousLabs/Sia/blob/master/doc/api/Wallet.md) but use the 

		/consensus [get] 
  
	call so that you know if the wallet is synced and what the blockheight is as described [here](https://github.com/NebulousLabs/Sia/blob/master/doc/api/Consensus.md#consensus-get)
    


#### Note the following from the top of api.md:

Notes:
- Requests must set their User-Agent string to contain the substring "Sia-Agent".
- By default, siad listens on "localhost:9980". This can be changed using the
  `--api-addr` flag when running siad.
- **Do not bind or expose the API to a non-loopback address unless you are
  aware of the possible dangers.**

Example GET curl call:
```
curl -A "Sia-Agent" "localhost:9980/wallet/transactions?startheight=1&endheight=250"
```

Example POST curl call:
```
curl -A "Sia-Agent" --data "amount=123&destination=abcd" "localhost:9980/wallet/siacoins"
```
##

6. init the wallet by calling /wallet/init as described here [https://github.com/NebulousLabs/Sia/blob/master/doc/api/Wallet.md#walletinit-post](https://github.com/NebulousLabs/Sia/blob/master/doc/api/Wallet.md#walletinit-post) and it will return the seed. 
7. call /wallet/unlock [post] to unlock the wallet, specify the password if you used one upon init'ing the wallet.
8. Call /wallet [get] to get info about the wallet such as if its encrypted, locked, and balance.
9. You now have a functional wallet.  Here are the calls you will need.
```
Get new address:
/wallet/address [get]

List all addresses
/wallet/addresses [get]

List all tx's
/wallet/transactions	GET

List tx's on an addy
/wallet/transactions/___:addr___	GET

get info about a tx
/wallet/transaction/___:id___	GET

verify an address (for withdrawals)
/wallet/verify/address/:addr	GET

