Malleability Exercise
=====================

This exercise is designed to help familiarize residents with possible sources of
malleability that have been fixed with the gradual implementation of BIP62. It
will also serve as a jumping off point for writing tests for Bitcoin Core, one
of the best ways to get acquainted with the codebase.

Setup
-----

Clone the Bitcoin Core repository and checkout the v0.18.0 tag.

Apply the following patch:

```diff
diff --git a/src/validation.cpp b/src/validation.cpp
index 0f860a4f1..e573be89b 100644
--- a/src/validation.cpp
+++ b/src/validation.cpp
@@ -1332,7 +1332,7 @@ void UpdateCoins(const CTransaction& tx, CCoinsViewCache& inputs, int nHeight)
 bool CScriptCheck::operator()() {
     const CScript &scriptSig = ptxTo->vin[nIn].scriptSig;
     const CScriptWitness *witness = &ptxTo->vin[nIn].scriptWitness;
-    return VerifyScript(scriptSig, m_tx_out.scriptPubKey, witness, nFlags, CachingTransactionSignatureChecker(ptxTo, nIn, m_tx_out.nValue, cacheStore, *txdata), &error);
+    return VerifyScript(scriptSig, m_tx_out.scriptPubKey, witness, fRequireStandard ? nFlags : nFlags & ~STANDARD_SCRIPT_VERIFY_FLAGS, CachingTransactionSignatureChecker(ptxTo, nIn, m_tx_out.nValue, cacheStore, *txdata), &error);
 }
 
 int GetSpendHeight(const CCoinsViewCache& inputs)
```

Compile Bitcoin Core:
```sh
./autogen.sh
./configure --disable-bench --disable-zmq --without-gui --without-libs --without-miniupnpc
make -j"$(($(nproc)+1))"
```

The exercise
------------

Copy `feature_malleability.py` to `test/functional/` and starting filling it in
with your favourite malleation. There's some boilerplate code in the file to get
you started and some comments you can fill out.

Some malleations that might work well for this exercise:

- extra stack item ignored by multisig (definitely works)
- malleability of failing signature
- inherent ECDSA signature malleability
- push operations in scriptSig of non-standard size type
- zero-padded number pushes
- non-push operations in scriptSig

Run `./test/functional/feature_malleability.py` to test for success.

Note that some of these might require extra modifications to Bitcoin Core to
work (e.g. making `-acceptnonstdtxn=1` work). Since you've compiled once before,
it'll be easy to do.

You might want to take a look at the modules I've imported at the top of the
file for more convenience methods to use.

Debugging tips
--------------

Kalle's [`btcdeb`](https://github.com/kallewoof/btcdeb) might be very useful for
this exercise, as it is a script debugger where you can turn standardness rules
on or off.

If you find yourself needing to modify Bitcoin Core. A good resource is our
[Doxygen](https://dev.visucore.com/bitcoin/doxygen/index.html), however, do keep
in mind that some of the functions are missing from Doxygen, so consider it a
subset of the truth.
