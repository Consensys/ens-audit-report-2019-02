# Ethlint report

## ethregistrar
```bash
contracts/BaseRegistrarImplementation.sol
  36:36     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  60:42     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  65:15     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  73:16     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  73:48     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  75:23     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  83:39     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  85:15     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  89:47     warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  114:37    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  118:35    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members

contracts/ETHRegistrarController.sol
  53:63    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  54:34    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  61:64    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members
  64:58    warning    Avoid using 'now' (alias to 'block.timestamp').    security/no-block-members

contracts/StringUtils.sol
  15:8     error    Avoid using Inline Assembly.    security/no-inline-assembly
  22:12    error    Avoid using Inline Assembly.    security/no-inline-assembly

✖ 2 errors, 15 warnings found.
```


## Root

```bash
No issues found.
```