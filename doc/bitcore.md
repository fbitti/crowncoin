Bitcore Integration
===================

We are implementing bitcore features in this repo by researching the difference between the official bitcoin and bitpay-bitcoin. See:

https://github.com/bitcoin/bitcoin/compare/9779e1e...bitpay:597c9b4

Basically, it adds "address index" feature.

But the difficulty of integrating it into Crown is that Crown is based on v0.10, while bitpay-bitcoin is based on v0.12. Notable changes:

All files in `qa/rpc-tests` are not added yet, because the test framework seems different.

In v0.10 there's no `src/dbwrapper.cpp` but `src/leveldbwrapper.cpp`, so all changes in `dbwrapper.cpp` should be reflected in `leveldbwrapper.cpp`. `CDBWrapper` also maps to `CLevelDBWrapper`. I added `CLevelDBIterator` (from `CDBIterator` in v0.12) because `src/txdb.cpp` will use it.

Obfuscation isn't supported in v0.10. So bitpay-bitcoin's obfuscation-related code should be removed. `CLevelDBWrapper` function shouldn't take `obfuscate` parameter. In `CLevelDBIterator` class `obfuscate_key` should be removed.

In `src/main.cpp`, v0.10 uses `state.Abort`, but v0.12 doesn't use this -- it uses `AbortNode`. My inserted code uses `AbortNode`.

`GetTransaction` function in v0.10 takes 4 parameters. In v0.12 it takes 5 parameters. My inserted code uses 4 parameters.

In `src/rpcserver.cpp`, each element in `vRPCCommands` array has 2 additional booleans (`threadSafe` and `reqWallet`) in v0.10, but v0.12 lacks them. My inserted code has those booleans. I set them to all `false` -- maybe that's incorrect.

`src/serialized.h` isn't changed because I can't find any existing function names starting with `ser_`. That may be incorrect.

`src/test/data/wallet.dat` wasn't changed because changing it needs further knowledge. It needs change.

For `src/test/dbwrapper_tests.cpp`, v0.10 doesn't have this file, so I also didn't create this file. But that may be incorrect.

Note: v0.10 and v0.12 in this article mean the versions of Bitcoin Core, not Crown Core.

Publish
=======

First, build "depends":

```bash
cd depends && make && cd ..
```

Then build Crown Core. Please note that while typing "./configure", it must be in the form of:

```
./configure --disable-tests --prefix=<depends-absolute-path>
```

`<depends-absolute-path>` must be an absolute path where the "depends" binaries are just made, for example:

```
/home/zzz/projects/crowncoin/depends/x86_64-unknown-linux-gnu/
```

After building Crown Core, type:

```bash
strip src/crownd
```

Create a "crownd.tar.gz" file containing this "crownd" by typing:

```bash
tar -cvzf crownd.tar.gz -C src crownd
```

Create a Git tag `bitcore-bin-<xxx>` where `<xxx>` is a 3-digit number:

```
git tag bitcore-bin-<xxx>
```

Then type:

```bash
git push --tags
```

In github.com, Upload crownd.tar.gz to the assets of this tag (release).
