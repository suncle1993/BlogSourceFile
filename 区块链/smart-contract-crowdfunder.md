---
abbrlink: 3775562041
alias: 2021/08/21/smart-contract-crowdfunder/index.html
categories:
- 区块链
date: '2021-08-21T16:12:37'
tags:
- Solidity
title: 智能合约案例-众筹
---







一个众筹的智能合约示例，来源于 [learnxinyminutes](https://learnxinyminutes.com/docs/solidity/)

主要是用来展示智能合约状态机（State machines）和函数修饰器（modifier）的使用

```solidity
// CrowdFunder.sol
pragma solidity ^0.6.6;

/// @title CrowdFunder
/// @author nemild
contract CrowdFunder {
    // Variables set on create by creator
    address public creator;
    address payable public fundRecipient; // creator may be different than recipient, and must be payable
    uint public minimumToRaise; // required to tip, else everyone gets refund
    string campaignUrl;
    byte version = "1";

    // Data structures
    enum State {
        Fundraising,
        ExpiredRefund,
        Successful
    }
    struct Contribution {
        uint amount;
        address payable contributor;
    }

    // State variables
    State public state = State.Fundraising; // initialize on create
    uint public totalRaised;
    uint public raiseBy;
    uint public completeAt;
    Contribution[] contributions;

    event LogFundingReceived(address addr, uint amount, uint currentTotal);
    event LogWinnerPaid(address winnerAddress);

    modifier inState(State _state) {
        require(state == _state);
        _;
    }

    modifier isCreator() {
        require(msg.sender == creator);
        _;
    }

    // Wait 24 weeks after final contract state before allowing contract destruction
    modifier atEndOfLifecycle() {
    require(((state == State.ExpiredRefund || state == State.Successful) &&
        completeAt + 24 weeks < now));
        _;
    }

    function crowdFund(
        uint timeInHoursForFundraising,
        string memory _campaignUrl,
        address payable _fundRecipient,
        uint _minimumToRaise)
        public
    {
        creator = msg.sender;
        fundRecipient = _fundRecipient;
        campaignUrl = _campaignUrl;
        minimumToRaise = _minimumToRaise;
        raiseBy = now + (timeInHoursForFundraising * 1 hours);
    }

    function contribute()
    public
    payable
    inState(State.Fundraising)
    returns(uint256 id)
    {
        contributions.push(
            Contribution({
                amount: msg.value,
                contributor: msg.sender
            }) // use array, so can iterate
        );
        totalRaised += msg.value;

        emit LogFundingReceived(msg.sender, msg.value, totalRaised);

        checkIfFundingCompleteOrExpired();
        return contributions.length - 1; // return id
    }

    function checkIfFundingCompleteOrExpired()
    public
    {
        if (totalRaised > minimumToRaise) {
            state = State.Successful;
            payOut();

            // could incentivize sender who initiated state change here
        } else if ( now > raiseBy )  {
            state = State.ExpiredRefund; // backers can now collect refunds by calling getRefund(id)
        }
        completeAt = now;
    }

    function payOut()
    public
    inState(State.Successful)
    {
        fundRecipient.transfer(address(this).balance);
        LogWinnerPaid(fundRecipient);
    }

    function getRefund(uint256 id)
    inState(State.ExpiredRefund)
    public
    returns(bool)
    {
        require(contributions.length > id && id >= 0 && contributions[id].amount != 0 );

        uint256 amountToRefund = contributions[id].amount;
        contributions[id].amount = 0;

        contributions[id].contributor.transfer(amountToRefund);

        return true;
    }

    function removeContract()
    public
    isCreator()
    atEndOfLifecycle()
    {
        selfdestruct(msg.sender);
        // creator gets all money that hasn't be claimed
    }
}
```

<!--more-->

## 修饰器

函数修饰器(Modifiers)可以用来改变一个函数的行为。比如用于在函数执行前检查某种前置条件。这个和python的修饰器(Decorators)的作用很类似，在python中，我们也经常使用装饰器对函数执行前后增加一些逻辑。下面是solidity修饰器的简单使用，在众筹支付前需要检查合约状态是否已完成

```solidity
function payOut() public inState(State.Successful) {_;}
```

python中一个函数可以有多个装饰器，solidity中的函数也是可以有多个修饰器的。

如果同一个函数有多个修饰器，他们之间以空格隔开，修饰器会依次检查执行。

## 状态机

状态State在合约中本质是一个合约的全局变量，在实际合约中，状态会有很多种，各个合约方法也会对State做修改，并且根据State执行不同的逻辑。对于多个State可以通过枚举管理

```solidity
enum State {
    Fundraising,	// 筹款中
    ExpiredRefund,	// 过期退款
    Successful	// 众筹成功
}
```



