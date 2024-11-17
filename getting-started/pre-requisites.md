---
description: >-
  Pint is Rust Based Smart Contract Language. It means that you need to have
  Rust Installed in order to run Pint smart contracts. Install Rust using the
  below command
---

# Pre-requisites

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Scaffold Essential is built upon Next JS and Node JS. Hence its recommended to have the Node LTS installed \
\
Install pint-cli&#x20;

```bash
cargo install pint-cli
```

Its important to have nix package manager to manage esssential and pint installation

```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install

nix develop github:essential-contributions/essential-integration

nix
```
