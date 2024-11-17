# Write a simple counter smart contract

In the backend/src/contract.pnt directory you'll have a simple token smart contract with burn, mint functionality

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

interface BurnAccount {
    predicate Owner(
        key: b256,
        amount: int,
        token_address: PredicateAddress,
    );
}

union BurnAuth = Signed(Secp256k1Signature) | Predicate(PredicateAddress);

predicate Burn(key: b256, amount: int, auth: BurnAuth) {
    let balance = mut storage::balances[key];
    let nonce = mut storage::nonce[key];

    constraint amount > 0;
    constraint @delta(balance) == 0 - amount;
    constraint balance' >= 0;
    constraint @safe_increment(nonce);
    
    constraint match auth {
        BurnAuth::Signed(sig) => @verify_key({key, amount, nonce'}; sig; key),
        BurnAuth::Predicate(addr) => @check_if_predicate_is_owner(BurnAccount; Owner; addr; key; amount),
    };
}

macro @check_if_predicate_is_owner($c, $p, $address, $arg0) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, { contract: __this_contract_address(), addr: __this_address() })
}

macro @check_if_predicate_is_owner($c, $p, $address, $arg0, $arg1) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, $arg1, { contract: __this_contract_address(), addr: __this_address() })
}

macro @check_if_predicate_is_owner($c, $p, $address, $arg0, $arg1, $arg2) {
    $c@[$address.contract]::$p@[$address.addr]($arg0, $arg1, $arg2, { contract: __this_contract_address(), addr: __this_address() })
}

union MintAuth = Signed(Secp256k1Signature) | Predicate(PredicateAddress);

interface MintAccount {
    predicate Owner(
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

union TransferSignedMode = All | Key | KeyTo | KeyAmount;

type TransferSignedAuth = { sig: Secp256k1Signature, mode: TransferSignedMode };

union TransferAuthMode = Signed(TransferSignedAuth) | Predicate(PredicateAddress);

type Extra = { addr: PredicateAddress };

union ExtraConstraints = Extra(Extra) | None;

type TransferAuth = { mode: TransferAuthMode, extra: ExtraConstraints };

interface TransferAccount {
    predicate Owner(
        key: b256,
        to: b256,
        amount: int,
        token_address: PredicateAddress,
    );
}

interface ExtraConstraintsI {
    predicate Check(
        token_address: PredicateAddress,
    );
}

predicate Transfer(key: b256, to: b256, amount: int, auth: TransferAuth) {
    let sender_balance = mut storage::balances[key];
    let receiver_balance = mut storage::balances[to];
    let nonce = mut storage::nonce[key];

    constraint amount > 0;
    constraint sender_balance' >= 0;
    constraint @delta(sender_balance) == 0 - amount;
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

interface CancelAccount {
    predicate Owner(
        key: b256,
        token_address: PredicateAddress,
    );
}

union CancelAuth = Signed(Secp256k1Signature) | Predicate(PredicateAddress);

predicate Cancel(key: b256, auth: CancelAuth) {
    let nonce = mut storage::nonce[key];

    constraint @safe_increment(nonce);

    constraint match auth {
        CancelAuth::Signed(sig) => @verify_key({key, nonce'}; sig; key),
        CancelAuth::Predicate(addr) => @check_if_predicate_is_owner(CancelAccount; Owner; addr; key),
    };
}

```

We'll now compile the contract and deploy it locally.\
\
To compile :- Head over\
