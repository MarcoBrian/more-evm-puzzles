
# More EVM Puzzles Solutions

***

Github Link for the EVM Puzzles: https://github.com/MarcoBrian/more-evm-puzzles

*** 

## Puzzle 1

**Answer:** 
- Callvalue = 2
- Calldata = 0x001122334455 (or any 6 bytes of data)

**Explanation:**
1. Stack: {calldatasize} 
2. Stack: {calldatasize, callvalue} 
3. EXP -> callvalue ** calldatasize 
4. There's 2 conditions need to be met. 
	1. Calldatasize needs to be 6 so that we can reach location 47 for JUMPDEST 
	2. Callvalue ** calldatasize = callvalue ** 6 = 0x40 
5. So callvalue needs to be 2 

## Puzzle 2

**Answer:** 0x6005600c60003960056000f3600a6000f3


**Explanation:**

1. Stack : {calldatasize}
2. Stack: {Calldatasize,0,0}
3. Create contract 
4. stack: {address, 0 ,0 ,0 ,0 ,0}
5. call contract 
6. CALL => stack: {success}
7.  RETURNDATASIZE => stack: {success, size } 
8. stack: {success, size, 0a}
9. stack : {success, equal }
10. stack : {success, equal, 1F }

So we need to provide a contract code that is able to return 0A (10 bytes of data size) during runtime.

To solve this we need to create an init bytecode that will store the runtime bytecode into memory. 

```
--- Init byte code
PUSH1    0x05
PUSH1    0x0c
PUSH1    0x00
CODECOPY
PUSH1    0x05
PUSH1    0x00
RETURN
--- Runtime code
PUSH1    0x0a
PUSH1    0x00
RETURN
```



## Puzzle 3

**Answer:** 0x6005600c60003960056000f360aa600555

**Explanation:**


stack: {calldatasize}
stack: {calldatasize, 0, 0}
Delegatecall  -> stack: {success} 
push1 -> stack: {success, 05}
SLOAD -> `storage[5]`
Push1 -> stack: {success, `storage[5]` , aa} 

So the goal here is that the storage at location 5 needs to equal aa. 

```
PUSH1 0x05      
PUSH1 0x0c        
PUSH1 0x00
CODECOPY
PUSH1 0x05
PUSH1 0x00
RETURN
-- runtime code --
PUSH1 0xaa
PUSH1 0x05
SSTORE
```

## Puzzle 4

**Answer:** 
- Call value: 4 
- Call data: `0x600060006000600260025a33f100`


**Explanation:** 

1. ADDRESS -> stack : {address}
2. BALANCE -> stack : { balance}
3. CALLDATASIZE -> stack: { balance, calldatasize}
4. PUSH1 -> {balance, calldatasize, 0 }
5. PUSH1 -> stack : { balance, calldatasize, 0 , 0}
6. calldatacopy -> call data is now in memory. stack : { balance} 
7. CALLDATASIZE -> stack : { balance, calldata size}  
8. PUSH1 0 =>  stack : { balance, calldata size, 0}  
9. ADDRESS => stack: { balance, calldata size, 0, address}  
10. BALANCE => stack: { balance, calldata size, 0, balance}   
11. CREATE => stack: {balance, address}
12. BALANCE => stack: { balance, balance2} 
13. Balance2 / balance1 needs to be  == 2

```
PUSH1 0x00
PUSH1 0x00
PUSH1 0x00
PUSH1 0x00
PUSH1 0x02
GAS
CALLER
CALL
STOP
```


## Puzzle 5


**Answer:** 0x00112233445566778800112233445566778800112233445566778800112233445566778800112233445566778800112233445566778800112233445566

**Explanation:** 
- We need to input calldata with a byte size of 3d
- Fulfill 2 conditions
	- Calldata size > 20 
	- MSize - Calldata size = 3
- After 0x3D, the next available memory in 32 byte slots is 0x40. 

## Puzzle 6


**Answer:** 17 

**Explanation:** 
- Basically we need to make the the bytes overflow , we add 17 to the value and it will become 1 


## Puzzle 7


**Answer:** 4 

**Explanation:** 
- The idea is that we need to spend some amount of gas by going through the loop until the amount of gas used is A6


## Puzzle 8 


**Answer:** `0x6002600c60003960026000f333ff`

**Explanation:** 
The goals here is to 
- Deploy a contract with calldata as its code.
- Call that contract and expect it to return `1` (sucess).
- Ensure this contract’s ETH balance stays the same before and after the call.

To do this we need to create a  contract code in the call data that will self destruct when it is called. 


```
PUSH1 0x02
PUSH1 0x0c
PUSH1 0x00
CODECOPY
PUSH1 0x02
PUSH1 0x00
RETURN

-- RUNTIME CODE --- 
CALLER 
SELFDESTRUCT

```


## Puzzle 9 

**Answer:** 47
**Explanation:** 

The goal is to find the value X (callvalue) such that the first byte of the keccak256 value is 0xA8 

solve this formula : `shr(248, keccak256(pad32(x))) == 0xA8`


```python

from eth_hash.auto import keccak

target_byte = 0xA8
matches = []

for i in range(256):
    padded = i.to_bytes(32, byteorder='big')
    digest = keccak(padded)
    if digest[0] == target_byte:
        matches.append(i)

for match in matches:
    print(f"  {match} -> keccak256 starts with {hex(target_byte)}")

```


## Puzzle 10

**Answer** : `0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B0B`

**Explanation**: 
Whats happening is the program is 

1. We are loading 32 bytes from calldata[0:32]
2. AND it with `f0f0f0f0...`
3. Load 32 bytes from calldata[32:64]
4. OR it with the result from step 2
5. Where the final result is `ababab...`


Now we need to find out two equations: 

1. `F0 AND X = Y` 
2. `Y OR Z = AB` 

Some details we know 
`F0` in binary is `1111 0000
`AA` in binary is `1010 1010`

`F0` AND with any bytes will keep the top 4 bits of the input bytes and set bottom 4 bits to 0. 

Therefore F0 AND AA = A0 

Now we need to find Z where `A0 OR Z = AB` . Since this is an OR operation and we want to keep the first 4 bits , Z's first 4 bits will be zero. And the bottom 4 bits will be B 

We can figure out `Z = 0B` 

1. `F0 AND AA = A0`
2. `A0 OR 0B = AB` 

so we need our first 32 bytes to be AAAAAA...
and the second 32 bytes to be 0B0B0B..




