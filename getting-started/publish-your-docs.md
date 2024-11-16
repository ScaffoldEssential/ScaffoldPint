---
icon: globe-pointer
---

# Deploy your first smart contract using pint

\
In this example you will get a clear understanding on how to create a simple token smart contract using pint, compile and deploy it on a local network.&#x20;

## Breakdown of essential components in the token smart contract

Here is an explanation of what each function does in the provided token smart contract\
\
1\) Storage Struct

```rust
use std::lib::PredicateAddress;
use std::lib::@delta;
use std::lib::@safe_increment;
use std::lib::@init_once;
use std::lib::@init_delta;
use std::lib::@mut_keys;
use std::auth::@verify_key;
use std::lib::Secp256k1Signature;

storage {
    balances: (b256 => int),
    nonce: (b256 => int),
    token_name: b256,
    token_symbol: b256,
    decimals: int,
}
```

*   **Storage Definitions**:

    * `balances`: A mapping of account addresses (`b256`) to their token balances (`int`).
    * `nonce`: Tracks the number of transactions made by each account to safeguard against replay attacks.
    * `token_name`: A hashed representation of the token's name.
    * `token_symbol`: A hashed representation of the token's symbol.
    * `decimals`: An integer indicating the token's decimal precision.



```rust
interface BurnAccount {
    predicate Owner(
        key: b256,
        amount: int,
        token_address: PredicateAddress,
    );
}

```

**`BurnAccount` Interface**:

* Contains a predicate `Owner` which verifies authorization to burn tokens. It requires:
  * `key`: The account address.
  * `amount`: The number of tokens to burn.
  * `token_address`: The address of the predicate used for verification purposes.

```rust
macro @check_if_predicate_is_owner($c, $p, $address, $arg0) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, { contract: __this_contract_address(), addr: __this_address() })
}

macro @check_if_predicate_is_owner($c, $p, $address, $arg0, $arg1) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, $arg1, { contract: __this_contract_address(), addr: __this_address() })
}

macro @check_if_predicate_is_owner($c, $p, $address, $arg0, $arg1, $arg2) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, $arg1, $arg2, { contract: __this_contract_address(), addr: __this_address() })
}
```

* **Macros:**
  *   Macros `@check_if_predicate_is_owner` are defined to streamline the process of checking if a given predicate is the owner. These macros allow for different numbers of arguments:

      * They evaluate whether a specified contract and predicate address match, utilizing the current contract and its address.
      * The macros can accept varying numbers of arguments (from one to three), enhancing flexibility. These arguments assist in conducting ownership checks in different contexts of authorization.

      These elements work together to ensure secure execution of token-burning functions, while also enabling efficient ownership verification processes.

```rust
union BurnAuth = Signed(Secp256k1Signature) | Predicate(PredicateAddress);
union MintAuth = Signed(Secp256k1Signature) | Predicate(PredicateAddress);
union TransferSignedMode = All | Key | KeyTo | KeyAmount;
type TransferSignedAuth = { sig: Secp256k1Signature, mode: TransferSignedMode };
union TransferAuthMode = Signed(TransferSignedAuth) | Predicate(PredicateAddress);
type Extra = { addr: PredicateAddress };
union ExtraConstraints = Extra(Extra) | None;
type TransferAuth = { mode: TransferAuthMode, extra: ExtraConstraints };
```

* **BurnAuth & MintAuth**: These are union types that determine the authorization method for burning and minting tokens. They can either be signed using a Secp256k1Signature or validated using a PredicateAddress.
* **TransferSignedMode**: An enumeration defining the different modes of signed transfer authorization including options such as All, Key, KeyTo, and KeyAmount.
* **TransferSignedAuth**: A type that combines a Secp256k1Signature and a TransferSignedMode, representing signed authorization for transfers.
* **TransferAuthMode**: A union that encapsulates the method of transfer authorization, accepting either a signed authorization or a predicate-based one.
* **Extra**: A type holding a PredicateAddress, providing additional information for authorization.
* **ExtraConstraints**: A union type that can either be an Extra or None, indicating if there are additional constraints on authorization.
* **TransferAuth**: A type representing the complete transfer authorization details, combining TransferAuthMode and ExtraConstraints for comprehensive transfer verification.

<pre class="language-rust"><code class="lang-rust"><strong>interface MintAccount {
</strong>    predicate Owner(
        key: b256,
        amount: int,
        decimals: int,
        token_address: PredicateAddress,
    );
}


predicate Mint(key: b256, amount: int, decimals: int, auth: MintAuth) {
    let balance = mut storage::balances[key];
    let nonce = mut storage::nonce[key];
    let token_name = mut storage::token_name;
    let token_symbol = mut storage::token_symbol;
    let token_decimals = mut storage::decimals;

    constraint key == config::MINT_KEY;
    constraint @init_once(balance; amount);
    constraint @init_once(token_name; config::NAME);
    constraint @init_once(token_symbol; config::SYMBOL);
    constraint @init_once(token_decimals; decimals);
    constraint @init_once(nonce; 1);

    constraint match auth {
        MintAuth::Signed(sig) => @verify_key({key, amount, decimals, nonce'}; sig; key),
        MintAuth::Predicate(addr) => @check_if_predicate_is_owner(MintAccount; Owner; addr; key; decimals; amount),
    };
}
</code></pre>

The `interface MintAccount` defines a predicate called `Owner` to verify ownership when minting new tokens. It requires certain parameters: a key (`b256`), an amount (`int`), decimals (`int`), and a token address (`PredicateAddress`). This predicate ensures that the entity attempting to mint tokens has legitimate ownership and the right parameters.

The `predicate Mint` function checks if tokens can be minted under specific conditions. It ensures:

* The `key` matches a predefined `MINT_KEY`.
* Various token attributes like `balance`, `token_name`, and `token_symbol` are initialized correctly.
* The `auth` parameter is validated either through a signature or by checking if a predicate is the owner, using the `@check_if_predicate_is_owner` macro.

This setup secures the minting process, ensuring only authorized entities can mint tokens by verifying ownership through signatures or predicates.

```rust
predicate Transfer(key: b256, to: b256, amount: int, auth: TransferAuth) {
  {
    let sender_balance = mut storage::balances[key];
    let receiver_balance = mut storage::balances[to];
    let nonce = mut storage::nonce[key];
    constraint amount > 0;
    constraint sender_balance' >= 0;
    constraint @delta(sender_balance) == 0 - amount;.
    constraint @init_delta(receiver_balance; amount);
    constraint @safe_increment(nonce);
    
   
    constraint match auth.mode {
        TransferAuthMode::Signed(auth) => match auth.mode {
            TransferSignedMode::All => @verify_key({key, to, amount, nonce'}; auth.sig; key),
            TransferSignedMode::Key => @verify_key({key, nonce'}; auth.sig; key),
            TransferSignedMode::KeyTo => @verify_key({key, to, nonce'}; auth.sig; key),
            TransferSignedMode::KeyAmount => @verify_key({key, amount, nonce'}; auth.sig; key),
        },
        TransferAuthMode::Predicate(addr) => @check_if_predicate_is_owner(TransferAccount; Owner; addr; key; to; amount),
    };
    constraint match auth.extra {
        ExtraConstraints::Extra(extra) => ExtraConstraintsI@[extra.addr.contract]::Check@[extra.addr.addr]({ contract: __this_contract_address(), addr: __this_address() }),
        ExtraConstraints::None => true,
    };
    }
    
```

The `Transfer` predicate validates a token transfer between two addresses in a smart contract. It checks several conditions:

1. **Balance checks**: The sender must have enough balance, and the receiver's balance is updated accordingly.
2. **Amount validation**: The transfer amount must be positive.
3. **Nonce management**: The sender’s nonce is safely incremented to prevent replay attacks.
4. **Authentication**: The transaction is authorized either by a digital signature (with various verification modes) or by checking a predicate to verify ownership.
5. **Additional constraints**: If there are any extra constraints (provided in `auth.extra`), they are validated.



