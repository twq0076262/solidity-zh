# 常见问题

This list was originally compiled by [fivedogit]([mailto:fivedogit@](mailto:fivedogit@gmail.com))[gmail.com)](mailto:fivedogit@gmail.com)).

## 基础问题

Solidity是什么？

Solidity是受Javascript启发的编程语言，可以被用来在以太坊区块链上创建智能合约。还有其它编程语言（LLL，Serpent等）也可以创建智能合约。Solidity更被开发者喜爱的主要原因是它是静态类型语言，提供许多高级特性，例如继承、函数库、用户定义的复杂类型和字节码优化。

Solidity合约能够以不同方式被编译（见下文），输出结果可以被剪贴/复制到一个geth控制台，然后被部署到以太坊区块链。

Fivedogit写了一些[合约例子](https://github.com/fivedogit/solidity-baby-steps/tree/master/contracts/)，Solidity的每个特性都应该有对应的[测试合约](https://github.com/ethereum/solidity/blob/develop/test/libsolidity/SolidityEndToEndTest.cpp)。

怎么编译合约？

最快的方式可能是使用[线上编译器](https://chriseth.github.io/browser-solidity/)。

你也可以使用cpp-ethereum自带的solc binary编译合约，或者使用集成开发环境Mix。

创建和发布最基础的合约

一个非常简单的合约是[greeter](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol)。

在特定区块上做一些事情 （例如发布一个合约或者执行一笔交易）是可能的吗？

不能保证交易在下一个区块或未来特定区块中发生，因为这取决于打包交易的矿工，而不是交易的提交者。  如果你想计划将来的合约调用，你可以使用[alarm clock](http://www.ethereum-alarm-clock.com/)。

交易“负载”（payload）是什么？

它只是与请求一起发送的字节码“数据”。

现在有可用的反编译器吗？

There is no decompiler to Solidity. This is in principle possible to some degree, but for example variable names will be lost and great effort will be necessary to make it look similar to the original source code.

Bytecode can be decompiled to opcodes, a service that is provided by several blockchain explorers.

Contracts on the blockchain should have their original source code published if they are to be used by third parties.

Does selfdestruct() free up space in the blockchain?

It removes the contract bytecode and storage from the current block into the future, but since the blockchain stores every single block (i.e. all history), this will not actually free up space on full/achive nodes.

Create a contract that can be killed and return funds

First, a word of warning: Killing contracts sounds like a good idea, because “cleaning up” is always good, but as seen above, it does not really clean up. Furthermore, if Ether is sent to removed contracts, the Ether will be forever lost.

If you want to deactivate your contracts, rather **disable** them by changing some internal state which causes all functions to throw. This will make it impossible to use the contract and ether sent to the contract will be returned automatically.

Now to answering the question: Inside a constructor, msg.sender is the creator. Save it. Thenselfdestruct(creator); to kill and return funds.

[example](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/05_greeter.sol)

Note that if you import “mortal” at the top of your contracts and declare contract SomeContract is mortal { ... and compile with a compiler that already has it (which includes [browser-solidity](https://chriseth.github.io/browser-solidity/)), thenkill() is taken care of for you. Once a contract is “mortal”, then you cancontractname.kill.sendTransaction({from:eth.coinbase}), just the same as my examples.

Store Ether in a contract

The trick is to create the contract with {from:someaddress, value: web3.toWei(3,”ether”)...}

See [endowment_retriever.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol).

Use a non-constant function (req sendTransaction) to increment a variable in a contract

See [value_incrementer.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/20_value_incrementer.sol).

Get contract address in Solidity

Short answer: The global variable this is the contract address.

See [basic_info_getter](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/15_basic_info_getter.sol).

Long answer: this is a variable representing the current contract. Its type is the type of the contract. Since any contract type basically inherits from the address type, this is always convertible to addressand in this case contains its own address.

What is the difference between a function marked constant and one that is not?

constant functions can perform some action and return a value, but cannot change state (this is not yet enforced by the compiler). In other words, a constant function cannot save or update any variables within the contract or wider blockchain. These functions are called usingc.someFunction(...) from geth or any other web3.js environment.

“non-constant” functions (those lacking the constant specifier) must be called withc.someMethod.sendTransaction({from:eth.accounts[x], gas: 1000000}); That is, because they can change state, they have to have a gas payment sent along to get the work done.

Get a contract to return its funds to you (not using selfdestruct(...)).

This example demonstrates how to send funds from a contract to an address.

See [endowment_retriever](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/30_endowment_retriever.sol).

What is a mapping and how do we use them?

A mapping is very similar to a K->V hashmap. If you have a state variable of type mapping (string -> uint) x;, then you can access the value by x[“somekeystring”].

How can I get the length of a mapping?

Mappings are a rather low-level data structure. It does not store the keys and it is not possible to know which or how many values are “set”. Actually, all values to all possible keys are set by default, they are just initialised with the zero value.

In this sense, the attribute length for a mapping does not really apply.

If you want to have a “sized mapping”, you can use the iterable mapping (see below) or just a dynamically-sized array of structs.

Are mappings iterable?

Mappings themselves are not iterable, but you can use a higher-level datastructure on top of it, for example the [iterable mapping](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol).

Can you return an array or a string from a solidity function call?

Yes. See [array_receiver_and_returner.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/60_array_receiver_and_returner.sol).

What is problematic, though, is returning any variably-sized data (e.g. a variably-sized array likeuint[]) from a fuction **called from within Solidity**. This is a limitation of the EVM and will be solved with the next protocol update.

Returning variably-sized data as part of an external transaction or call is fine.

How do you represent double/float in Solidity?

This is not yet possible.

Is it possible to in-line initialize an array like so: string32[] myarray = [“a”, “b”];

This is not yet possible.

What are events and why do we need them?

Let us suppose that you need a contract to alert the outside world when something happens. The contract can fire an event, which can be listened to from web3 (inside geth or a web application). The main advantage of events is that they are stored in a special way on the blockchain so that it is very easy to search for them.

What are the different function visibilities?

The visibility specifiers do not only change the visibility but also the way functions can be called. In general, functions in the same contract can also be called internally (which is cheaper and allows for memory types to be passed by reference). This is done if you just use f(1,2). If you use this.f(1,2) orotherContract.f(1,2), the function is called externally.

Internal function calls have the advantage that you can use all Solidity types as parameters, but you have to stick to the simpler ABI types for external calls.

- external: all, only externally


- public: all (this is the default), externally and internally


- internal: only this contract and contracts deriving from it, only internally


- private: only this contract, only internally

Do contract constructors have to be publicly visible?

You can use the visibility specifiers, but they do not yet have any effect. The constructor is removed from the contract code once it is deployed,

Can a contract have multiple constructors?

No, a contract can have only one constructor.

More specifically, it can only have one function whose name matches that of the constructor.

Having multiple constructors with different number of arguments or argument types, as it is possible in other languages is not allowed in Solidity.

Is a constructor required?

No. If there is no constructor, a generic one without arguments and no actions will be used.

Are timestamps (now, block.timestamp) reliable?

This depends on what you mean by “reliable”. In general, they are supplied by miners and are therefore vulnerable.

Unless someone really messes up the blockchain or the clock on your computer, you can make the following assumptions:

You publish a transaction at a time X, this transaction contains same code that calls now and is included in a block whose timestamp is Y and this block is included into the canonical chain (published) at a time Z.

The value of now will be identical to Y and X <= Y <= Z.

Never use now or block.hash as a source of randomness, unless you know what you are doing!

Can a contract function return a struct?

Yes, but only in “internal” function calls.

If I return an enum, I only get integer values in web3.js. How to get the named values?

Enums are not supported by the ABI, they are just supported by Solidity. You have to do the mapping yourself for now, we might provide some help later.

What is the deal with “function () { ... }” inside Solidity contracts? How can a function not have a name?

This function is called “fallback function” and it is called when someone just sent Ether to the contract without providing any data or if someone messed up the types so that they tried to call a function that does not exist.

The default behaviour (if no fallback function is explicitly given) in these situations is to just accept the call and do nothing. This is desireable in many cases, but should only be used if there is a way to pull out Ether from a contract.

If the contract is not meant to receive Ether with simple transfers, you should implement the fallback function as

function() { throw; }

this will cause all transactions to this contract that do not call an existing function to be reverted, so that all Ether is sent back.

Another use of the fallback function is to e.g. register that your contract received ether by using an event.

*Attention*: If you implement the fallback function take care that it uses as little gas as possible, because send() will only supply a limited amount.

Is it possible to pass arguments to the fallback function?

The fallback function cannot take parameters.

Under special circumstances, you can send data. If you take care that none of the other functions is invoked, you can access the data by msg.data.

Can state variables be initialized in-line?

Yes, this is possible for all types (even for structs). However, for arrays it should be noted that you must declare them as static memory arrays.
Examples:

```
contract C {

    struct S { uint a; uint b; }

    S public x = S(1, 2);

    string name = "Ada";

    string[4] memory AdaArr = ["This", "is", "an", "array"];}contract D {

    C c = new C();}
```

What is the “modifier” keyword?

Modifiers are a way to prepend or append code to a function in order to add guards, initialisation or cleanup functionality in a concise way.

For examples, see the [features.sol](https://github.com/ethereum/dapp-bin/blob/master/library/features.sol).

How do structs work?

See [struct_and_for_loop_tester.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol).

How do for loops work?

Very similar to JavaScript. There is one point to watch out for, though:

If you use for (var i = 0; i < a.length; i ++) { a[i] = i; }, then the type of i will be inferred only from 0, whose type is uint8. This means that if a has more than 255 elements, your loop will not terminate because i can only hold values up to 255.

Better use for (uint i = 0; i < a.length...

See [struct_and_for_loop_tester.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/65_struct_and_for_loop_tester.sol).

What character set does Solidity use?

Solidity is character set agnostic concerning strings in the source code, although utf-8 is recommended. Identifiers (variables, functions, ...) can only use ASCII.

What are some examples of basic string manipulation (substring, indexOf, charAt, etc)?

There are some string utility functions at [stringUtils.sol](https://github.com/ethereum/dapp-bin/blob/master/library/stringUtils.sol) which will be extended in the future.

For now, if you want to modify a string (even when you only want to know its length), you should always convert it to a bytes first:

```
contract C {

    string s;

    function append(byte c) {

        bytes(s).push(c);

    }

    function set(uint i, byte c) {

        bytes(s)[i] = c;

    }}
```

Can I concatenate two strings?

You have to do it manually for now.

Why is the low-level function .call() less favorable than instantiating a contract with a variable (ContractB b;) and executing its functions (b.doSomething();)?

If you use actual functions, the compiler will tell you if the types or your arguments do not match, if the function does not exist or is not visible and it will do the packing of the arguments for you.

See [ping.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol) and [pong.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol).

Is unused gas automatically refunded?

Yes and it is immediate, i.e. done as part of the transaction.

When returning a value of say “uint” type, is it possible to return an “undefined” or “null”-like value?

This is not possible, because all types use up the full value range.

You have the option to throw on error, which will also revert the whole transaction, which might be a good idea if you ran into an unexpected situation.

If you do not want to throw, you can return a pair:

```
contract C {

    uint[] counters;

    function getCounter(uint index)

        returns (uint counter, bool error) {

            if (index >= counters.length) return (0, true);

            else return (counters[index], false);

        }

    function checkCounter(uint index) {

        var (counter, error) = getCounter(index);

        if (error) { ... }

        else { ... }

    }}
```

Are comments included with deployed contracts and do they increase deployment gas?

No, everything that is not needed for execution is removed during compilation. This includes, among others, comments, variable names and type names.

What happens if you send ether along with a function call to a contract?

It gets added to the total balance of the contract, just like when you send ether when creating a contract.

Is it possible to get a tx receipt for a transaction executed contract-to-contract?

No, a function call from one contract to another does not create its own transaction, you have to look in the overall transaction. This is also the reason why several block explorer do not show Ether sent between contracts correctly.

What is the memory keyword? What does it do?

The Ethereum Virtual Machine has three areas where it can store items.

The first is “storage”, where all the contract state variables reside. Every contract has its own storage and it is persistent between function calls and quite expensive to use.

The second is “memory”, this is used to hold temporary values. It is erased between (external) function calls and is cheaper to use.

The third one is the stack, which is used to hold small local variables. It is almost free to use, but can only hold a limited amount of values.

For almost all types, you cannot specify where they should be stored, because they are copied everytime they are used.

The types where the so-called storage location is important are structs and arrays. If you e.g. pass such variables in function calls, their data is not copied if it can stay in memory or stay in storage. This means that you can modify their content in the called function and these modifications will still be visible in the caller.

There are defaults for the storage location depending on which type of variable it concerns:

- state variables are always in storage


- function arguments are always in memory


- local variables always reference storage

Example:

```
contract C {

    uint[] data1;

    uint[] data2;

    function appendOne() {

        append(data1);

    }

    function appendTwo() {

        append(data2);

    }

    function append(uint[] storage d) {

        d.push(1);

    }}
```

The function append can work both on data1 and data2 and its modifications will be stored permanently. If you remove the storage keyword, the default is to use memory for function arguments. This has the effect that at the point where append(data1) or append(data2) is called, an independent copy of the state variable is created in memory and append operates on this copy (which does not support .push - but that is another issue). The modifications to this independent copy do not carry back to data1 or data2.

A common mistake is to declare a local variable and assume that it will be created in memory, although it will be created in storage:

```
/// THIS CONTRACT CONTAINS AN ERRORcontract C {

    uint someVariable;

    uint[] data;

    function f() {

        uint[] x;

        x.push(2);

        data = x;

    }}
```

The type of the local variable x is uint[] storage, but since storage is not dynamically allocated, it has to be assigned from a state variable before it can be used. So no space in storage will be allocated forx, but instead it functions only as an alias for a pre-existing variable in storage.

What will happen is that the compiler interprets x as a storage pointer and will make it point to the storage slot 0 by default. This has the effect that someVariable (which resides at storage slot 0) is modified by x.push(2).

The correct way to do this is the following:

```
contract C {

    uint someVariable;

    uint[] data;

    function f() {

        uint[] x = data;

        x.push(2);

    }}
```

Can a regular (i.e. non-contract) ethereum account be closed permanently like a contract can?

No. Non-contract accounts “exist” as long as the private key is known by someone or can be generated in some way.

What is the difference between bytes and byte[]?

bytes is usually more efficient: When used as arguments to functions (i.e. in CALLDATA) or in memory, every single element of a byte[] is padded to 32 bytes which wastes 31 bytes per element.

Is it possible to send a value while calling an overloaded function?

It’s a known missing feature. [https://www.pivotaltracker.com/story/show/92020468](https://www.pivotaltracker.com/story/show/92020468) as part of[https://www.pivotaltracker.com/n/projects/1189488](https://www.pivotaltracker.com/n/projects/1189488)

Best solution currently see is to introduce a special case for gas and value and just re-check whether they are present at the point of overload resolution.

Advanced Questions

How do you get a random number in a contract? (Implement a self-returning gambling contract.)

Getting randomness right is often the crucial part in a crypto project and most failures result from bad random number generators.

If you do not want it to be safe, you build something similar to the [coin flipper](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/35_coin_flipper.sol) but otherwise, rather use a contract that supplies randomness, like the [RANDAO](https://github.com/randao/randao).

Get return value from non-constant function from another contract

The key point is that the calling contract needs to know about the function it intends to call.

See [ping.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_ping.sol) and [pong.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/45_pong.sol).

Get contract to do something when it is first mined

Use the constructor. Anything inside it will be executed when the contract is first mined.

See [replicator.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol).

Can a contract create another contract?

Yes, see [replicator.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/50_replicator.sol).

Note that the full code of the created contract has to be included in the creator contract. This also means that cyclic creations are not possible (because the contract would have to contain its own code) - at least not in a general way.

How do you create 2-dimensional arrays?

See [2D_array.sol](https://github.com/fivedogit/solidity-baby-steps/blob/master/contracts/55_2D_array.sol).

Note that filling a 10x10 square of uint8 + contract creation took more than 800,000 gas at the time of this writing. 17x17 took 2,000,000 gas. With the limit at 3.14 million... well, there’s a pretty low ceiling for what you can create right now.

Note that merely “creating” the array is free, the costs are in filling it.

Note2: Optimizing storage access can pull the gas costs down considerably, because 32 uint8 values can be stored in a single slot. The problem is that these optimizations currently do not work across loops and also have a problem with bounds checking. You might get much better results in the future, though.

What does p.recipient.call.value(p.amount)(p.data) do?

Every external function call in Solidity can be modified in two ways:

1. You can add Ether together with the call


1. You can limit the amount of gas available to the call

This is done by “calling a function on the function”:

f.gas(2).value(20)() calls the modified function f and thereby sending 20 Wei and limiting the gas to 2 (so this function call will most likely go out of gas and return your 20 Wei).

In the above example, the low-level function call is used to invoke another contract with p.data as payload and p.amount Wei is sent with that call.

Can a contract function accept a two-dimensional array?

This is not yet implemented for external calls and dynamic arrays - you can only use one level of dynamic arrays.

What is the relationship between bytes32 and string? Why is it that ‘bytes32 somevar = “stringliteral”;’ works and what does the saved 32-byte hex value mean?

The type bytes32 can hold 32 (raw) bytes. In the assignment bytes32 samevar = “stringliteral”;, the string literal is interpreted in its raw byte form and if you inspect somevar and see a 32-byte hex value, this is just “stringliteral” in hex.

The type bytes is similar, only that it can change its length.

Finally, string is basically identical to bytes only that it is assumed to hold the utf-8 encoding of a real string. Since string stores the data in utf-8 encoding it is quite expensive to compute the number of characters in the string (the encoding of some characters takes more than a single byte). Because of that, string s; s.length is not yet supported and not even index access s[2]. But if you want to access the low-level byte encoding of the string, you can use bytes(s).length and bytes(s)[2] which will result in the number of bytes in the utf-8 encoding of the string (not the number of characters) and the second byte (not character) of the utf-8 encoded string, respectively.

Can a contract pass an array (static size) or string or bytes (dynamic size) to another contract?

Sure. Take care that if you cross the memory / storage boundary, independent copies will be created:

```
contract C {

  uint[20] x;

  function f() {

    g(x);

    h(x);

  }

  function g(uint[20] y) {

    y[2] = 3;

  }

  function h(uint[20] storage y) {

    y[3] = 4;

  }
```

The call to g(x) will not have an effect on x because it needs to create an independent copy of the storage value in memory (the default storage location is memory). On the other hand, h(x)successfully modifies x because only a reference and not a copy is passed.

Sometimes, when I try to change the length of an array with ex: “arrayname.length = 7;” I get a compiler error “Value must be an lvalue”. Why?

You can resize a dynamic array in storage (i.e. an array declared at the contract level) witharrayname.length = <some new length>;. If you get the “lvalue” error, you are probably doing one of two things wrong.

1. You might be trying to resize an array in “memory”, or


1. You might be trying to resize a non-dynamic array.

**int8**[] **memory** memArr;       *// Case 1*memArr.length**++**;            *// illegal***int8**[5] storageArr;         *// Case 2*somearray.length**++**;         *// legal***int8**[5] **storage** storageArr2; *// Explicit case 2*somearray2.length**++**;         *// legal*

**Important note:** In Solidity, array dimensions are declared backwards from the way you might be used to declaring them in C or Java, but they are access as in C or Java.

For example, int8[][5] somearray; are 5 dynamic int8 arrays.

The reason for this is that T[5] is always an array of 5 T`s, no matter whether `T itself is an array or not (this is not the case in C or Java).

Is it possible to return an array of strings ( string[] ) from a Solidity function?

Not yet, as this requires two levels of dynamic arrays (string is a dynamic array itself).

If you issue a call for an array, it is possible to retrieve the whole array? Or must you write a helper function for that?

The automatic accessor function for a public state variable of array type only returns individual elements. If you want to return the complete array, you have to manually write a function to do that.

What could have happened if an account has storage value/s but no code? Example:[http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5](http://test.ether.camp/account/5f740b3a43fbb99724ce93a879805f4dc89178b5)

The last thing a constructor does is returning the code of the contract. The gas costs for this depend on the length of the code and it might be that the supplied gas is not enough. This situation is the only one where an “out of gas” exception does not revert changes to the state, i.e. in this case the initialisation of the state variables.

[https://github.com/ethereum/wiki/wiki/Subtleties](https://github.com/ethereum/wiki/wiki/Subtleties)

After a successful CREATE operation’s sub-execution, if the operation returns x, 5 * len(x) gas is subtracted from the remaining gas before the contract is created. If the remaining gas is less than 5 * len(x), then no gas is subtracted, the code of the created contract becomes the empty string, but this is not treated as an exceptional condition - no reverts happen.

## How do I use .send()?

If you want to send 20 Ether from a contract to the address x, you use x.send(20 ether);. Here, x can be a plain address or a contract. If the contract already explicitly defines a function send (and thus overwrites the special function), you can use address(x).send(20 ether);.

What does the following strange check do in the Custom Token contract?

**if** (balanceOf[_to] **+** _value **<** balanceOf[_to]) **throw**;

Integers in Solidity (and most other machine-related programming languages) are restricted to a certain range. For uint256, this is 0 up to 2**256 - 1. If the result of some operation on those numbers does not fit inside this range, it is truncated. These truncations can have [serious consequences](https://en.bitcoin.it/wiki/Value_overflow_incident), so code like the one above is necessary to avoid certain attacks.

## More Questions?

If you have more questions or your question is not answered here, please talk to us on [gitter](https://gitter.im/ethereum/solidity) or file an [issue](https://github.com/ethereum/solidity/issues).
