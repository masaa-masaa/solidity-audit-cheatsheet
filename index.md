# Solidity Audit Cheat sheet

## Integer Over flow and Under flow

#### Over flow Example

 ```solidity
 	uint8 a = 255;
 	a++; //overflow error, 256 is not an unsigned int8. This will cause overflow and a will be 0.    
 ```

#### Under flow Example

```solidity
	uint8 a = 1;
	uint8 b = a - 2; //underflow error, -1 is not an unsigned int8. This will cause an undeflow, a = 255
```



