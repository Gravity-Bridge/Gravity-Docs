# Gravity Bridge Proposal
## Software Upgrade to Version 1.2.0

Due to a bug in the Gravity Bridge module, jailed validators were unable to claim rewards after unjailing.

A detailed issue summary can be found at [Issue Summary](https://github.com/Gravity-Bridge/Gravity-Docs/blob/main/docs/updates/informational-dec-26-2021.md) and has been resolved with [this patch](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/146b11fcfd8d6c4dcb541507d634839f37e0cce8).

Shortly after Gravity Bridge launch, [Proposal 2](https://commonwealth.im/gravity-bridge/proposal/cosmosproposal/2-softer-slashingjailing-bug-fix) was passed to mitigate the issue.

With the issue root-caused, verified and fixed, `soma|staking` proposes that we take the upgrade in version `1.2.0` which contains the patch linked above.

Additionally, this version `1.2.0` contains:
* [Upgrade cosmos-sdk version from v0.44.2 to v0.44.5](https://github.com/Gravity-Bridge/Gravity-Bridge/pull/5)
* [Upgrade ibc-go to v2.0.2](https://github.com/Gravity-Bridge/Gravity-Bridge/pull/6)
* [Add authorization (x/authz) module initialization](https://github.com/Gravity-Bridge/Gravity-Bridge/pull/7)
* [Fix invalid deposit issue](https://github.com/Gravity-Bridge/Gravity-Bridge/pull/10)
- [Release images build with Go 1.17](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/e6f0f9ccea3699742bc775239dd64178ce4236af)
- [Correct gbt bug where issues with Ethereum nodes could cause Gravity module slashing](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/d9e5b5174e535bdc92706ce9db981d10f867e5b6)
- [Enable RELRO and PIE for Ledger enabled release binaries](https://github.com/Gravity-Bridge/Gravity-Bridge/commit/89b323e7782217c2aff7022f340d3b502d261b6d)

A YES vote on this proposal indicates the Gravity Bridge should upgrade the chain to take in the bug fix and all the above changes.  Voting NO would result in additional discussions on mitigating the bug as well as taking in the additional changes.

