# Solidity Audit Cheat sheet

* #### [Integer Over flow and Under flow](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/index.md#Integer-Over-flow-and-Under-flow)

* #### [Frozen / Locked Ether](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/index.md#Frozen-Locked-Ether)

* #### [Re-entrancy](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/index.md#re-entrancy)

* #### [Logic control using address balance](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/index.md#re-entrancy)

## Integer Over flow and Under flow <a name="Integer-Over-flow-and-Under-flow"></a>

#### Over flow example

 ```solidity
 uint8 a = 255;
 a++; 
 //overflow error, 256 is not an unsigned int8. This will cause an integer overflow, b = 0.    
 ```

#### Under flow example

```solidity
uint8 a = 1;
uint8 b = a - 2; 
//underflow error, -1 is not an unsigned int8. This will cause an integer undeflow, b = 255
```



Check for integer overflows and underflows in transfer, mint and burn functions. Balances are mostly stored as `unit256`. 

Use [library SafeMath](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol) from [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts) for uint256.

### Vulnerable Code example

```solidity
function transfer(address _to, uint256 _value) {
    require(balanceOf[msg.sender] >= _value);
    balanceOf[msg.sender] -= _value; 				//potential underflow
    balanceOf[_to] += _value; 					//potential overflow 
}
```

```solidity
//another fix style
require(balanceOf[msg.sender] >= _value && balanceOf[_to] + _value >= bal
```

## Frozen / Locked Ether <a name="Frozen-Locked-Ether"></a>

* Avoid receiving ether on the smart contract. No payable `function, constructor, receive or fallback` function.

* But ether can still be received on the contract if the contract address in the recipient of mining rewards or is passed as a parameter to the `selfdestruct(address payablecontractAddress)`

* Implement a withdrawal function with access restriction: `onlyOwner` modifier pattern or [role based access](https://docs.openzeppelin.com/contracts/4.x/api/access) management.

#### Withdrawal function example

``` solidity
 function withdraw(uint amount) public onlyOwner{	//or some other access control mechanism you like
        payable(msg.sender).transfer(amount);
 } //the function could end up withdrawing everything??
```

##  Re-entrancy <a name="re-entrancy"></a>

![](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/re-entrancy.png)

#### Conditions for re-entracy

* The calling function provides enough gas for the called function
* Re-entrancy back into the calling functions finds that it has not updated state variables "balance"

### Logic control using address balance  <a name="Logic-control-using-address-balance"></a>

```solidity
uint bal = address(this).balance
if (bal == 0)
	doSomeThing();
```

Assuming the contract cannot receive ether:

* no payable function
* no receive function
* no payable fallback
* constructor is not payable

`bal` could still be positive:

* Another contract could call `selfdestruct(your-contract-address)`, ether in that contract will be sent to your contract
* `your-contract-address` could be used as recipient of miners fee

