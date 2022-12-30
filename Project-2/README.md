# Akbank-Practicum-Project-2
Hands on Task (Intermediate Level): Build and Deploy a To-Do List

![image](https://user-images.githubusercontent.com/85889196/192114055-51e6f7d1-8d63-443d-9b2f-468db1a9db12.png)

```solidity

// SPDX-License-Identifier: MIT

pragma solidity >=0.7.0 <0.9.0;

contract TodoList{

    struct Todo{
        string text;
        bool completed;
    }

    // Creating instance of Todo[] struct array
    Todo[] public todos;

    // Function that create mission on todos
    function create(string calldata _text) external{
        todos.push(Todo({
            text: _text,
            completed: false
        }));
    
    }

    // Function that update todo mission with taking index
    function updateText(uint _index, string calldata _text) external{
        todos[_index].text = _text;
    }

    // Function that return by indexed todo mission
    function get(uint _index) external view returns(string memory, bool){
        Todo memory todo = todos[_index];
        return(todo.text, todo.completed);
    }

    // Function that change of completed situation
    function toogleCompleted(uint _index) external {
        todos[_index].completed = !todos[_index].completed;
    }

    
}

```

