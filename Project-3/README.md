# Akbank-Practicum-Project-3
Hands on Task (Intermediate Level): Build and Deploy a Crowdfund Application

```solidity

// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract CrowdFund {

    event Launch(
        uint id, 
        address indexed creator, 
        uint goal, 
        uint32 startAt, 
        uint32 endAt 
        );
    event Cancel(uint id);
    event Pledge(uint indexed id, address indexed caller, uint amount);
    event Unpledge(uint indexed id, address indexed caller, uint amount);
    event Claim(uint id);
    event Refund(uint indexed id, address indexed caller , uint amount);
    
    // Campaign struct
struct Campaign{

    address creator;
    uint goal;
    uint pledged;
    uint32 startAt;
    uint32 endAt;
    bool claimed;
}

// Description of token
IERC20 public immutable token;
uint public count;

mapping(uint => Campaign) public campaigns;

mapping(uint => mapping(address => uint)) public pledgedAmount;

constructor(address _token){
    token = IERC20(_token);
}
function launch(
    uint _goal,
    uint32 _startAt,
    uint32 _endAt
) external {
    require(_startAt >= block.timestamp,"start at < NOW");
    require(_endAt >= _startAt, "end at < start et");
    require(_endAt <= block.timestamp + 90 days, "end at > max duration");

    count+=1;
    campaigns[count] = Campaign({
        creator: msg.sender,
        goal: _goal,
        pledged: 0,
        startAt: _startAt,
        endAt: _endAt,
        claimed: false


    });

    emit Launch(count, msg.sender, _goal, _startAt, _endAt);
}

// We can cancel the campaign if the campaign was started
function cancel(uint _id) external {
    Campaign memory campaign = campaigns[_id];
    require(msg.sender == campaign.creator,"Not creator");
    require(block.timestamp < campaign.startAt,"Started");
    delete campaigns[_id];
    emit Cancel(_id);
}
// We can add money which we want
function pledge(uint _id, uint _amount) external {
    Campaign storage campaign = campaigns[_id];
    require(block.timestamp >= campaign.startAt,"not started");
    require(block.timestamp <= campaign.endAt,"ended");
    campaign.pledged += _amount;
    pledgedAmount[_id][msg.sender] += _amount;
    token.transferFrom(msg.sender, address(this), _amount);

    emit Pledge(_id, msg.sender, _amount);
}

// We can take the money from the campaign if we change our idea and the campaign not finished 
function unpledge(uint _id, uint _amount) external {
    Campaign storage campaign = campaigns[_id];
    require(block.timestamp <= campaign.endAt,"ended");

    campaign.pledged -= _amount;
    pledgedAmount[_id][msg.sender] -= _amount;
    token.transfer(msg.sender, _amount);

    emit Unpledge(_id, msg.sender, _amount);
}

// If the pledged amount is greater than goal we can claim this money just one time
function claim(uint _id) external {
    Campaign storage campaign = campaigns[_id];
    require(msg.sender == campaign.creator,"not creator");
    require(block.timestamp > campaign.endAt,"not ended");
    require(campaign.pledged >= campaign.goal,"pledged < goal");
    require(!campaign.claimed, "claimed");

    campaign.claimed = true;
    token.transfer(msg.sender, campaign.pledged);

    emit Claim(_id);
}
// If the campaign goal is greater than pledged amount and the campaign time is ended we can retake the money from campaign if the campaign is unsuccessfull
function refund(uint _id) external {
    Campaign storage campaign = campaigns[_id];
    require(block.timestamp > campaign.endAt,"not ended");
    require(campaign.pledged < campaign.goal,"pledged < goal");

    uint bal = pledgedAmount[_id][msg.sender];
    pledgedAmount[_id][msg.sender] = 0;
    token.transfer(msg.sender, bal);
    
    emit Refund(_id, msg.sender, bal);
}
  
}
```
