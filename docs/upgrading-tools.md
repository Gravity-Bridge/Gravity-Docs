# Update the Gravity Bridge tools

The Gravity Bridge tools package contains a number of programs for interacting with the bridge. These are updated much more frequently than the complete chain binary

In order to download ARM binaries change the name in the wget link from ‘gbt' to gbt-arm'

```bash

mkdir gravity-bin
cd gravity-bin

# Tools for the gravity bridge from the gravity repo

wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.0.7/gbt
chmod +x *
sudo mv * /usr/bin/

```

At specific points you may be told to 'update your orchestrator'. In order to do that you can simply repeat the above instructions and then restart the affected software.

to check what version of the tools you have run `gbt --version` the current latest version is `gbt 1.0.7`
