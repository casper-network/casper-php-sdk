# casper-php-sdk
The PHP SDK allows developers to interact with the Casper Network using PHP. This page covers different examples of using the SDK.

## Install

```
composer require make-software/casper-php-sdk
```

## API

- [RpcClient](docs/API/RpcClientAPI.md)

## Entity

- [Account](docs/Entity/Account.md)
- [Deploy](docs/Entity/Deploy.md)

## Examples

#### Obtain some information from the network:
```php
use Casper\Rpc\RpcClient;

$nodeUrl = 'http://127.0.0.1:7777';
$client = new RpcClient($nodeUrl);

$peers = $client->getPeers();
$latestBlock = $client->getLatestBlock();
$auctionState = $client->getAuctionState();
...
```

#### Generate key pair, export in pem, sign, and verify a message:
```php
use Casper\Util\Crypto\Ed25519Key;
use Casper\Util\Crypto\Secp256K1Key;

$keyPair = new Ed25519Key();
// or
$keyPair = new Secp256K1Key();
```

```php
$publicKeyPemString = $keyPair->exportPublicKeyInPem();
$privateKeyPemString = $keyPair->exportPrivateKeyInPem();

$message = 'Hello';
$signature = $keyPair->sign($message);

$isVerified = $keyPair->verify($signature, $message);
```

#### Sending a Transfer:
```php
use Casper\Entity\DeployExecutable;
use Casper\Entity\DeployParams;
use Casper\Rpc\RpcClient;
use Casper\Serializer\CLPublicKeySerializer;
use Casper\Service\DeployService;
use Casper\Util\Crypto\Secp256K1Key;

// Replace '/path/to/secp256k1_secret_key.pem' by real path to secret key
$senderKeyPair = Secp256K1Key::createFromPrivateKeyFile('/path/to/secp256k1_secret_key.pem');
$senderPublicKey = CLPublicKeySerializer::fromAsymmetricKey($senderKeyPair);
$networkName = 'casper';
$deployParams = new DeployParams($senderPublicKey, $networkName);

// Replace 'recipient_hex_public_key_here' by real public key
$recipientPublicKey = CLPublicKeySerializer::fromHex('recipient_hex_public_key_here');
$transferId = 1;
$transferAmount = 2500000000;
$session = DeployExecutable::newTransfer($transferId, $transferAmount, $recipientPublicKey);

$paymentAmount = 10;
$payment = DeployExecutable::newStandardPayment($paymentAmount);

$deployService = new DeployService();
$deploy = $deployService->makeDeploy($deployParams, $session, $payment);
$signedDeploy = $deployService->signDeploy($deploy, $senderKeyPair);

$rpcClient = new RpcClient('http://127.0.0.1:7777');
$deployHash = $rpcClient->putDeploy($signedDeploy);
```

## Roadmap
- [x] Create an RPC client that returns array responses
- [x] Implement all GET requests
- [x] Add `Ed25519` key support
- [x] Add `CLType` primitives
- [x] Add domain-specific entities
- [x] Add deploy creation related services
- [x] Add `Secp256k1` key support
- [ ] Add extensive validations
- [ ] Add automated tests
- [ ] Add documentation
