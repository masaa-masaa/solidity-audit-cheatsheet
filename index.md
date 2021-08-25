# Solidity Audit Cheat sheet

## Integer Over flow and Under flow

#### Over flow Example

 ```solidity
 uint8 a = 255;
 a++; //overflow error, 256 is not an unsigned int8. This will cause an integer overflow, b = 0.    
 ```

#### Under flow Example

```solidity
uint8 a = 1;
uint8 b = a - 2; //underflow error, -1 is not an unsigned int8. This will cause an integer undeflow, b = 255
```



Check for integer overflows and underflows in transfer, mint and burn functions. Balances are mostly stored as `unit256`. 

Use [library SafeMath](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol) from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) for uint256.

