# Send a Transaction

To send a transaction using the provided code, follow these steps:

**Choose the Adapter:** Choose the appropriate adapter based on your scenario. The `adapterName` variable should be set to the desired adapter's name, such as "MetaMask," "Trust," "BitKeep," or any other supported wallet.

<pre class="language-kotlin"><code class="lang-kotlin">// Choose your adapter based on the scenario (e.g., MetaMask, Trust, BitKeep)
<strong>val adapterName = MobileWCWalletName.Particle.name 
</strong>val adapter = ParticleConnect.getAdapters().first {
    it.name == adapterName
}
</code></pre>

**Prepare Transaction Data:** Refer to the [Transaction Construction](transaction-construction.md) Guide . It can be in the form of a hex string for EVM (Ethereum Virtual Machine) transactions or a Base58 string for Solana transactions.

```kotlin
val trans ="Replace this with the constructed transaction data (hex for EVM or Base58 for Solana)"
```

**Get Wallet Address:** Obtain the wallet address after the user has logged in.

```kotlin
val address = "Replace this with the wallet address obtained after login"
```

**Sign and Send Transaction:** Use the obtained adapter to sign and send the transaction.

```kotlin
adapter!!.signAndSendTransaction(address, trans, object : TransactionCallback {
    override fun onError(error: ConnectError) {
        // Handle transaction sending error
        // This block will be executed if there is an error sending the transaction
    }
​
    override fun onTransaction(transactionId: String?) {
        // Handle successful transaction
        // This block will be executed if the transaction is sent successfully
    }
})
```

* The `signAndSendTransaction` method takes the wallet address, transaction data, and a callback for handling errors and successful transactions.
* If an error occurs during the transaction, the `onError` method will be called, and if the transaction is successful, the `onTransaction` method will be called with the transaction ID.

\
