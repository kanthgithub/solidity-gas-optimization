# solidity-gas-optimization

Coding in solidity is a little different than other languages as every action you do cost real gas. Here are some tips and tricks to help you keep your gas cost low.

Credits: https://mudit.blog/solidity-gas-optimization-tips/

## Pack your variables!

```
In ethereum, you pay gas for every storage slot you use. 
A slot is of 256 bits, and you can pack as many variables as you want in it. 
Packing is done by solidity compiler and optimizer automatically, 
you just need to declare the packable functions consecutively.
The below code is an example of poor code and will consume 3 storage slot
```

``js
uint8 numberOne;
uint256 bigNumber;
uint8 numberTwo;
``

```
A much more efficient way to do this in solidity will be
```

``js
uint8 numberOne;
uint8 numberTwo;
uint256 bigNumber;
``

```
This small change will save you a lot of gas as it will now only need 2 slots to store 
(It takes 20k gas to store 1 slot of data).
These small things are what make solidity different from other languages.
Stuff like the order of variables never mattered as much in other languages when it came to optimization.
Another thing to notice is that structs, mappings, and arrays always start from a new slot.

2) uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

3) Mappings are cheaper than Arrays! Mostly

Solidity is the first language that I have used in which mappings are less expensive than arrays! This is because of how EVM works, An array is not stored sequentially in memory but as a mapping. You can pack Arrays but not Mappings though. So, it’s cheaper to use arrays if you are using smaller elements like uint8 which can be packed together. You can’t get the length of a mapping or parse through all its elements, so depending on your use case, you might be forced to use an Array even though it might cost you more gas.

4) Not all elements can be packed.

Elements in Memory and Call Data cannot be packed. There is no gas saving in solidity by using smaller variables in function calls and memory.

5) Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

6) Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

7) Use external function modifier.

For all the public functions, the input parameters are copied to memory automatically, and it costs gas. If your function is only called externally, then you should explicitly mark it as external. External function’s parameters are not copied into memory but are read from calldata directly. This small optimization in your solidity code can save you a lot of gas when the function input parameters are huge.

8) Delete variables that you don’t need.

In Ethereum, you get a gas refund for freeing up storage space. If you don’t need a variable anymore, you should delete it using the delete keyword provided by solidity or by setting it to its default value. This will also help in keeping the blockchain size smaller. An ocean is filled with droplets.

9) Use Short Circuiting rules to your advantage.

When using logical disjunction (||), logical conjunction (&&), make sure to order your functions correctly for optimal gas usage. In logical disjunction (OR), if the first function resolves to true, the second one won’t be executed and hence save you gas. In logical disjunction (AND), if the first function evaluates to false, the next function won’t be evaluated. Therefore, you should order your functions accordingly in your solidity code to reduce the probability of needing to evaluate the second function.

10) Avoid changing storage data.

Changing storage data costs a lot more gas than changing memory or stack variables so you should update the storage variable after all the calculations rather than updating it on every calculation. The following solidity code will help you understand the difference between a poor code and better-optimized code

``js
contract Demo
{
    uint internal counter;
 
    // The below function updates the storage counter every time
    // This is a bad coding practice and should be avoided
    // as updating a storage variable is expensive
    function badFunction(){
        for (uint i = 0; i < 100; i++){
            counter++;
        }
    }
	
    // This function uses a stack variable, j for calculations
    // and updates the storage variable at the last.
    // it's cheaper as updating a stack variable is almost free
    function betterfunction(){
        uint j;
        for (uint i = 0; i < 100; i++){
            j++;
        }
        counter = j;
    }
    // I know, you don't need a loop for this :/
}
``

## Solidity tips and tricks to save gas and reduce bytecode size

- https://blog.polymath.network/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size-c44580b218e6
