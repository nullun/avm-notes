# AVMv12 Fields

## Summary

In AVMv12 two new fields have been introduced to application (`appl`) transactions.
 * `apar` - Reject Version: If the application being called has a version equal-to or greater-than the provided reject version, the transaction will be rejected.
 * `al` - Access List: An array of resources that the application or group can use. Each resource will be one of two types; "simple" or "complex".
   * A simple resource can be either "address" (`d`), "asset" (`s`), or "app" (`p`). Where each resource includes a single reference.
   * A complex resource can be either "holding" (`h`), "locals" (`l`), or "box" (`b`). Where each resource includes up to two 1-based indexes to simple resources included in the access list. An index of zero (`0`) has a special meaning of "current sender address" or "current application id", and would also be omitted from the structure.

In addition to the new fields, the maximum number of resources that can be included on a transaction is twice the maximum number of foreign references a transaction had previously been able to use.
 * `v24.MaxAppTotalTxnReferences = 8` - Old value, used for foreign references.
 * `v41.MaxAppAccess = 16` - Higher limit, when using the Access List.

However the number of accounts that can be included in the old foreign references array has been increased to 8. Matching apps, assets, and boxes.
 * `v41.MaxAppTxnAccounts = 8` - Increased from 4.

The new Access List and existing Foreign Reference arrays are mutually exclusive, and cannot be used together on the same transaction.

Note that when using foreign references, you are automatically getting the cross-product of provided references. For example, by including account A and asset B, you'd be able to lookup the balance of asset B on account A. However when using the Access List, you must explicitly include account A, asset B, and then a holding reference pointing to the relevant address and asset indexes within the array.

## Examples

For brevity I will only show the full transaction structure once per section. For subsequent examples, I will only include the fields of interest.

### Reject Version

```jsonc
{
  "apid": 1005,
  "aprv": 3, // Reject if application 1005 version is 3 or more
  "fee": 1000,
  "fv": 931,
  "gh": "t9fO3Zr2fsmd8Dg+0HkTKwX9dkf73CViBarLDH2hLtw=",
  "lv": 1931,
  "snd": "ALICE7Y2JOFGG2VGUC64VINB75PI56O6M2XW233KG2I3AIYJFUD4QMYTJM",
  "type": "appl"
}
```

### Access List

#### Simple Resources

```jsonc
{
  "al": [ // Access List array
    { // Address reference
      "d": "BOBBYB3QD5QGQ27EBYHHUT7J76EWXKFOSF2NNYYYI6EOAQ5D3M2YW2UGEA"
    }
  ],
  "apid": 1005,
  "fee": 1000,
  "fv": 931,
  "gh": "t9fO3Zr2fsmd8Dg+0HkTKwX9dkf73CViBarLDH2hLtw=",
  "lv": 1931,
  "snd": "ALICE7Y2JOFGG2VGUC64VINB75PI56O6M2XW233KG2I3AIYJFUD4QMYTJM",
  "type": "appl"
}
```

```jsonc
{
// ...
  "al": [
    { // Asset ID reference
      "s": 1010
    }
  ],
// ...
}
```

```jsonc
{
// ...
  "al": [
    { // Application ID reference
      "p": 1042
    }
  ],
// ...
}
```

#### Complex Resources

```jsonc
{
// ...
  "al": [
    { // Address reference
      "d": "BOBBYB3QD5QGQ27EBYHHUT7J76EWXKFOSF2NNYYYI6EOAQ5D3M2YW2UGEA"
    },
    { // Asset ID reference
      "s": 1010
    },
    { // Holding Resource
      "h": {
        "d": 1, // 1-based Index to Address resource
        "s": 2  // 1-based Index to Asset resource
      }
    }
  ],
// ...
}
```

```jsonc
{
// ...
    "al": [
      { // Application ID reference
        "p": 1042
      },
      { // Address reference
        "d": "BOBBYB3QD5QGQ27EBYHHUT7J76EWXKFOSF2NNYYYI6EOAQ5D3M2YW2UGEA"
      },
      { // Locals Resource
        "l": {
          "d": 2, // 1-based Index to Address resource
          "p": 1  // 1-based Index to Application resource
        }
      }
    ],
// ...
}
```

```jsonc
{
// ...
    "al": [
      { // Application ID reference
        "p": 1042
      },
      { // Box Resource
        "b": {
          "i": 1, // 1-based Index to Application resource
          "n": "Ym94TmFtZQ==" // Base64 encoded Box name
        }
      }
    ],
// ...
}
```
