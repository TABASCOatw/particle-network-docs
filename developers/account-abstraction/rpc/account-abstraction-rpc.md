---
description: High-Level RPCs to simplify the integration of AA
---

# Enhanced AA RPC

{% content-ref url="../available-networks.md" %}
[available-networks.md](../available-networks.md)
{% endcontent-ref %}

## Before Start

1. All testnets have Particle Verifying Paymaster enabled, you can send any gasless transaction;
2. If you want to use [Particle Verifying Paymaster](../paymaster.md) on Mainnet, you can deposit and use it directly without extra configration in SDK, [check docs](../paymaster.md);
3. If you want to use Biconomy Paymaster, please go to [https://dashboard.biconomy.io/](https://dashboard.biconomy.io/), create Paymaster & get dappApiKey;
4. Token Paymaster is automatically enabled (we are using Biconomy Token Paymaster).

\-> [RPC Authentication & chainId config](../../node-service/authentication.md)

## How to use

1. Get User Smart Account Status: `particle_aa_getSmartAccount`
2. Call `particle_aa_getFeeQuotes` to obtain data for `verifyingPaymasterGasless`, `verifyingPaymasterNative`, and `tokenPaymaster`, corresponding to Gasless, Native Gas Payment, and ERC20 Token Gas Payment options:
   1. If `verifyingPaymasterGasless` is null, it means gasless is not available; if not null, it contains `userOp` and `userOpHash`.
   2. Inside `verifyingPaymasterNative`, there are `userOp`, `userOpHash`, and `feeQuote` (used for display and balance verification).
   3. `tokenPaymaster` contains `feeQuotes` and `tokenPaymasterAddress`.
3. If the user is eligible and chooses the gasless, sign `userOpHash`, then add the signature field to `userOp`, and call `particle_aa_sendUserOp`.
4. If the user is eligible and chooses native gas payment, sign `userOpHash`, then add the signature field to `userOp`, and call `particle_aa_sendUserOp`.
5. If the user is eligible and chooses token gas payment, the first call `particle_aa_createUserOp` to create a `userOp` and the corresponding `userOpHash`; then sign `userOpHash`, add the signature field to `userOp`, and call `particle_aa_sendUserOp`.
6. The `result` returned by `particle_sendUserOp` is the `txHash`.
7. If you need to use the Session Key feature, you can call the API `particle_aa_createSessions` to register a session key on chain. It should be noted that this feature is currently only supported by Biconomy 2.0.0 smart account implementation.

## Common Params

The first param for the params field of JSON RPC is **account config**, the structure is as follows:

```json
  {
    "name": "BICONOMY", // SIMPLE | CYBERCONNECT | BICONOMY
    "version": "1.0.0", // SIMPLE 1.0.0; CYBERCONNECT 1.0.0; BICONOMY 1.0.0, 2.0.0.
    "ownerAddress": "0x8d5f6E013b74D3aBa5cEB483fD5209C8Dab794E9", // The Owner's address for the Smart Account
    "biconomyApiKey": "dappApiKey" // optional: Biconomy Paymaster ApiKey if you use Biconomy Paymaster
  }
```

## particle\_aa\_getSmartAccount

Request

```JSON
{
  "jsonrpc": "2.0",
  "id": "ee9cce2a-2f34-4c66-879e-c84c6f0e7f2d",
  "chainId": 80001,
  "method": "particle_aa_getSmartAccount",
  "params": [
    // account config array
    {
      "name": "BICONOMY",
      "version": "1.0.0",
      "ownerAddress": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5"
    },
    {
      "name": "BICONOMY",
      "version": "1.0.0",
      "ownerAddress": "0x329a7f8b91Ce7479035cb1B5D62AB41845830Ce8"
    }
  ]
}
```

Response

```JSON
{
  "jsonrpc": "2.0",
  "id": "ee9cce2a-2f34-4c66-879e-c84c6f0e7f2d",
  "result": [
    {
      "isDeployed": true,
      "chainId": 80001,
      "eoaAddress": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5", // also owner
      "smartAccountAddress": "0x8Fb859E944561678be40Cdd2dB16551396c0b074",
      "fallBackHandlerAddress": "0xa04EeF9bBFd8F64d5218d4f3a3d03e8282810F51",
      "implementationAddress": "0x00006B7e42e01957dA540Dc6a8F7C30c4D816af5",
      "factoryAddress": "0x000000F9eE1842Bb72F6BBDD75E6D3d4e3e9594C",
      "owner": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5",
      "entryPointAddress": "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
      "index": 0,
      "name": "BICONOMY",
      "version": "1.0.0"
    },
    {
      "isDeployed": false,
      "chainId": 80001,
      "eoaAddress": "0x329a7f8b91Ce7479035cb1B5D62AB41845830Ce8",
      "factoryAddress": "0x000000F9eE1842Bb72F6BBDD75E6D3d4e3e9594C",
      "entryPointAddress": "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
      "smartAccountAddress": "0x71f522803316C92731DDD007Fdb13281f52c1eef",
      "owner": "0x329a7f8b91Ce7479035cb1B5D62AB41845830Ce8",
      "index": 0,
      "implementationAddress": "0x00006B7e42e01957dA540Dc6a8F7C30c4D816af5",
      "fallBackHandlerAddress": "0xa04EeF9bBFd8F64d5218d4f3a3d03e8282810F51",
      "name": "BICONOMY",
      "version": "1.0.0",
      "createdAt": 1691460553368,
      "updatedAt": 1691460553368
    }
  ],
  "chainId": 80001
}
```

## particle\_aa\_getFeeQuotes

Request

```JSON
{
  "jsonrpc": "2.0",
  "id": "f216c7ea-3986-4784-954f-e54320c75b40",
  "chainId": 80001,
  "method": "particle_aa_getFeeQuotes",
  "params": [
    // account config
    {
      "name": "BICONOMY",
      "version": "1.0.0",
      "ownerAddress": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5",
      "biconomyApiKey": "LdF-gC43H.6f0ec763-fde5-4d5e-89f6-1b9ef12ba4a4" // optional
    },
    // txs
    [
      {
        "to": "0x329a7f8b91Ce7479035cb1B5D62AB41845830Ce8",
        "value": "0x1",
        "data": "0x"
      }
    ],
    // optional boolen: true - will check and include gasless; false - will skip gasless check
    // default is true
    true
  ]
}
```

Response

```JSON
{
  "jsonrpc": "2.0",
  "id": "f216c7ea-3986-4784-954f-e54320c75b40",
  "result": {
    // maybe null
    "verifyingPaymasterGasless": {
      "userOp": {
        "sender": "0x8fb859e944561678be40cdd2db16551396c0b074",
        "nonce": "0x62",
        "initCode": "0x",
        "callData": "0x9e5d4c49000000000000000000000000329a7f8b91ce7479035cb1b5d62ab41845830ce8000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000",
        "signature": "0x73c3ac716c487ca34bb858247b5ccf1dc354fbaabdd089af3b2ac8e78ba85a4959a2d76250325bd67c11771c31fccda87c33ceec17cc0de912690521bb95ffcb1b",
        "maxFeePerGas": "1500000034",
        "maxPriorityFeePerGas": "1500000000",
        "verificationGasLimit": "0x13ac6",
        "callGasLimit": "0x99a5",
        "preVerificationGas": "0xb678",
        "paymasterAndData": "0x000031dd6d9d3a133e663660b959162870d755d4000000000000000000000000329a7f8b91ce7479035cb1b5d62ab41845830ce800000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000041de683496be0efcc84e2d3e4d0de6d2f4a5baae1999dfc9da0cbc23c6f8efd0572213f5d86955632257adafca88c6126a173136db2cb7f5dcca68e03947acb3771c00000000000000000000000000000000000000000000000000000000000000"
      },
      "userOpHash": "0x0300e5013b55034c9005f4faf2339e813f1bb9dfe06ef85c5ee483dca32ef891"
    },
    // won't be null
    "verifyingPaymasterNative": {
      "userOp": {
        "sender": "0x8fb859e944561678be40cdd2db16551396c0b074",
        "nonce": "0x62",
        "initCode": "0x",
        "callData": "0x9e5d4c49000000000000000000000000329a7f8b91ce7479035cb1b5d62ab41845830ce8000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000",
        "signature": "0x73c3ac716c487ca34bb858247b5ccf1dc354fbaabdd089af3b2ac8e78ba85a4959a2d76250325bd67c11771c31fccda87c33ceec17cc0de912690521bb95ffcb1b",
        "maxFeePerGas": "1500000034",
        "maxPriorityFeePerGas": "1500000000",
        "verificationGasLimit": "0x13ac6",
        "callGasLimit": "0x99a5",
        "preVerificationGas": "0xb678",
        "paymasterAndData": "0x"
      },
      "userOpHash": "0x6093806a498a4fb0e07d3a81292fde32592ecc36ddf2346338ef9e5300230ac8",
      "feeQuote": {
        "tokenInfo": {
          "chainId": 80001,
          "address": "0x0000000000000000000000000000000000000000",
          "name": "MATIC",
          "symbol": "MATIC",
          "decimals": 18,
          "logoURI": "https://static.particle.network/token-list/polygon/native.png"
        },
        "fee": "249940505665318",
        "balance": "975499686355712581"
      }
    },
    "tokenPaymaster": {
      "tokenPaymasterAddress": "0x00000f7365cA6C59A2C93719ad53d567ed49c14C",
      "feeQuotes": [
        {
          "tokenInfo": {
            "chainId": 80001,
            "name": "WMATIC",
            "symbol": "WMATIC",
            "decimals": 18,
            "address": "0x9c3C9283D3e44854697Cd22D3Faa240Cfb032889",
            "logoURI": "https://polygonscan.com/token/images/wMatic_32.png"
          },
          "fee": "374428266633331",
          "balance": "1019119852023946296",
          "premiumPercentage": "10"
        },
        {
          "tokenInfo": {
            "chainId": 80001,
            "name": "USDT",
            "symbol": "USDT",
            "decimals": 18,
            "address": "0xeaBc4b91d9375796AA4F69cC764A4aB509080A58",
            "logoURI": "https://raw.githubusercontent.com/spothq/cryptocurrency-icons/master/128/color/usdt.png"
          },
          "fee": "278847823565275",
          "balance": "0",
          "premiumPercentage": "20"
        },
        {
          "tokenInfo": {
            "chainId": 80001,
            "name": "USDC",
            "symbol": "USDC",
            "decimals": 6,
            "address": "0xdA5289fCAAF71d52a80A254da614a192b693e977",
            "logoURI": "https://raw.githubusercontent.com/spothq/cryptocurrency-icons/master/128/color/usdc.png"
          },
          "fee": "279",
          "balance": "0",
          "premiumPercentage": "20"
        },
        {
          "tokenInfo": {
            "chainId": 80001,
            "name": "DAI",
            "symbol": "DAI",
            "decimals": 18,
            "address": "0x27a44456bEDb94DbD59D0f0A14fE977c777fC5C3",
            "logoURI": "https://raw.githubusercontent.com/spothq/cryptocurrency-icons/master/128/color/dai.png"
          },
          "fee": "278594978301257",
          "balance": "0",
          "premiumPercentage": "20"
        },
        {
          "tokenInfo": {
            "chainId": 80001,
            "name": "SAND",
            "symbol": "SAND",
            "decimals": 18,
            "address": "0xE03489D4E90b22c59c5e23d45DFd59Fc0dB8a025",
            "logoURI": "https://raw.githubusercontent.com/spothq/cryptocurrency-icons/master/128/color/sand.png"
          },
          "fee": "683186002167101",
          "balance": "0",
          "premiumPercentage": "20"
        }
      ]
    }
  },
  "chainId": 80001
}
```

## particle\_aa\_createUserOp

Request

<pre class="language-JSON"><code class="lang-JSON">{
  "jsonrpc": "2.0",
  "id": "4259f7fe-87aa-44fd-89c8-fc1e41f6d3d8",
  "chainId": 80001,
  "method": "particle_aa_createUserOp",
  "params": [
    // account config
    {
      "name": "BICONOMY",
      "version": "1.0.0",
      "ownerAddress": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5",
      "biconomyApiKey": "LdF-gC43H.6f0ec763-fde5-4d5e-89f6-1b9ef12ba4a4" // optional
    },
    // txs
    [
      {
        "to": "0x329a7f8b91Ce7479035cb1B5D62AB41845830Ce8",
        "value": "0x1",
        "data": "0x"
      }
    ],
<strong>    // optional: if not pass the following params, which will creates a gasless/user paid user op
</strong>    // token feeQuote
    {
      "tokenInfo": {
        "chainId": 80001,
        "name": "WMATIC",
        "symbol": "WMATIC",
        "decimals": 18,
        "address": "0x9c3C9283D3e44854697Cd22D3Faa240Cfb032889",
        "logoURI": "https://polygonscan.com/token/images/wMatic_32.png"
      },
      "fee": "374428266633331",
      "balance": "1019119852023946296",
      "premiumPercentage": "10"
    },
    // token paymaster address
    "0x00000f7365cA6C59A2C93719ad53d567ed49c14C"
  ]
}
</code></pre>

Response

```JSON
{
  "jsonrpc": "2.0",
  "id": "4259f7fe-87aa-44fd-89c8-fc1e41f6d3d8",
  "result": {
    "userOp": {
      "sender": "0x8fb859e944561678be40cdd2db16551396c0b074",
      "nonce": "0x64",
      "initCode": "0x",
      "callData": "0x912ccaa3000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000c0000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000020000000000000000000000009c3c9283d3e44854697cd22d3faa240cfb032889000000000000000000000000329a7f8b91ce7479035cb1b5d62ab41845830ce80000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000044095ea7b300000000000000000000000000000f7365ca6c59a2c93719ad53d567ed49c14c0000000000000000000000000000000000000000000000000001548a5fd39873000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
      "signature": "0x73c3ac716c487ca34bb858247b5ccf1dc354fbaabdd089af3b2ac8e78ba85a4959a2d76250325bd67c11771c31fccda87c33ceec17cc0de912690521bb95ffcb1b",
      "verificationGasLimit": "0x163b8",
      "callGasLimit": "0xd1e7",
      "preVerificationGas": "50736",
      "paymasterAndData": "0x00000f7365ca6c59a2c93719ad53d567ed49c14c000000000000000000000000000000000000000000000000000000000064d2762e0000000000000000000000000000000000000000000000000000000064d26f260000000000000000000000009c3c9283d3e44854697cd22d3faa240cfb03288900000000000000000000000000000f7748595e46527413574a9327942e744e910000000000000000000000000000000000000000000000000de567836c0e69c0000000000000000000000000000000000000000000000000000000000010c8e09292fc583c0427b08765e4eccc030122182d141b6dbc5bb539fc4de726935042106faec9ee5b7537cf15c5f7d317fc355a0ff41877815fcbf9c906e0b584c4b61b",
      "maxFeePerGas": "1500000034",
      "maxPriorityFeePerGas": "1500000000"
    },
    "userOpHash": "0xc25a6b6a7003a173bfa61cf09a6774c221d8328e5b0c84c6e3bb12ecfd52e8b0"
  },
  "chainId": 80001
}
```

## particle\_aa\_sendUserOp

Request

```JSON
{
  "jsonrpc": "2.0",
  "id": "89916c65-2a47-4933-aa4c-55f7e29edbf0",
  "chainId": 80001,
  "method": "particle_aa_sendUserOp",
  "params": [
    // account config
    {
      "name": "BICONOMY",
      "version": "1.0.0",
      "ownerAddress": "0xA60123a1056e9D38B64c4993615F27cCe9A9E8D5",
      "biconomyApiKey": "LdF-gC43H.6f0ec763-fde5-4d5e-89f6-1b9ef12ba4a4" // optional
    },
    // user op
    {
      "sender": "0x8fb859e944561678be40cdd2db16551396c0b074",
      "nonce": "0x64",
      "initCode": "0x",
      "callData": "0x912ccaa3000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000c0000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000020000000000000000000000009c3c9283d3e44854697cd22d3faa240cfb032889000000000000000000000000329a7f8b91ce7479035cb1b5d62ab41845830ce80000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000c00000000000000000000000000000000000000000000000000000000000000044095ea7b300000000000000000000000000000f7365ca6c59a2c93719ad53d567ed49c14c0000000000000000000000000000000000000000000000000001548a5fd39873000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
      "signature": "0xf173083acfa6ddfdbbb84023ebdeb07ff2a38aa1a2e7f7b1b2eb27e180ff13987c14d72273b82f448b8e89066045aebb9e5423eea3c2314136cc9c75d7bb716f1b",
      "verificationGasLimit": "0x163b8",
      "callGasLimit": "0xd1e7",
      "preVerificationGas": "50736",
      "paymasterAndData": "0x00000f7365ca6c59a2c93719ad53d567ed49c14c000000000000000000000000000000000000000000000000000000000064d2762e0000000000000000000000000000000000000000000000000000000064d26f260000000000000000000000009c3c9283d3e44854697cd22d3faa240cfb03288900000000000000000000000000000f7748595e46527413574a9327942e744e910000000000000000000000000000000000000000000000000de567836c0e69c0000000000000000000000000000000000000000000000000000000000010c8e09292fc583c0427b08765e4eccc030122182d141b6dbc5bb539fc4de726935042106faec9ee5b7537cf15c5f7d317fc355a0ff41877815fcbf9c906e0b584c4b61b",
      "maxFeePerGas": "1500000034",
      "maxPriorityFeePerGas": "1500000000"
    },
    // Optional
    {
      "sessions": [
          {
              "validUntil": 0,
              "validAfter": 0,
              "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
              "sessionKeyDataInAbi": [ // or use sessionKeyData to replace
                  ["address", "address", "address", "uint256"],
                  [
                      "0x1dacDa1087C4048774bEce7784EB8EC4CfBeDB2c",
                      "0x909E30bdBCb728131E3F8d17150eaE740C904649",
                      "0x11D266772b85C2C5D4f84A41ca3E08e9f04Fb5D3",
                      1
                  ]
              ]
          }
      ],
      "targetSession": {
          "validUntil": 0,
          "validAfter": 0,
          "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
          "sessionKeyDataInAbi": [ // or use sessionKeyData to replace
              ["address", "address", "address", "uint256"],
              [
                  "0x1dacDa1087C4048774bEce7784EB8EC4CfBeDB2c",
                  "0x909E30bdBCb728131E3F8d17150eaE740C904649",
                  "0x11D266772b85C2C5D4f84A41ca3E08e9f04Fb5D3",
                  1
              ]
          ]
      }
    }
  ]
}
```

Response

```JSON
{
  "jsonrpc": "2.0",
  "id": "f4702ec0-a0ad-4f20-82b1-302b28223c4c",
  "result": "0x9dae2ad934a6bc3f96ea6bf7fc3230f9c425f84ea99ae64330bdfb13c1b45362",
  "chainId": 80001
}

```

## particle\_aa\_createSessions

Request

```json
{
    "chainId": 80001,
    "jsonrpc": "2.0",
    "id": "f7423e6b-0f69-4b96-8d1e-dcd485f8c2eb",
    "method": "particle_aa_createSessions",
    "params": [
        { "name": "BICONOMY", "version": "2.0.0", "ownerAddress": "0xc19dd1f3e212b39a30036EF3DE3F83dEf5a66E41" },
        [
            {
                "validUntil": 0,
                "validAfter": 0,
                "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
                "sessionKeyDataInAbi": [ // or use sessionKeyData to replace
                    ["address", "address", "address", "uint256"],
                    [
                        "0x1dacDa1087C4048774bEce7784EB8EC4CfBeDB2c",
                        "0x909E30bdBCb728131E3F8d17150eaE740C904649",
                        "0x11D266772b85C2C5D4f84A41ca3E08e9f04Fb5D3",
                        1
                    ]
                ]
            }
        ]
    ]
}
```

Response

```json
{
    "jsonrpc": "2.0",
    "id": "b22300f7-958f-4a7b-8750-e4b67f74fa4c",
    "result": {
        "verifyingPaymasterGasless": {
            "userOp": {
                "sender": "0xcbeba65449cBeF0DcfD4a63CDf4090fA1D4A9F63",
                "nonce": "0x00",
                "initCode": "0x000000a56aaca3e9a4c479ea6b6cd0dbcb6634f5df20ffbc0000000000000000000000000000001c5b32f37f5bea87bdd5374eb2ac54ea8e0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000242ede3bc0000000000000000000000000c19dd1f3e212b39a30036ef3de3f83def5a66e4100000000000000000000000000000000000000000000000000000000",
                "callData": "0x00004680000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000001200000000000000000000000000000000000000000000000000000000000000002000000000000000000000000cbeba65449cbef0dcfd4a63cdf4090fa1d4a9f63000000000000000000000000000002fbffedd9b33f4e7156f2de8d48945e74890000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000024610b5925000000000000000000000000000002fbffedd9b33f4e7156f2de8d48945e74890000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000247cb647596c98955fe6db976fde4318beb22b3aa902fa5de66756158c4f4ef5c012e9055100000000000000000000000000000000000000000000000000000000",
                "paymasterAndData": "0xc89b723809598d0ebf821e89087dc8e1a6ee049900000000000000000000000000000000000000000000000000000000655f45390000000000000000000000000000000000000000000000000000000000000000a774c26e6cb51a0dd3cf973c4536c15fee28e0ce373eba84d5b2a3f7920141b51ce235e4170c5793fabb72f3548548630a8a9137187ce6c7efa9b24230da5f6c1b",
                "signature": "0x00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000001c5b32F37F5beA87BDD5374eB2aC54eA8e000000000000000000000000000000000000000000000000000000000000004181d4b4981670cb18f99f0b4a66446df1bf5b204d24cfcb659bf38ba27a4359b5711649ec2423c5e1247245eba2964679b6a1dbb85c992ae40b9b00c6935b02ff1b00000000000000000000000000000000000000000000000000000000000000",
                "preVerificationGas": "0xd9ec",
                "verificationGasLimit": "0x04db73",
                "callGasLimit": "0x013c73",
                "maxFeePerGas": "0x7558bdca",
                "maxPriorityFeePerGas": "0x7558bdb4"
            },
            "userOpHash": "0x4f09fd29144fd196915f248dffd6d4a496d1cfb20f369a21622ef88ad8010a2b"
        },
        "verifyingPaymasterNative": {
            "userOp": {
                "sender": "0xcbeba65449cBeF0DcfD4a63CDf4090fA1D4A9F63",
                "nonce": "0x00",
                "initCode": "0x000000a56aaca3e9a4c479ea6b6cd0dbcb6634f5df20ffbc0000000000000000000000000000001c5b32f37f5bea87bdd5374eb2ac54ea8e0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000242ede3bc0000000000000000000000000c19dd1f3e212b39a30036ef3de3f83def5a66e4100000000000000000000000000000000000000000000000000000000",
                "callData": "0x00004680000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000001200000000000000000000000000000000000000000000000000000000000000002000000000000000000000000cbeba65449cbef0dcfd4a63cdf4090fa1d4a9f63000000000000000000000000000002fbffedd9b33f4e7156f2de8d48945e74890000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000024610b5925000000000000000000000000000002fbffedd9b33f4e7156f2de8d48945e74890000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000247cb647596c98955fe6db976fde4318beb22b3aa902fa5de66756158c4f4ef5c012e9055100000000000000000000000000000000000000000000000000000000",
                "paymasterAndData": "0x",
                "signature": "0x00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000001c5b32F37F5beA87BDD5374eB2aC54eA8e000000000000000000000000000000000000000000000000000000000000004181d4b4981670cb18f99f0b4a66446df1bf5b204d24cfcb659bf38ba27a4359b5711649ec2423c5e1247245eba2964679b6a1dbb85c992ae40b9b00c6935b02ff1b00000000000000000000000000000000000000000000000000000000000000",
                "preVerificationGas": "0xd9ec",
                "verificationGasLimit": "0x04db73",
                "callGasLimit": "0x013c73",
                "maxFeePerGas": "0x7558bdca",
                "maxPriorityFeePerGas": "0x7558bdb4"
            },
            "userOpHash": "0xf885ae16099dc6bb91d8907200409fd4c0f06a844ff47b87e1a0a7b340cc28e5",
            "feeQuote": {
                "tokenInfo": {
                    "chainId": 80001,
                    "address": "0x0000000000000000000000000000000000000000",
                    "name": "MATIC",
                    "symbol": "MATIC",
                    "decimals": 18,
                    "logoURI": "https://static.particle.network/token-list/polygon/native.png"
                },
                "fee": "896021449333172",
                "balance": "0"
            }
        },
        "tokenPaymaster": {
            "tokenPaymasterAddress": "0x00000f7365cA6C59A2C93719ad53d567ed49c14C",
            "feeQuotes": [
                {
                    "tokenInfo": {
                        "chainId": 80001,
                        "name": "WMATIC",
                        "symbol": "WMATIC",
                        "decimals": 18,
                        "address": "0x9c3C9283D3e44854697Cd22D3Faa240Cfb032889",
                        "logoURI": "https://polygonscan.com/token/images/wMatic_32.png"
                    },
                    "fee": "374428266633331",
                    "balance": "1019119852023946296",
                    "premiumPercentage": "10"
                }
            ]
        },
        "sessions": [ // The client should save the sessions locally
            {
                "validUntil": 0,
                "validAfter": 0,
                "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
                "sessionKeyData": "0x0000000000000000000000001dacda1087c4048774bece7784eb8ec4cfbedb2c000000000000000000000000909e30bdbcb728131e3f8d17150eae740c90464900000000000000000000000011d266772b85c2c5d4f84a41ca3e08e9f04fb5d30000000000000000000000000000000000000000000000000000000000000001"
            }
        ],
        "transactions": [
            {
                "to": "0x0E2e83Bb46ecB097306d98Bf52bdBB992f93BEe9",
                "value": "0x00",
                "data": "0x610b5925000000000000000000000000000002fbffedd9b33f4e7156f2de8d48945e7489"
            },
            {
                "to": "0x000002FbFfedd9B33F4E7156F2DE8D48945E7489",
                "data": "0x7cb647596e8976217c7332d7ed4c3086631c27c8bda59e0c34ba2ebc729c548d4b00badd",
                "value": "0x00"
            }
        ]
    },
    "chainId": 80001
}
```

## particle\_aa\_validateSession

Request

```json
{
    "chainId": 80001,
    "jsonrpc": "2.0",
    "id": "0a7a18a1-53af-45b1-8a7f-4ece06c09e04",
    "method": "particle_aa_validateSession",
    "params": [
        { "name": "BICONOMY", "version": "2.0.0", "ownerAddress": "0xc19dd1f3e212b39a30036EF3DE3F83dEf5a66E41" },
        {
            "sessions": [
                {
                    "validUntil": 0,
                    "validAfter": 0,
                    "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
                    "sessionKeyDataInAbi": [ // or use sessionKeyData to replace
                        ["address", "address", "address", "uint256"],
                        [
                            "0x1dacDa1087C4048774bEce7784EB8EC4CfBeDB2c",
                            "0x909E30bdBCb728131E3F8d17150eaE740C904649",
                            "0x11D266772b85C2C5D4f84A41ca3E08e9f04Fb5D3",
                            1
                        ]
                    ]
                }
            ],
            "targetSession": {
                "validUntil": 0,
                "validAfter": 0,
                "sessionValidationModule": "0x4b7f018Fa27a97b6a17b6d4d8Cb3c0e2D9340133",
                "sessionKeyDataInAbi": [ // or use sessionKeyData to replace
                    ["address", "address", "address", "uint256"],
                    [
                        "0x1dacDa1087C4048774bEce7784EB8EC4CfBeDB2c",
                        "0x909E30bdBCb728131E3F8d17150eaE740C904649",
                        "0x11D266772b85C2C5D4f84A41ca3E08e9f04Fb5D3",
                        1
                    ]
                ]
            }
        }
    ]
}
```

Response

```json
{
    "jsonrpc": "2.0",
    "id": "f45d609c-844d-40ae-9af2-4e89e0baf3c7",
    "result": false,
    "chainId": 80001
}
```
