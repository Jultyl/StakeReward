//SPDX-License-Identifier: MIT

pragma solidity ^0.8.18;

import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract StakeReward is ReentrancyGuard {

    uint256 public rewardPercentage;
    uint256 public timeDeposit;
    address owner ;
    uint256 lastCalculateTime ;
    bool iswithdrawal = true;

    error NotSufficientBalance();
    error TransactionFail();
    error NotOwner();

    struct Staker {
        address stakerAddress;
        uint256 lastUpdateTime;        
        uint256 totalAmount;
    }

    Staker[] stakerDatabase;
    mapping (address => uint256) stakerAddressToStakerIndex ;
    uint256 stakerIndex = 0;

    modifier updateStake (uint256 _amount, bool _iswithdrawal) {
        uint256 _newtotalAmount;
        if (stakerAddressToStakerIndex[msg.sender] == 0) {        
        stakerDatabase.push(Staker(msg.sender, block.timestamp, _amount));        
        stakerAddressToStakerIndex[msg.sender] = stakerIndex ;
            stakerIndex ++;
        }
        else{
        Staker memory _staker = stakerDatabase[stakerAddressToStakerIndex[msg.sender]];
        uint256 _reward = _staker.totalAmount * (block.timestamp - lastCalculateTime) * rewardPercentage / 1000 / 31536000 ;
        if (_iswithdrawal == true){
        _newtotalAmount = _reward - _amount + _staker.totalAmount;       
        }
        else
        _newtotalAmount = _reward + _amount + _staker.totalAmount;       
        _staker.totalAmount = _newtotalAmount;
        _staker.lastUpdateTime = block.timestamp;
        }
        _;
    }

    modifier isOwner () {
        if (msg.sender != owner) revert NotOwner();
        _;
    }

    constructor (uint256 _rewardPercentage){
        rewardPercentage = _rewardPercentage;
        owner = msg.sender; 
    }

    function deposit () external payable updateStake(msg.value, false) nonReentrant {
    }

    function withdraw (uint256 _withdrawAmount) external updateStake(_withdrawAmount, true) nonReentrant {
        Staker memory _staker = stakerDatabase[stakerAddressToStakerIndex[msg.sender]]; 
        if (_withdrawAmount > _staker.totalAmount) revert NotSufficientBalance();
        (bool success,) = payable(msg.sender).call{value: _withdrawAmount}(""); 
        if (!success) revert TransactionFail();
        _staker.totalAmount -= _withdrawAmount ;
    }

    function checkStackAmount() external updateStake(0, false) isOwner() returns (uint256) {
        return stakerDatabase[stakerAddressToStakerIndex[msg.sender]].totalAmount;
    }

    function liquidityManagement (address _transferTo, uint256 _amount) external isOwner() {
        (bool success,) = payable(_transferTo).call{value: _amount}("");
        if (!success) revert TransactionFail();        
    }

}
