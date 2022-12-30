# Akbank-Web3-Practicum-Final-Project

``` solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

// This contract organize communication between customer and freelancer with formal way.
// It prevents the freelancer from grappling with a bunch of additional requests without paying extra.     
contract Freelancer{

  // These are the events that I use in the function. 
  event Launch(uint id, address indexed customer,string customerName, uint32 startAt, uint32 endAt, uint amount);
  event ContinueProject(uint _id);
  event Request(uint amount, string textOfRequest, uint id);
  event Cancel(uint id, string cancelText);

  // This is the struct that includes customer information who initialize the project.
  struct Customer{
      address payable customerAddress;
      string customerName;
      uint32 startAt;
      uint32 endAt;
      uint256 amount;
      string request;
      uint halfFunctionCallTime;
    }
    
    //Freelancer's address.
    address payable owner;

    // Minimum amounts to launch functions.
    uint public minRequestAmount = 1 ;
    uint public minLaunchAmount = 10 ;
    uint public customerCount;

    // Mapping the contract address and it's balances.
    mapping(address => uint) balances;

    // Mapping the customer informations and customer id's.
    mapping(uint => Customer) public customers;

    // Modify the function controlling sender's balance is greater than minimum amount for the request.
    modifier minRequest {
      require(balances[msg.sender] > minRequestAmount,"Your balance is greater than minimum request amount: 100000");
      _;
    }

    // Modify the function controlling sender's address is equal contract creator address.
    modifier onlyOwner {
      require(msg.sender == owner,"You are not owner");
      _;
    }

    // The contract creator is the owner.
    constructor() {
      owner =payable(msg.sender);
    }

    // Load your balance using your money amount in your balance.
    function loadBalance() external {
      balances[msg.sender] = 100;
    }

    // You can see your balance after loadinG your balance.
    function showBalance(address _yourAddress) external view returns(uint) {

      return(balances[_yourAddress]);
    }

    // Launching the project with controlling the requiring things.
    // Loading customer struct with entering datas.
    function launchProject(uint _amount, uint32 _startAt, uint32 _endAt, string calldata _name) external  {
      require(_amount > minLaunchAmount, "You dont have enough money to launch a project");
      require(_startAt > block.timestamp,"Start at < NOW");
      require(_endAt >= _startAt, "end at < start at");
      require(_endAt <= block.timestamp + 50 days, "End at > max duration");
      require(balances[msg.sender] > _amount,"Please load balance: Your balance is not enough");

      balances[msg.sender] -= _amount/2;
      balances[owner] += _amount/2;

      customerCount+=1;

      customers[customerCount] = Customer({
        customerAddress: payable(msg.sender),
        customerName: _name,
        startAt: _startAt,
        endAt: _endAt,
        amount: _amount/2,
        halfFunctionCallTime: 0,
        request: ""
      });

      emit Launch(customerCount, msg.sender,_name, _startAt, _endAt, _amount);

    }
    
    // If freelancer completed the half of project and customer wants to continue to work with current freelancer
    // it can continue with entering his id and paying the remaining half of the money needed to complete the project
    function halfProject(uint _id) external{
      Customer storage customer = customers[_id];  

      require(customer.halfFunctionCallTime < 1,"You can call this function just one time");
      require(msg.sender == customers[_id].customerAddress, "You did not start the project");
      require(balances[msg.sender]> customer.amount, "Your balance is not enough to continuing the project");
      require(block.timestamp > customer.startAt, "Project is not started");
      require(block.timestamp < customer.endAt, "Project time is ended");

      balances[customer.customerAddress] -= customer.amount;
      balances[owner] += customer.amount;
      customer.amount *= 2;

      customer.halfFunctionCallTime = 1;

      emit ContinueProject(_id);
    }

    // If customer wants to add something extra thing about the project, it have to pay amount of money
    function addRequest(uint _amount, string calldata _requestText, uint _id) external minRequest{
      Customer storage customer = customers[_id];

      require(block.timestamp > customer.startAt, "Project is not started");
      require(block.timestamp < customer.endAt, "Project time is ended");
      require(msg.sender == customers[_id].customerAddress, "You are not current customer");
      require(_amount > minRequestAmount,"You dont have enough money to add request. minRequestAmount = 100000");
      require(balances[customer.customerAddress] > _amount, "Your balance must be greater than entered amount");

      customer.request = _requestText;
      balances[customer.customerAddress] -= _amount;
      balances[owner] += _amount;
      
      emit Request(_amount, _requestText, _id);
    }

    // If freelancer wants to cancel the project, he can cancel with paying the whole money which has been payed 
    // Entering the text about that why it cancel the project (request amount does not include)
    function cancelProject(uint _id, string memory _message) external onlyOwner returns(string memory){

      Customer storage customer = customers[_id];

      require(block.timestamp > customer.startAt, "Project is not started");
      require(block.timestamp < customer.endAt, "Project time is ended");

      customer.customerAddress.transfer(customer.amount);

      delete customers[_id];
      
      emit Cancel(_id,_message);

      return _message;
    }

}
    



```