# Error Message Investigation

This repo is for investigating the solution to replicate on-chain code error into off-chain error.

## **The problem**

This problem was first purposed by Peter Drago as issue 
https://github.com/input-output-hk/plutus/issues/3003

## **Possible Solution**

```
first off, we've moved off of PlutusTx on the project I'm on (and switched to Plutarch)
12:15
But secondly, in terms of what the solution would look like:
I'm not sure what the implementation would look like, but basically it would be good to have a more informative error. For instance, if the error is encountered via file Foo.hs:5:10, the error message will currently still print
CallStack (from HasCallStack):
  error, called at src/PlutusTx/Utils.hs:4:26 in plutus-tx-0.1.0.0-6EwthIoMENA5GRJ8xORyCp:PlutusTx.Utils
(edited)
12:18
Which is obviously less helpful than an error message that indicated the file, function, or line number from which it is called
When I talked to some of my team member about this, we floated having a StateT wrapper type around our contracts to maintain a call-stack, so that we could annotate /where/ in the contract things failed. In the mean time, I had just started liberally sprinkling trace statements around all of our off-chain code; it worked somewhat, but its about as ergonomic as relying exclusively on printf for debugging
12:19
But in summary: if you can find a way to make the on-chain error messages be useful off-chain, that'd be a boon for plutustx
```
## **Goal**
 
Investivate the error function, and add a more informative error message function.

Improve from printf and trace functions.

Indicate the file, function, and line number from which the error is called.

## **Restriction**

Not modifying the plutus compiler.

## **Final Solution**

By adding HasCallStack to each functions that may throw exception, GHC.Err can be used to annotate the error with the call stack.

For example:
```
CallStack (from HasCallStack):
  error, called at src/PlutusTx/Utils.hs:8:26 in plutus-tx-0.1.0.0-inplace:PlutusTx.Utils
  mustBeReplaced, called at src/PlutusTx/Builtins/Internal.hs:78:9 in plutus-tx-0.1.0.0-inplace:PlutusTx.Builtins.Internal
  error, called at src/PlutusTx/Builtins.hs:224:11 in plutus-tx-0.1.0.0-inplace:PlutusTx.Builtins
  error, called at src/PlutusTx/Trace.hs:17:18 in plutus-tx-0.1.0.0-inplace:PlutusTx.Trace
  traceError, called at src/PlutusTx/Ratio.hs:335:18 in plutus-tx-0.1.0.0-inplace:PlutusTx.Ratio
  reduce, called at app/Main.hs:20:12 in main:Main
  testFunc, called at app/Main.hs:34:11 in main:Main
  main, called at app/Main.hs:32:1 in main:Main
```

# **On-Chain code testing**

To test if this is change is working with no issue for on-chain code. 