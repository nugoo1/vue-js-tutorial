# Vue.js
<a href="https://www.youtube.com/watch?v=Wy9q22isx3U" target="_blank">Traversy Media Tutorial</a>

## Intro
Vue.js makes creates UI's and front-end apps much easier.
It has a smaller learning curve than React and Angular.
It is extremely fast and lightweight
Single Page Application like React
Vue comes with its own router

## Getting started
<a href="https://cli.vuejs.org" target="_blank">Vue CLI Documentation</a>
Let's start by installing the Vue-CLI (you'll need Nodejs)

`npm install -g @vue/cli`

Now create a new vue application with the CLI

`vue create test`

To start the server, run the following command

`npm run serve`

Alternatively, you can start the Vue GUI by running the following command

`vue ui`

This will start the GUI on port 8000. You can run all of the CLI commands on the GUI, it's way more user friendly! Go ahead and create a new project.

## Coding the Application

The main code that runs the application is stored in App.vue (like index.js in React).
In Vuejs each component has three parts. The template (html), the logic (javascript) and the styling (css). An example is shown below.

```
<template>
  <div id="app">
    {{ msg }}
  </div>
</template>

<script>
export default {
  name: "app",
  components: {

  },
  data() {
    return {
      msg: 'Hello'
    }
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, Helvetica, sans-serif;
  line-height: 1.4;
}
</style>
```

Notice that we've stored a msg in the data function. This is most basic way we handle state in Vue.js.

### State
Now let's actually start storing some data in the App component.
Notice the v-bind directive which passes in the data to the child component.

```
<template>
  <div id="app">
    <Todos v-bind:todos="todos"/>
  </div>
</template>


<script>
import Todos from './components/Todos.vue'

export default {
  name: 'app',
  components: {
    Todos
  },
  data() {
    return {
      todos: [
        {
          id: 1,
          title: "Todo One",
          completed: false
        },
        {
          id: 2,
          title: "Todo Two",
          completed: false
        },
        {
          id: 3,
          title: "Todo Three",
          completed: false
        },
      ]
    }
  }
}
</script>
```

Now we don't actually have access to this in the Todos Component. Let's fix that. The v-bind directive is used here again to set a unique key (like in React), and the v-for directive lets us loop through each item in our array. Don't forget to state that props are being used in the export method.

```
<template>
  <div>
    <div v-bind:key="todo.id" v-for="todo in todos">
      <h3>
          {{todo.title}}
      </h3>
    </div>
  </div>
</template>

<script>
export default {
  name: "Todos",
  props: ["todos"]
};
</script>


<style scoped>
h1 {
  color: black;
}
</style>
```

### Conditionals

The following snippet of code conditionally adds a 'is-complete' class to the div if the todo.completed property is equal to true.

```
<template>
  <div class="todo-item" v-bind:class="{ 'is-complete':todo.completed }">
    <p>{{ todo.title }}</p>
  </div>
</template>
```

Notice that we added a normal class attribute as well a the v-bind:class attribute. No problems here.

Now let's add some functionality to this. In the code snippet below, we're using the v-on directive to bind a method on to the checkbox. Whenever the checkbox is clicked, the markComplete() function is fired.

```
<template>
  <div class="todo-item" v-bind:class="{ 'is-complete':todo.completed }">
    <p>
      <input type="checkbox" v-on:change="markComplete" />
      {{ todo.title }}
    </p>
  </div>
</template>

<script>
export default {
  name: "TodoItem",
  props: ["todo"],
  methods: {
    markComplete() {
      this.todo.completed = !this.todo.completed;
    }
  }
};
</script>
```

Next, let's add a button to delete the todo. To do this, we have to access the data in our (grand)parent component. We can either use v-bind or @click, it does the same thing.

```
<template>
  <div class="todo-item" v-bind:class="{ 'is-complete':todo.completed }">
    <p>
      <input type="checkbox" v-on:change="markComplete" />
      {{ todo.title }}
      <button @click="$emit('del-todo', todo.id)" class="del">x</button>
    </p>
  </div>
</template>
```

So since we're actually passing this to our (grand)parent component, we need a very similar $emit call in our direct parent component as well, to pass the data to it's parent function. So let's do that.

```
<template>
  <div>
    <div v-bind:key="todo.id" v-for="todo in todos">
      <TodoItem v-bind:todo="todo" v-on:del-todo="$emit('del-todo', todo.id)"/>
    </div>
  </div>
</template>
```

All done. Now we have to catch this in our App component and filter the state object to remove the todoItem with the given ID. same as before, we use the v-on directive to declare the method that we want to run. In that method we simply access this.todos and set it to the new filtered value using the filter higher order function.

```
<template>
  <div id="app">
    <Todos v-bind:todos="todos" v-on:del-todo="deleteTodo"/>
  </div>
</template>

<script>
import Todos from './components/Todos.vue'

export default {
  name: 'app',
  components: {
    Todos
  },
  data() {
    return {
      todos: [
        {
          id: 1,
          title: "Todo One",
          completed: true
        },
        {
          id: 2,
          title: "Todo Two",
          completed: true
        },
        {
          id: 3,
          title: "Todo Three",
          completed: false
        },
      ]
    }
  },
  methods: {
    deleteTodo(id) {
      this.todos = this.todos.filter(todo => todo.id !== id)
    }
  }
}
</script>
```

Now let's add a form to Add Todos. We'll be exploring the v-model directive here which we will be using for the inputs. Other than that, it's all quite similar. We add a listener to the form-submit which creates a newTodo and passes it to the parent App component which can add the newTodo to the list of todos.

```
<template>
  <div>
    <form @submit="addTodo">
      <input type="text" name="title" placeholder="Add Todo.." v-model="title"/>
      <input type="submit" value="Submit" class="btn" />
    </form>
  </div>
</template>

<script>
import uuid from 'uuid'

export default {
  name: "AddTodo",
  data() {
      return {
          title: ''
      }
  },
  methods: {
      addTodo(e) {
          e.preventDefault()
          const newTodo = {
              id: uuid.v4(),
              title: this.title,
              completed: false
          }
        //   Send to parent
        this.$emit('add-todo', newTodo)
      }
  }
};
</script>


<style scoped>
form {
  display: flex;
}

input[type="text"] {
  flex: 10;
  padding: 5px;
}

input[type="submit"] {
  flex: 2;
}
</style>
```

### Rest API
Now let's move on to working with REST APIs. We'll be fetching some todos from the API and rendering them in our app.
For this we'll be using a special method called created (similar to componentDidMount() in React).

```

<template>
  <div id="app">
    <Header />
    <AddTodo v-on:add-todo="addTodo" />
    <Todos v-bind:todos="todos" v-on:del-todo="deleteTodo" />
  </div>
</template>

<script>
/* eslint-disable */

import axios from "axios";

import Todos from "./components/Todos.vue";
import Header from "./components/layout/Header";
import AddTodo from "./components/AddTodo";

export default {
  name: "app",
  components: {
    Todos,
    Header,
    AddTodo
  },
  data() {
    return {
      todos: []
    };
  },
  methods: {
    deleteTodo(id) {
      this.todos = this.todos.filter(todo => todo.id !== id);
    },
    addTodo(newTodo) {
      this.todos = [...this.todos, newTodo];
    }
  },
  created() {
    axios
      .get("https://jsonplaceholder.typicode.com/todos?_limit=5")
      .then(res => {
        this.todos = res.data;
      })
      .catch(err => alert(err));
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, Helvetica, sans-serif;
  line-height: 1.4;
}

.btn {
  display: inline-block;
  border: none;
  background: #555;
  color: #fff;
  padding: 7px 20px;
  cursor: pointer;
}

.btn:hover {
  background: #666;
}
</style>
```

