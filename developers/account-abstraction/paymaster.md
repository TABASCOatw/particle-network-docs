---
description: >-
  Omnichain Paymaster - Recharge USDT on one chain and use it for multiple
  chains
---

# Paymaster

Particle Network has implemented a Paymaster service that supports multiple chains. You can recharge USDT on one chain(Ethereum and BNB Chain supported now) and use the Paymaster service on any chain. Particle Paymaster will automatically calculate the exchange rate and deduct it in the background.

## What is Particle Paymaster

Particle Paymaster is a multi-chain Verifying Paymaster, which means

1. Deposit once and sponsor UserOp on every chain;
2. Bring any sponsorship logic using Webhooks;
3. Monitor every UserOp you sponsored.

{% hint style="info" %}
The ERC20 Token Paymaster is used by the user directly, so it's hooked into our SDK directly, no extra configuration is needed.
{% endhint %}

## How to use

1. Use [Particle Network's AA SDKs](sdks/), Particle's Paymaster will be automatically included, and no extra configuration is needed;

{% content-ref url="sdks/" %}
[sdks](sdks/)
{% endcontent-ref %}

2. Use [Particle Paymaster's RPC](rpc/paymaster-rpc.md) directly to plug the multi-chain paymaster in your AA project.

{% content-ref url="rpc/paymaster-rpc.md" %}
[paymaster-rpc.md](rpc/paymaster-rpc.md)
{% endcontent-ref %}

{% hint style="info" %}
For any testnet, it's sponsored by Particle Network automatically, no need to deposit
{% endhint %}

3\. Because of our modular design, you can always use any other paymaster you want, we have also natively integrated [Biconomy Paymater](https://dashboard.biconomy.io/).

{% hint style="info" %}
Biconomy Paymaster only supports BICONOMY smart account.
{% endhint %}

## Expiration time

You have the option to define an expiration time for the Paymaster Signature. Once the user operation reaches its expiration, the Paymaster Signature will automatically lapse.

## Webhook

> Configuring webhooks allows you to accurately control which userOP can be accepted by Paymaster

There are two types of webhooks

* before paymaster sign (before\_paymaster\_sign)
* after paymaster sign (after\_paymaster\_sign)

### When sponsor

If the response for **before paymaster sign** webhook's status code is **200**, we will consider it passed;

We recommend you use **400** for rejected sponsorship.

### Webhook Source Verification

Every time a Webhook request is made, we generate a signature for the body, and developers can verify the signature to determine whether the request was sent by Particle Network

We have generated a unique public and private key (RSA-2048) for each project, and you can download the public key from the dashboard page for verification

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption><p>Paymaster Page</p></figcaption></figure>

Verification Example:

```typescript
import NodeRSA from 'node-rsa';

const signature = request.headers['x-particle-signature'];
const data = JSON.stringify(request.body);
const nodeRSA = new NodeRSA(publicKey);
const verified: boolean = nodeRSA.verify(data, Buffer.from(signature, 'base64'));

```

### Before paymaster sign

> This hook will trigger before Paymaster signs. The Paymaster will determine whether to sign the UserOP based on the status code returned by Hook

* Body
  * chainId
  * userOp - the struct of user operation
  * entryPoint - The entry point address
  * parsed - Transaction struct. Paymaster will attempt to parse the calldata of UserOP. If it cannot be parsed, this field may not exist
* Response
  * If the status code of the response returned is **200**, then the Paymaster would accept the UserOP and sign it

Example:

```typescript
POST https://your-domain/hook-before-paymaster-sign

Header
x-particle-signature: WH/6YifcF2dad3qbS4HkjQ7EboDXNTcIfCH4cCW0lfOWJsQR1GPoHZpqtqr3XedAG3bi+RlTUGmetJiaiCqflr1+dWuXFsvrML7LNu2wWNDzlGAEM3iLwrdHl1vIZx6ftlbYyyQD4CHEBpDhbnrDn2IGj64dF8nmho4gc/PNQQUTBJL+Xy0MEHVLSNSDeRMA5/BhwFjiufvDqmW3verX/ynvUAVLnLiAWkVnN2aBF4TncsqV37W8EN3q2SILpdnbG2UWzeawg+0owtw9xgo4QXgqi8PfYrd3ta5PxjoenbUA7OBJtXt4R/TwYs8Vb+mggJ+ZbrJrVd45F8U9hNt7bQ==

BODY
{
    "type": "before_paymaster_sign",
    "chainId": 80001,
    "projectUuid": "ef6c29f5-ad2b-545c-ad0c-54441068b71d",
    "userOp": {
        "sender": "0x2e9661BDA6201F97430dcc1541A1579b83980DD1",
        "nonce": "0x05",
        "initCode": "0x",
        "callData": "0xb61d27f6000000000000000000000000ac6a87c681a5ed4cb58bc4fa7bf81a83b928c83c00000000000000000000000000000000000000000000000000005af3107a400000000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000",
        "callGasLimit": "0x9acc",
        "paymasterAndData": "0x",
        "signature": "0x",
        "verificationGasLimit": "0x0186a0",
        "maxFeePerGas": "0xa79dd02a",
        "maxPriorityFeePerGas": "0xa79dd015",
        "preVerificationGas": "0xc5dc"
    },
    "entryPoint": "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
    "parsed": {
        "accountType": "SIMPLE",
        "txs": [
            {
                "to": "0xaC6A87c681A5Ed4cb58bC4fa7bF81a83b928C83C",
                "value": "0x5af3107a4000",
                "data": "0x"
            }
        ]
    }
}

```

### After paymaster sign

> This hook will trigger after Paymaster signs

* Body
  * chainId
  * userOp - the struct of user operation
  * entryPoint - The entry point address
  * parsed - Transaction struct. Paymaster will attempt to parse the calldata of UserOP. If it cannot be parsed, this field may not exist

Example

```typescript
POST https://your-domain/hook-after-paymaster-sign

Header
x-particle-signature: WH/6YifcF2dad3qbS4HkjQ7EboDXNTcIfCH4cCW0lfOWJsQR1GPoHZpqtqr3XedAG3bi+RlTUGmetJiaiCqflr1+dWuXFsvrML7LNu2wWNDzlGAEM3iLwrdHl1vIZx6ftlbYyyQD4CHEBpDhbnrDn2IGj64dF8nmho4gc/PNQQUTBJL+Xy0MEHVLSNSDeRMA5/BhwFjiufvDqmW3verX/ynvUAVLnLiAWkVnN2aBF4TncsqV37W8EN3q2SILpdnbG2UWzeawg+0owtw9xgo4QXgqi8PfYrd3ta5PxjoenbUA7OBJtXt4R/TwYs8Vb+mggJ+ZbrJrVd45F8U9hNt7bQ==

BODY
{
    "type": "after_paymaster_sign",
    "chainId": 80001,
    "projectUuid": "ef6c29f5-ad2b-545c-ad0c-54441068b71d",
    "userOp": {
        "sender": "0x2e9661BDA6201F97430dcc1541A1579b83980DD1",
        "nonce": "0x05",
        "initCode": "0x",
        "callData": "0xb61d27f6000000000000000000000000ac6a87c681a5ed4cb58bc4fa7bf81a83b928c83c00000000000000000000000000000000000000000000000000005af3107a400000000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000",
        "callGasLimit": "0x9acc",
        "paymasterAndData": "0xc89b723809598d0ebf821e89087dc8e1a6ee04990000000000000000000000000000000000000000000000000000000065520368000000000000000000000000000000000000000000000000000000000000000053264ff4c95bfc05d0635b8745f698911a103d11e4995f2d532e931807800aa0313d269ef5824ea5d122af0b08e70c0777898add80e70fffbd81154760eac0991b",
        "signature": "0x",
        "verificationGasLimit": "0x0186a0",
        "maxFeePerGas": "0xa79dd02a",
        "maxPriorityFeePerGas": "0xa79dd015",
        "preVerificationGas": "0xc5dc"
    },
    "entryPoint": "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
    "userOpHash": "0x45d4f5bb4114907a09426000e152eea5511576924d58700f7f2561f32a552476",
    "parsed": {
        "accountType": "SIMPLE",
        "txs": [
            {
                "to": "0xaC6A87c681A5Ed4cb58bC4fa7bF81a83b928C83C",
                "value": "0x5af3107a4000",
                "data": "0x"
            }
        ]
    }
}

```

## How to use Webhook for your sponsorship policy

1. For the **before\_paymaster\_sign** webhook, we have returned all the data necessary&#x20;
   1. **projectUuid**
   2. **chainId**
   3. **userOp**
   4. and parsed data
      1. **accountType: BICONOMY, CYBERCONNECT, SIMPLE**
      2. **txs**
2. Any policy
   1. You can use **chainId** to decide which chains you want to sponsor;
   2. You can use **userOp** to decide which user address(sender) you want to sponsor;
   3. You can use **userOp** to calculate the gas fee and decide whether to sponsor or not;
   4. You can use the **accountType** and **txs** to use them as a smart contract whitelist or blacklist.
   5. ...

## Config & Check Paymaster

1. Check Balance & Deposit;
2. Check UserOp sponsor records.

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>
