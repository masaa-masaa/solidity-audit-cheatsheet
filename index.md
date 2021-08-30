# Solidity Audit Cheat sheet

* #### Integer Over flow and Under flow

* #### Frozen / Locked Ether

* #### Reentrancy

* #### Logic control using address balance

* #### Unchecked Return Values For Low Level Calls

* #### Unspecified Function Visibility

* #### Sensitive Data Storage

* #### Unprotected selfDestruct Call

## Integer Over flow and Under flow 

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

## Frozen / Locked Ether 

* Avoid receiving ether on the smart contract. No payable `function, constructor, receive or fallback` function.

* But ether can still be received on the contract if the contract address in the recipient of mining rewards or is passed as a parameter to the `selfdestruct(address payablecontractAddress)`

* Implement a withdrawal function with access restriction: `onlyOwner` modifier pattern or [role based access](https://docs.openzeppelin.com/contracts/4.x/api/access) management.

#### Withdrawal function example

``` solidity
 function withdraw(uint amount) public onlyOwner{	//or some other access control mechanism you like
        payable(msg.sender).transfer(amount);
 } //the function could end up withdrawing everything??
```

##  Reentrancy 

![](https://github.com/masaa-masaa/solidity-audit-cheatsheet/blob/main/re-entrancy.png)

#### Conditions for reentrancy

* The calling function provides enough gas for the called function
* Reentrancy back into the calling functions finds that it has not updated state variables "balance"

### Logic control using address balance  

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

# Unchecked Return Values For Low Level Calls  

Solidity low level calls:

```solidity
<address payable>.send(uint256 amount) returns (bool) //sending ether to an address
<address>.call(bytes memory) returns (bool, bytes memory) //calling code to a contract, calling a function. To be avoided
<address>.delegatecall(bytes memory) returns (bool, bytes memory) //calling code in a library or another contract
<address>.staticcall(bytes memory) returns (bool, bytes memory)

//example
function withdraw(uint256 _amount) public {
	require(balances[msg.sender] >= _amount);
	balances[msg.sender] -= _amount; //if the next line fails our current accounting is erroneous
	msg.sender.send(_amount); //bad code since return value not checked
}

//Transfer triggers revert on failure
function withdraw(uint256 _amount) public {
	require(balances[msg.sender] >= _amount);
	balances[msg.sender] -= _amount; //if the next line fails our accounting is rolled back
	msg.sender.transfer(_amount);
}
```

Always check the returned values from low level calls.

The low level calls do not trigger a revert on failure. 

# Unspecified Function Visibility

Functions without visibility specified are `public`.

```solidity
function withdrawAll() { //this is a public function, can be called by anyone
        msg.sender.transfer(this.balance); //send all contracts ether to caller of this function
}
```

The above function visibility should be set to external, public, internal or private plus access control applied to ensure that the caller has rights to withdraw.

# Sensitive Data Storage

The data store in a smart contract can be read by anyone with access to the blockchain data. Making the data private does not restrict external parties from reading the data it just ensures that inheriting contracts cannot access it.

```solidity
string password = "mypassword"; // the password can be read from the blockchain
uint x = 1000; //x can be read from the blockchain

function getX(string memory _password) external returns(uint){ // the functions protects nothing with the password
	if(keccak256(password) == keccak256(_password)){
		return x;
	}
}
```

Using the `web3.eth.getStorageAt(address, position [, defaultBlock] [, callback])` web3.js function to retrieve contract data.

# Unprotected selfDestruct Call

Calling the selfDestruct function on a contract without proper access control.

```solidity
function accidentallyKillMe()external { //no access control enforced
    selfdestruct(msg.sender); // send all ether to your self
  }
```



