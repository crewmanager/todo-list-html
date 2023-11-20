## todo list 


`main.mo`

```js
import Text "mo:base/Text";
import List "mo:base/List";
import Debug "mo:base/Debug";
import Nat "mo:base/Nat";

actor {

  public type Note = {
    textarea : Text;
    id : Nat;
  };

  var notes : List.List<Note> = List.nil<Note>();

  public func addTodo(title : Text) : async Nat {
    let todoId = List.size(notes) +1;

    let newNote : Note = {
      id = todoId;
      textarea = title;
    };

    notes := List.push(newNote, notes);
    Debug.print(debug_show (notes));

    return newNote.id;
  };

  public query func getAllTodo() : async [Note] {
    return List.toArray(notes);

  };

  public func deleteTodo(id : Nat) {
    Debug.print(debug_show (id));

    notes := List.filter(
      notes,
      func(note : Note) : Bool {
        return note.id != id;
      },
    );

    Debug.print("id is deleted");
  };

};

```

`main.css`

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Times New Roman", Times, serif;
}

body {
  background-image: linear-gradient(to right, #605e7e, #6b73c1, #37b3cc);
  margin: 50px 2%;
}

.container-title {
  font-size: 25px;
  text-align: center;
  margin-bottom: 30px;
  color: white;
  text-shadow: 3px 1px black;
}

.inputcol {
  display: grid;
  column-gap: 5px;
  grid-template-columns: 60% 10%;
  justify-content: center;
  margin-top: 10px;
  margin-bottom: 20px;
}

.textarea {
  min-height: 50px;
  max-width: 100%;
  border-radius: 10px;
  border-color: #333;
  font-size: 20px;
  padding: 10px;
  overflow: auto;
  overflow-x: hidden;
}

.textarea::-webkit-scrollbar-track {
  border-radius: 10px;
  background-color: #333;
}

.textarea::-webkit-scrollbar {
  width: 10px;
  cursor: pointer;
}

.textarea::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background-color: #8f8acc;
}

.textarea:focus {
  outline: none;
}

.buttoninput {
  border-radius: 10px;
  border-color: #333;
  background-color: white;
  font-size: 20px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.buttoninput:hover {
  background-color: #8f8acc;
  color: white;
}

#todolist .item {
  background-color: #fff;
  border-radius: 8px;
  margin-bottom: 10px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: space-between;
  word-wrap: break-word;
  word-break: break-all;
  font-size: 20px;
}

.trash-button {
  background-color: transparent;
  color: black;
  border: none;
  border-radius: 5px;
  padding: 5px 10px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.trash-button:hover {
  background-color: #6b73c1;
  color: white;
  transform: scale(1.05);
}

.trash-button:active {
  transform: scale(0.95);
}

.fa-check,
.fa-trash {
  pointer-events: none;
}

```
`index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="main.css" />
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css"
    />
    <title>ToDo List</title>
  </head>
  <body>
    <div class="container-title">
      <h1>TO DO LIST</h1>
    </div>
    <div class="container">
      <form class="inputcol">
        <textarea
          class="textarea"
          type="text"
          name="textarea"
          placeholder="Add todo"
          required
        ></textarea>
        <button class="buttoninput" type="submit">
          <i class="fa-solid fa-check"></i>
        </button>
      </form>
    </div>
    <ul id="todolist"></ul>
    <script src="index.js"></script>
  </body>
</html>

```

`index.js`

```js
import { todo_backend } from "../../declarations/todo_backend";

const textarea = document.querySelector(".textarea");
const appointForm = document.querySelector(".inputcol");

appointForm.addEventListener("submit", async (e) => {
  e.preventDefault();
  const obj = {
    textarea: textarea.value,
  };

  try {
    const getId = await todo_backend.addTodo(obj.textarea);
    showUserOnScreen({ id: getId, textarea: obj.textarea });
    textarea.value = ""; 
  } catch (error) {
    console.error("Error adding todo:", error);
  }
});

window.addEventListener("DOMContentLoaded", async () => {
  try {
    const getAll = await todo_backend.getAllTodo();
    getAll.forEach((todo) => showUserOnScreen(todo));
  } catch (error) {
    console.error("Error fetching todo list:", error);
  }
});

function showUserOnScreen(user) {
  const parentNode = document.getElementById("todolist");

  const newListItem = document.createElement("li");
  newListItem.className = "item";
  newListItem.id = Number(user.id);
  newListItem.textContent = user.textarea;

  const deleteButton = document.createElement("button");
  deleteButton.className = "trash-button";
  deleteButton.innerHTML = '<i class="fa-solid fa-trash"></i>';
  deleteButton.onclick = () => deleteTodo(user.id);

  newListItem.appendChild(deleteButton);
  parentNode.appendChild(newListItem);
}

async function deleteTodo(userId) {
  try {
    console.log("userId in del =>", userId);
    await todo_backend.deleteTodo(userId);
    removeUserFromScreen(userId);
  } catch (error) {
    console.error("Error deleting todo:", error);
  }
}

function removeUserFromScreen(userId) {
  const childNodeToBeDeleted = document.getElementById(userId);
  if (childNodeToBeDeleted) {
    childNodeToBeDeleted.parentNode.removeChild(childNodeToBeDeleted);
  }
}

```
