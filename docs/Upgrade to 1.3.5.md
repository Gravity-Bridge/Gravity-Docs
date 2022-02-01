# Upgrading Gravity Bridge and GBT to 1.3.5

For this binary upgrade there are just a few, relatively straightforward steps.

We will do the following:

 - Stop your node and stop your orchestrator
 - Download new binaries (gravity and GBT)
 - Turn them into executables
 - Put new binaries into the right place
 - Restart your node and orchestrator
 - Some checks to make sure everything is copacetic

<br>

<b>Step 1: Stop your orchestrator and node</b>

```sudo service gravity-node stop```
```sudo service orchestrator stop```

<br>

**Step 2: Go to ``~/gravity-bin`` or any other empty, temporary folder in your home (~) directory**

```cd ~/gravity-bin```

<br>
<b>Step 3: Download, make executable and move the new binaries</b>




- *Starting with Gravity, download the binary*:

```wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.3.5/gravity-linux-amd64```

- *Rename the file "gravity-linux-amd64" to "gravity"*:

```mv gravity-linux-amd64 gravity```

- *Turn it into an executable*

```chmod +x gravity```
<br>


- *Now let's do the same for GBT, first download the binary:*

```wget https://github.com/Gravity-Bridge/Gravity-Bridge/releases/download/v1.3.5/gbt```

- *Turn it into an executable*

```chmod +x gbt```
<br>
**Step 4: Put the binaries where they belong**

- *Move gravity and gbt to /etc/bin*

```sudo mv gravity /etc/bin/```
```sudo mv gbt /etc/bin/```
<br>
**Step 5: Start your node and orchestrator**

```sudo service gravity-node start```
```sudo service orchestrator start```
<br>
**Step 6: Check versions**

```gravity version```
```gbt --version```

- *Both should display version: 1.3.5*

<br>

**You are now essentially done, you can check the following to confirm that your node and orchestrator are happy:**

```journalctl -u gravity-node.service -f --output cat```
 
```journalctl -u orchestrator.service -f --output cat```

```gravity status```
