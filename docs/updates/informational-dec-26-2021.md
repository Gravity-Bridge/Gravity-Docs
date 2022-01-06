# Gravity Bridge Development Status update

## Meta

This document is intended as a status update and collected list of suggestions for the Gravity Bridge community. None of these suggestions are the only way forward and the community is encouraged to discuss and develop alternate solutions and proposals.

The information in this document is accurate, to the best of our knowledge as of December 26th 2021

## Delegation Bug, diagnosis and fix

### Issue summary

Quickly after the launch of `gravity-bridge-1` some users reported [issues](https://github.com/Gravity-Bridge/Gravity-Bridge/issues/2) withdrawing their delegated tokens.

This issue has been root caused and resolved in [this patch](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/146b11fcfd8d6c4dcb541507d634839f37e0cce8). The root cause is that during the initialization process for the Gravity Bridge module staking hooks were being set on only one of two copies of the StakingKeeper.

Staking hooks are pieces of code that CosmosSDK calls when events related to staking occur. This issue specifically deals with Staking hooks and Gravity Bridge module Slashing.

Due to an oversight in the Gravity Bridge module, staking hooks were being set on one copy of the StakingKeeper but not on the copy being used by the Gravity Bridge module when performing slashing. This results in several hooks not running when a validator is slashed by the Gravity Bridge module.

In this specific case the hooks take care of recording that a slashing event has occurred in the StakingKeeper, but the actual modification of the user's stake does not depend on the hook.

During the withdrawal of rewards CosmosSDK performs a sanity check, where it ensures that a validator's total rewards for that period are consistent with any slashing events that have occurred. Since the Gravity Bridge module was not triggering the correct hooks in the staking keeper there are no recorded slashing events.

This results in the sanity check failing and an inability to withdraw rewards.

This issue was not detected in our unit tests because they do not use the same initialization environment as the actual chain. We have added an integration test that validates delegating, bonding, and unbonding around Gravity Bridge slashing to prevent such an oversight in the future in addition to correcting the root cause.

### Who is currently affected

The following is output from a tool [located here](https://github.com/jkilpatr/inspect-delegators) designed to identify affected validators and delegators.

```text
Total 309 delegations delegators 192 bugged delegators 17 {
    gravity19uwnjc4fhftygyzr8wv5m8m0l4kjzkpsvsmgqh,
    gravity1xf869399xq4pnxsllzvnwdcra8ly3huj9n2saq,
    gravity1s8zjma74efff5avgz2ah80mxs7gfr6zctmffzp,
    gravity1qtc3axqj2pf9h92arc2lmk7keyvrzqc8y4hef3,
    gravity18u9pws4989n7fmx5pduev7starj8wqggyepcf9,
    gravity1d63hvdgwy64sfex2kyujd0xyfhmu7r7cedj2fg,
    gravity1kp6kn7jazzn6leqvqzdm9ftmlpp72l406fun2r,
    gravity166mwr42yq2khtmwcv9akwjc0f3e62lwslxrkzg,
    gravity1cfj5aksn6tgz5m9885qwp8dwaa844qe2yhlsx7,
    gravity134f6petuvgwtmrpwsfk9ng0fpkmqxqtfkmnq23,
    gravity1vu04rhxr4sq65vfd074m5gjzuggwukzhznzdgz,
    gravity19lhajns3drveve4q8jc4kus6knjcmf2j36ta6e,
    gravity1xmspr43xsfu6ycjgdqqllntc62rzvncv7x8trk,
    gravity104vmnec7p96qgzzdzddv5jan070v7vj6j7mslj,
    gravity1gfeh4793ug8l6nu69n8lqssvkdqs8qghwxka0l,
    gravity12qx44c75dd8ykckexxxxjn7ygehwxkqjxyvzud,
    gravity1lu9p4tl02nl979l9huk33x8rgnwzwmysv2kwsm,
} and bugged validators 16 {
    gravityvaloper1gfeh4793ug8l6nu69n8lqssvkdqs8qghld0r9t,
    gravityvaloper18u9pws4989n7fmx5pduev7starj8wqgg4jcxr3,
    gravityvaloper1vu04rhxr4sq65vfd074m5gjzuggwukzhncmnzk,
    gravityvaloper19lhajns3drveve4q8jc4kus6knjcmf2jq3jrsd,
    gravityvaloper166mwr42yq2khtmwcv9akwjc0f3e62lwswd6ggu,
    gravityvaloper1cfj5aksn6tgz5m9885qwp8dwaa844qe24uxwv2,
    gravityvaloper104vmnec7p96qgzzdzddv5jan070v7vj6r4zw4x,
    gravityvaloper134f6petuvgwtmrpwsfk9ng0fpkmqxqtf8s27q9,
    gravityvaloper12qx44c75dd8ykckexxxxjn7ygehwxkqjh04uke,
    gravityvaloper19uwnjc4fhftygyzr8wv5m8m0l4kjzkpsamzk2r,
    gravityvaloper1xf869399xq4pnxsllzvnwdcra8ly3huj5cnwh5,
    gravityvaloper1d63hvdgwy64sfex2kyujd0xyfhmu7r7cgxt5ru,
    gravityvaloper1kp6kn7jazzn6leqvqzdm9ftmlpp72l40tz9dqh,
    gravityvaloper1qtc3axqj2pf9h92arc2lmk7keyvrzqc847w8r9,
    gravityvaloper1lu9p4tl02nl979l9huk33x8rgnwzwmysap0s60,
    gravityvaloper1xmspr43xsfu6ycjgdqqllntc62rzvncv0d74fz,
} total affected stake 3206902101516
```

As you can see all affected delegations are self-delegations to genesis validators. Each holding `181,818 graviton` with slight
variations based on withdraws and redelegation before their slashing and exactly how many times they have been slashed by the Gravity module.

### Recommended Action

The latest release version of Gravity Bridge `v1.1.0` and greater correctly set the hooks in the Gravity module `app.go`. It is important that the community upgrades to release v1.1.0 or later as any validator slashed by the Gravity Bridge module will be unable to withdraw delegation rewards. Additionally, delegators to such a validator will be unable to withdraw the funds they have delegated to that validator, along with any rewards they have earned from that delegation.

### Resolution for affected individuals

The most correct way to resolve this issue is to insert the correct slashing events into the history during a chain upgrade. This would allow the affected validators to withdraw their rewards with no disruption.

Sadly this option has significant downsides. Since it can only be done during chain upgrade, the upgrade to `v1.1.0` can either be delayed, risking more validators encountering the issue, or the affected users may have to wait until the following upgrade to be able to withdraw their rewards.

Furthermore given the compounding effect of withdrawing and re-delegating, simply allowing affected individuals to withdraw rewards does not compensate for opportunity cost. Resolution to this issue will likely require significant discussion.

Given the risks to the health of the chain, if any additional validators were to be slashed by the Gravity module, the community focus should be on the `v1.1.0` upgrade.

## Other changes included in this upgrade

- [Release images build with Go 1.17](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/e6f0f9ccea3699742bc775239dd64178ce4236af)
- [Correct gbt bug where issues with Ethereum nodes could cause Gravity module slashing](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/d9e5b5174e535bdc92706ce9db981d10f867e5b6)
- [Enable RELRO and PIE for Ledger enabled release binaries](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/89b323e7782217c2aff7022f340d3b502d261b6d)
