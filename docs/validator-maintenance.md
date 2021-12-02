# Validator Maintenance

This document contains instructions for routine operations that may be required while operating a validator.

## Index

1. [Increase your stake](#increase-your-stake)
1. [Unjail your validator](#unjail-your-validator)

## Increase your stake

To increase your ugraviton stake, if you have extra tokens lying around. The first command will show an output like this, you want to take the key starting with gravityvaloper1 in the 'address' field.

```text
- name: jkilpatr
  type: local
  address: gravityvaloper1jpz0ahls2chajf78nkqczdwwuqcu97w6z3plt4
  pubkey: gravityvaloperpub1addwnpepqvl0qgfqewmuqvyaskmr4pwkr5fwzuk8286umwrfnxqkgqceg6ksu359m5q
  mnemonic: ""
  threshold: 0
  pubkeys: []

```

```bash
gravity keys show myvalidatorkeyname --bech val
gravity tx staking delegate <the address from the above command> 99000000ualtg --from myvalidatorkeyname --chain-id althea-testnet2v3 --fees 1altg --broadcast-mode block
```

## Unjail your validator

This command will unjail you, completing the process of getting the chain back online!

replace 'myvalidatorkeyname' with your validator keys name, if you don't remember run `gravity keys list`

```bash
gravity tx slashing unjail --from myvalidatorkeyname --chain-id=gravity-bridge-test1
```
