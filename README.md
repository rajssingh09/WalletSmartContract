# WalletSmartContract

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

//Transcition - from , to, timing ,amount


contract SimpleWallet {
   
   struct Transaction{
      address from;
      address to;
      uint timestamp;
      uint Amount;
   }

   address public owner;
   string public str;
   bool public  stop;
    
  Transaction[] public TransactionHistory;

  
    
    event Transfer(address receiver, uint amount);
    event Receive(address sender, uint amount);
    event UserInfo(address sender, address receiver, uint amount);
    //event Balance(address _useraddress);


 constructor(){
    owner=msg.sender;
 }
  
  function toogleStop () external onlyOwner{
    stop=!stop;  //false, true, false , true
  }

  function emergencywithdraw() external payable {
   require(stop==true, "Emergency not Declared");
  // require(address(this).balance>msg.value,"insufficient amount");
   payable(owner).transfer(address(this).balance);
  } 

  modifier Emergencydeclared(){
    require(stop==false, "emergency stop");
    _;
  }
  function newowner(address _owner) public onlyOwner Emergencydeclared{
    owner=_owner;
  }


   /*modifier time(){
    require(block.timestamp>=1724687386, "Invalid Time");
    _;
   }*/
    modifier onlyOwner() {
        require(msg.sender==owner, "You don't have access");
        _;
       }

       mapping (address=>uint) Suspicioususer;

   
    modifier getSuspicioususer(address _sender){
      require(Suspicioususer[_sender]<5 ,"Activity found Suspiciou , Try Later");
      _;
    }
    function transferToContract(uint _startTime) external payable  getSuspicioususer(msg.sender) Emergencydeclared /*ban(block.timestamp)*/ {
      require(block.timestamp>_startTime ,"Invalid Time");
      TransactionHistory.push(Transaction({
       from:msg.sender,// msg.sender;
       to: address(this),
       timestamp:block.timestamp,
       Amount: msg.value
      }));}
      

     /* for(uint i=1; true; i++){
         uint id=i;
      address from= owner;
      address to= address(this).msg.sender;
      uint timestamp= block.timestamp;
      uint Amount= msg.value;
       TransactionHistory.push(Transaction);
      }}
      /*while (true){
       uint id=1;
      address from= owner;
      address to= address(this).msg.sender;
      uint timestamp;
      uint Amount;
      }*/
    function transferToUserViaContract(address payable  _to, uint _weiAmount) external onlyOwner {
        require(address(this).balance>=_weiAmount, "Balance Insufficient");
        require(_to!=address(0),"Address Format Is Incorrect");
       // require(block.timestamp==_to(block.timestamp), "Invalid Time");
        _to.transfer(_weiAmount);
        emit Transfer(_to, _weiAmount); 
         TransactionHistory.push(Transaction({
       from:msg.sender,// msg.sender;
       to: _to,
       timestamp:block.timestamp,
       Amount: _weiAmount
      }));
        
      }

      /*function getbanSususer(uint _bantime) external {

      }*/



//0xAAC129A3e6e9f44147951dDD5655d66c312A4713

    function withdrawFromContract(uint _weiAmount) external onlyOwner {
      require(_weiAmount<=address(this).balance, "balance insuffcient");
     payable (owner).transfer(_weiAmount);
     TransactionHistory.push(Transaction({
       from:address(this),// msg.sender;
       to: owner,
       timestamp:block.timestamp,
       Amount: _weiAmount
      }));
     
    }




    function getContractBalanceInWei() external view returns(uint){
        return address(this).balance; //balance of smart contract address(this) 
       
    }
   
     //user related function
    function transferToUserViaMsgValue(address  _to ) external payable onlyOwner {
      require(address(this).balance>=msg.value,"Insufficient value");
      require(_to!=address(0),"Address Format Is Incorrect");
      payable (_to).transfer(msg.value);
       TransactionHistory.push(Transaction({
       from:address(this),// msg.sender;
       to: _to,
       timestamp:block.timestamp,
       Amount: msg.value
      }));
        
       
    }

 //event UserInfo(address sender, address receiver, uint amount);

    function receiveFromUser() external payable {
      require(msg.value>=0, "Wei Value Must be greater than zero");
      payable(owner).transfer(msg.value);// eth send to owner account 
    //  payable (address(this)).transfer(msg.value); //eth send to this smart contract address 
    //  payable (address(this)).transfer(msg.value(owner));
    emit UserInfo(msg.sender, owner, msg.value);
    TransactionHistory.push(Transaction({
       from:msg.sender,// msg.sender;
       to: owner,
       timestamp:block.timestamp,
       Amount: msg.value
      }));
        
    }


    function getOwnerBalanceInWei() external onlyOwner view returns(uint){
      // return address(this).balance; // if i want to know smart contract balance 
       return owner.balance; // owner account uska amount
       // emit Balance(msg.sender);
    }

    receive() external payable { 
   emit Receive(msg.sender,msg.value);
   TransactionHistory.push(Transaction({
       from:msg.sender,// msg.sender;
       to: msg.sender,
       timestamp:block.timestamp,
       Amount: msg.value
      }));
        
    }
 function Suspiciousactivity(address _sender) public   {
    Suspicioususer[_sender]+=1;
 }

 /*modifier ban(uint bantime){
  require(block.timestamp>=bantime+36000, "you're banned for 10hrs");
  _;
 }*/
 
 
  /*function GetbanSususer(uint _bantime) public pure returns(uint){
    uint bantime=_bantime;
   return bantime;
   }*/
   fallback() external payable  { 
     //payable(msg.sender).transfer(msg.value); //returning ether to the user 
    Suspiciousactivity(msg.sender);
    //GetbanSususer(block.timestamp);
   }
   
function getTranscitionHistory() external   view returns(Transaction[] memory){
 return  TransactionHistory;
}
}




