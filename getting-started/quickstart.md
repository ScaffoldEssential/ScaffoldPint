---
icon: bullseye-arrow
---

# Quickstart

<figure><img src="../.gitbook/assets/Screenshot 2024-11-16 at 4.42.33 PM.png" alt="" width="375"><figcaption></figcaption></figure>

Essential is the first declarative blockchain protocol. The protocol implements the declarative principles described in the above sections in a decentralized, permissionless system, architected around the idea of blocks as pure state mutations.

{% hint style="info" %}
Get Started by deploying your first pint smart contract using the ts-sdk
{% endhint %}

### Install Pint&#x20;

This command will install all the necessary files needed such as nix, essential-integration, pint-cli etc.

```
npx scaffold-essential-cli
```

### Compile your pint contracts

```
npm run compile
```

### Download any new updates from essential-integration package ( if any )

```
npm run shell
```

### Deploy your contract locally on the respective ports ( for eg :- builder port 3554 )

```
npm run chain
```
