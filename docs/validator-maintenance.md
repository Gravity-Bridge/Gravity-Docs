# Validator Maintenance

This document contains instructions for routine operations that may be required while operating a validator.

## Index

1. [Increase your stake](#increase-your-stake)
1. [Withdraw your validator rewards](#withdraw-your-validator-rewards)
1. [Unjail your validator](#unjail-your-validator)
1. [Change validator identity (profile pic)](#update-your-validator-identity)

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
gravity tx staking delegate <the address from the above command> 99000000ugraviton --from myvalidatorkeyname --chain-id gravity-bridge-3 --fees 1ugraviton --broadcast-mode block
```

## Withdraw your validator rewards

In order to withdraw your validation rewards run the following command. This will withdraw all the rewards
from your validator into your validator key. In order to modify the destination you can run `set-withdraw-addr`

In order to apply your rewards to stake you must run this command followed by the `staking delegate` command

```bash
gravity tx distribution withdraw-all-rewards --from <validator-key-name> --chain-id gravity-bridge-3
```

## Unjail your validator

This command will unjail you, completing the process of getting the chain back online!

replace 'myvalidatorkeyname' with your validator keys name, if you don't remember run `gravity keys list`

```bash
gravity tx slashing unjail --from myvalidatorkeyname --chain-id=gravity-bridge-3
```

## Update your validator identity

Want your validator to have that sweet, sweet image?  The following will use your profile image from keybase.io, I believe there is another data source as well, but this example is for keybase.

```bash
gravity tx staking edit-validator --moniker <moniker> --chain-id=gravity-bridge-3 --fees 1ugraviton --identity "<your keybase pgp key>" --from <validator-name>
```
