    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.17;

    contract Rental_disk{

        struct customer_info{
            address tenant;
            uint desk;
            uint due_time;
        }

    mapping (address => customer_info) public customer;

    address[] customer_record;

    address manager = msg.sender;

    uint amount_1_disk = 1 ether;

    uint public Total_desk = 15;

    event Desk_rented(address _Address,uint Desk_quantity);
    event Desk_return(address _Address,uint Desk_quantity);
    event Pay_rent(address _Address,uint _Amount);

    modifier onlyowner(){
        require(manager == msg.sender,"only maanger can be call this");
        _;
    }
    
    function Rent_A_Desk(uint _quantity) external  payable {

    require(Total_desk >= _quantity,"sorry but desk is not avalible");
    require(msg.value >= amount_1_disk * _quantity ,"1 desk price is 1 ether");

    customer_info storage temp =  customer[msg.sender];

    if (temp.tenant == address(0)){
        temp.tenant = msg.sender;
        }
        Total_desk -= _quantity;
        temp.desk += _quantity;
        temp.due_time = block.timestamp + 5 minutes;

        emit Desk_rented(msg.sender, _quantity);
        }

    function  Return_Desk(uint _Return_desk) external {

    customer_info storage temp =  customer[msg.sender];

    require(temp.due_time > block.timestamp,"your time is already complete");
    require(temp.tenant != address(0) ,"you are not a tenant");
    require(temp.desk >= _Return_desk,"you have no desk to return");

    Total_desk += _Return_desk;
    temp.desk -= _Return_desk;

    emit Desk_return(msg.sender, _Return_desk);
    }

    function pay_rent() external payable {

    customer_info storage temp =  customer[msg.sender];

    require(temp.tenant != address(0) ,"you are not a tenant");
    require(msg.value >= amount_1_disk * temp.desk,"amount is less when you pay the rent");
    
    temp.due_time += 5 minutes;

    emit Pay_rent(msg.sender, msg.value);
    }

    function Withdraw(uint _amount) external payable onlyowner {
        require(_amount <= address(this).balance ,"balance is less then your amount");
        payable (manager).transfer(_amount);
    }

    function check_conract_balance() external view returns (uint Balance) {

    return address(this).balance;
    }
}
