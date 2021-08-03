- There are two ways of using Vue, `widget` approach and `SPA` approach.
- It has `declarative` paradigm. We just define the end results and with the special features (we mark the dynamic content), we connect html to data and logic.
- You can add Vue by using CDN link before our own script tag. It will provide your script with `Vue` object:

```js
Vue.createApp({
  data() {
    return {
      // we return the data that our Vue app needs to be aware of
      tasks: [],
      enteredValue: "",
    };
  },
  methods: {
    addTask() {
      this.tasks.push(this.enteredValue); // Vue will establish the connection for `this` to the above-returned object
      this.enteredValue = "";
    },
    removeTask(index) {
      this.tasks.splice(index, 1);
    },
  },
}).mount("#app");
```

```html
<body>
  <div id="app">
    <div>
      <input type="text" v-model="enteredValue" />
      <button @click="addTask">Add Task</button>
    </div>
    <p v-if="tasks.length === 0">No more tasks</p>
    <ul v-else>
      <li v-for="(task, index) in tasks" :key="task" @click="removeTask(index)">
        {{ task }}
      </li>
    </ul>
  </div>
  <script src="https://unpkg.com/vue@next"></script>
  <script src="/app.js"></script>
</body>
```

# Basics: Bindings, Data, Methods, Computed, and Watchers

- First create a Vue app: `const app = Vue.createApp();`;
- Then we specify where in our HTML, we want to be manipulated by Vue: `app.mount("#target");`.
- We pass an object to `createApp` to configure the app: data and logic.
- `data` property needs to be a function that returns a data `object`.
- To output data in Vue-controlled html, you can use `{{ }}` interpolation syntax.
- Vue can interpolate arrays and objects as well.
- In Vue, we cannot interpolate html attributes with interpolation (`{{}}`). We have to use **v-bind directive** `v-bind:attributeName="dataProperty"`.

```html
<a v-bind:href="vueLink">About</a>
```

- `data` was a function but `methods` is an object with different methods. Do not use `arrow-functions`.
- In `methods`, you can have a function that returns a string and then call it in the interpolation -> `{{ generateRandomText() }}`.
- In general, you can use simple `JavaScript expressions and statements` (such as simple mathematical operation, ternary, function call, etc.) in both interpolation and v-bind directive.
- Vue will merge the data returned from data method, into Vue app instance and therefore it is available in each method in methods object by `this`.
- Another methods are also available inside each method by using `this`. Methods and computed properties will also be merged.
- To output raw HTML in the markup, you should use `v-html="htmlData"`. It has `XSS` vulnerability.
- You can listen for any html events with `v-on:eventName="doSomething"`.

```html
<button v-on:click="counter++">Add</button>
<button v-on:click="counter--">Reduce</button>
<p>{{ counter }}</p>
```

- In general, do not put logic in html like above. Instead:

```html
<button v-on:click="increment">Add</button>
<button v-on:click="decrement()">Reduce</button>
<p>{{ counter }}</p>
```

- Note that, you can call the function or not. It doesn't matter if there is no argument passed. BUT if we want to pass any argument, we should use the one with `increment(5)`.

```js
const app = Vue.createApp({
  data() {
    return {
      counter: 0,
    };
  },
  methods: {
    increment() {
      this.counter++;
    },
    decrement() {
      this.counter--;
    },
  },
});

app.mount("#app");
```

- To change data with input, you have two options:

```html
<input type="text" v-on:input="setName" />
<input type="text" v-on:input="setName($event)" />
```

- `$event` is a reserved keyword. The second method is useful when you want to pass an argument.

```js
setName(event) {
  this.name = event.target.value;
}
```

- to have a `controlled` element:

```html
<input type="text" v-bind:value="name" v-on:input="setName" />
<button v-on:click="resetInput">Reset</button>
```

```js
setName(event) {
  this.name = event.target.value;
},
resetInput() {
  this.name="";
}
```

- The shortcut for `two-way binding` is:

```html
<input type="text" v-model="name" />
```

- In forms:

```html
<form v-on:submit="submitForm">
  <input type="text" />
  <button type="submit">Submit</button>
</form>
```

```js
submitForm(event) {
  event.preventDefault();
  alert("Submitted");
}
```

- In Vue, we have `event modifiers`, which can do some of the heavy-lifting like what we did above:

```html
<form v-on:submit.prevent="submitForm">
  <input type="text" />
  <button type="submit">Submit</button>
</form>
```

```js
submitForm() {
  alert("Submitted");
}
```

- For `click` we have `click.right` modifier to only react to right clicks.
- We also have `click.stop` which when used on a child element will stop the propagation to its parent. `<input type="text" @click.stop />`.
- For `keyup` event, we have `keyup.enter` modifier which will be triggered only when `enter` is released.
- There is a directive `v-once` which if you give it to an element, the data in that element won't be changed (only evaluated once):

```html
<p v-once>{{ counter }}</p>
```

- Methods are `not good` for outputting data: `{{ outputFullname() }}`. Because the method will be re-executed whenever anything on the page changes.
- So, we should use `computed` properties, which has some dependencies and Vue will re-execute them when one of its dependencies has changed.
- Note that you should use them like a data property and Vue will execute it.

```html
<div id="app">
  <input type="text" v-model="name" />
  <p>{{ name }}</p>
  <p>{{ fullName }}</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      name: "",
    };
  },
  computed: {
    fullName() {
      return this.name + " Doe";
    },
  },
  methods: {},
});

app.mount("#app");
```

- So use methods, to calculate something that has to be changed every time something on page has changed or for event handling.
- Watchers: we define a `watch` property. Then, we re-declare a data property or a computed property as a method name inside watch propert, so whenever that data property or computed has changed, the watcher will be re-executed.
- You should not return any value from a watcher, because it will not be used in the template.

```html
<div id="app">
  <input type="text" v-model="name" />
  <p>{{ name }}</p>
  <p>{{ fullName }}</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      name: "",
      fullName: "",
    };
  },
  watch: {
    name(value) {
      this.fullName = value + " Doe";
    },
  },
});

app.mount("#app");
```

- The watcher will be called with the `new value` of that data property. The `second argument` would be the `previous value`.
- In the above example, we re-wrote the computed property with watcher but if the computed property has more than one dependency (for example firstName and lastName), we have to write two watchers for each data property! which is not good. But with computed, it will be only one.
- For example, if we want to reset counter if its value is more than 50, watchers will shine:

```js
watch: {
  counter(value) {
    if (value > 50) {
      this.counter = 0;
    }
  },
},
```

![](/md/176.jpg)

- You usually use `methods` for event binding and `computed` for outputting data that depends on other data.
- Also, as a best practice, do not put logic in html and use `computed` as much as you can. You have to add `this.` in computed to access data properties but in html, you can use data properties without `this`.
- Shorthand for `v-on:` --> `@` e.g. `<button @click="add">Add</button>`.
- Shorthand for `v-bind:` --> `:` e.g `<input :value="..">`.

# Dynamic Styling

- `Dynamic Styling`: when we want to bind style attribute, we can pass a js object:

```html
<div
  :style="{borderColor: boxASelected ? 'red' : '#ccc'}"
  @click="boxSelected('A')"
></div>
```

- `Dynamic classes`:

```html
<div
  :class="boxASelected ? 'demo active' : 'demo'"
  @click="boxSelected('A')"
></div>
```

or a better way is:

```html
<div
  :class="{demo: true, active: boxASelected}"
  @click="boxSelected('A')"
></div>
```

or even a better way is:

```html
<div
  class="demo"
  :class="{active: boxASelected}"
  @click="boxSelected('A')"
></div>
```

or

```html
<div :class="['demo', {active: boxASelected}]" @click="boxSelected('A')"></div>
```

or even a more better way (if you have complex code for choosing classes) is:

```html
<div class="demo" :class="boxAClasses" @click="boxSelected('A')"></div>
```

```js
computed:{
  boxAClasses() {
    return { active: this.boxASelected }
  }
}
```

![](/md/177.jpg)

# Conditional Rendering

- We use `v-if="condition"` directive. The condition can be truthy or falsy (not just true or false).

```html
<p v-if="tasks.length === 0">No more tasks</p>
```

- To have an `v-else` block, the element must come directly after the `v-if` element (no other element in between).

```html
<p v-if="tasks.length === 0">No more tasks</p>
<ul v-else>
  <li>...</li>
</ul>
```

- To have more conditions, use `v-if-else="anotherCondition"`:

```html
<p v-if="tasks.length === 0">No more tasks</p>
<ul v-else-if="tasks.length > 0 && tasks.length <= 5">
  <li>...</li>
</ul>
<p v-else>...</p>
```

- Instead of `v-if`, we can use `v-show`. In this case, we don't have `v-else` or `v-else-if`.
- The difference is `v-if` will remove or add element to the DOM; But `v-show` makes the `display` to `none` and the element is already in the DOM.
- As a rule of thumb, use `v-if`, unless there is an element which its visibility changes `a lot`.

# List Rendering

- Use `v-for` on the element that you want to repeat. In js, it was `of` for iterating arrays but here we use `in`.

```html
<li v-for="task in tasks">{{ task }}</li>
```

- Note that we don't have access to `task` outside `li` element.
- If we want to have access to the index:

```html
<li v-for="(task, index) in tasks">{{ index + 1 }} - {{ task }}</li>
```

- You can `iterate over objects` as well (key and index can be omitted):

```html
<li v-for="(value, key, index) in {name: 'Ben', age: 35 }">
  {{ index }} - {{ key }}: {{ value }}
</li>
```

- You can have a `range`:

```html
<li v-for="num in 10">{{ num }}</li>
<!-- It will be from 1 to 10 -->
```

- It is strongly recommended to give a `:key` attribute to the element that you want to be repeated. Because Vue re-uses DOM elements to optimize performance -> This can lead to bugs if elements contain state.
- Note that `index` is `not a good candidate` because for example index 0 is changing when you remove the first element.
- If you need `v-if` and `v-for`, don't use them on the same element. Use a wrapper with `v-if` instead.
- Most of the time, you will use `v-for` on `li`.

# Under the hood

- Vue uses `Proxy` which is a JavaScript feature.
- It is a constructor function, which receives two parameters: an object an a handler object.

```js
const data = {
  message: "",
  wrappedMessage: "''",
};
const handler = {
  set(target, key, value) {
    if (key === "message") {
      target.longMessage = "'" + value + "'";
    }
    target[key] = value;
  },
};
const proxy = new Proxy(data, handler);

proxy.message = "Hi";
console.log(proxy.longMessage);
```

- So in Vue, when a data property changes, it will update part of the DOM that the property has been used. Also watchers and computed properties will be triggered when a data property changes.

# Multiple Apps

- You can have `multiple apps`:

```js
const app1 = Vue.createApp({
  data() {
    return {
      name: "Ben",
    };
  },
});

app1.mount("#app1");

const app2 = Vue.createApp({
  data() {
    return {
      name: "Nas",
    };
  },
});

app2.mount("#app2");
```

- The data in each app is unique for their own. They are standalone apps.

```html
<div id="app1">
  <p>{{ name }}</p>
</div>
<div id="app2">
  <p>{{ name }}</p>
</div>
```

- You should not control the same html part (`template`) with two apps. And you should not use one app to control two templates.

# Template Property

- You can write template inside js code (it is not good, because we don't have syntax highlighting. We will use components later):

```js
const app3 = Vue.createApp({
  template: `
    <p>{{ name }}</p>
  `,
  data() {
    return {
      name: "Bidel",
    };
  },
});

app3.mount("#app3");
```

```html
<body>
  <div id="app3"></div>
  <script src="https://unpkg.com/vue@next"></script>
  <script src="/app.js"></script>
</body>
```

# Refs

- You can set `ref` attribute on an html element and that element will be available by `this.$refs.arbitraryRef` inside Vue.

```html
<div id="app">
  <input type="text" ref="nameInput" />
  <button @click="setText">Set Text</button>
  <p>{{ name }}</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      name: "",
    };
  },
  methods: {
    setText() {
      this.name = this.$refs.nameInput.value;
    },
  },
});

app.mount("#app");
```

- Vue creates an in-memory `virtual DOM` -> updates are made to the virtual DOM first -> `compares` it to the `previous virtual DOM` -> only `differences` are then rendered to the `actual DOM`.

# Vue App Lifecycle Hooks

![](/md/178.jpg)

```js
const app = Vue.createApp({
  data() {
    return {
      name: "",
      text: "",
    };
  },
  methods: {
    setText() {
      this.text = this.name;
    },
  },
  beforeCreate() {
    // Nothing is on the screen
    console.log("beforeCreate");
  },
  created() {
    // Nothing is on the screen
    console.log("created");
  },
  beforeMount() {
    // Nothing is on the screen
    console.log("beforeMount");
  },
  mounted() {
    // App is rendered (the above for is all on creation phase)
    console.log("mounted");
  },
  beforeUpdate() {
    console.log("beforeUpdate");
  },
  updated() {
    console.log("updated");
  },
  beforeUnmount() {
    // The app is still visible
    console.log("beforeUnmount");
  },
  unmounted() {
    console.log("unmounted");
  },
});

app.mount("#app");

setTimeout(() => {
  app.unmount();
}, 2000);
```

```html
<div id="app">
  <input type="text" v-model="name" />
  <button @click="setText">Set Text</button>
  <p>{{ text }}</p>
</div>
```

- `mounted`, `updated`, and `unmounted` are more popular ones.

# Introduction to Components

- A Vue component is essentially just another Vue app which belongs to another app. So you have to pass it a configuration object just like the one passed to Vue app.
- You should define components on app by using `app.component("name-of-component", {...});`.
- The name of component must have `-`.
- Components are reusable piece of html with connected data and logic. So they encapsulate structure, content and logic into small reusable pieces.

```js
const app = Vue.createApp({
  data() {
    return {
      friends: [
        {
          id: "manuel",
          name: "Manuel Lorenz",
          email: "manuel@localhost.com",
        },
        {
          id: "julie",
          name: "Julie Jones",
          email: "julie@localhost.com",
        },
      ],
    };
  },
});

app.component("friend-contact", {
  template: `
  <li>
    <h2>{{ friend.name }}</h2>
    <button @click="toggleDetails()">
      {{ detailsAreVisible ? 'Hide' : 'Show' }} Details
    </button>
    <ul v-if="detailsAreVisible">
      <li><strong>Email:</strong> {{ friend.email }}</li>
    </ul>
  </li>
  `,
  data() {
    return {
      detailsAreVisible: false,
      friend: {
        id: "manuel",
        name: "Manuel Lorenzo",
        email: "manuel@localhost.com",
      },
    };
  },
  methods: {
    toggleDetails() {
      this.detailsAreVisible = !this.detailsAreVisible;
    },
  },
});

app.mount("#app");
```

```html
<section id="app">
  <ul>
    <friend-contact></friend-contact>
    <friend-contact></friend-contact>
  </ul>
</section>
```

# Vue CLI: A Better Development Setup

- Why we need Vue CLI:

  - Using development web server with https protocol.
  - Hot module reloading.
  - Auto completion.
  - Error highlighting.
  - Webpack: working on different files.

- Install `@vue/cli` globally:

```
npm i -g @vue/cli
```

- To start a Vue project:

```
vue create my-app
```

- You will prompted to select some options. Choose Babel and Linter. Choose Vue 3.x. Choose ESLint with error prevention only.
- It will create the project skeleton.

```
cd my-app
npm run serve
```

- Install `vetur` extension in VSCode.
- There `node_modules`, `public` and `src` folders.
- In `public`, we have `index.html` which has a div with the id of `app`.
- In `src`, `main.js` is the main entry point:
- Unlike `React`, we don't have to (but we can) import component into another component; Instead, we can register all components that app uses in `main.js`.

```js
import { createApp } from "vue";

import App from "./App.vue";
import FriendContact from "./components/FriendContact.vue";
import NewFriend from "./components/NewFriend.vue";

const app = createApp(App);

app.component("friend-contact", FriendContact);
app.component("new-friend", NewFriend);

app.mount("#app");
```

# Props: Component Communication

- This is parent-child communication. For sibling communications, you should lift the state up to the common parent.
- In `parent` component, set props attribute using `kebab-case` convention.
- Note that, because not all props are string, we have to use `v-bind:` on those that are not string (like `is-favorite` here).

```vue
<template>
  <div>
    <new-friend @add-contact="addContact"></new-friend>
    <ul>
      <friend-contact
        v-for="{ id, name, email, isFavorite } in friends"
        :key="id"
        :name="name"
        :email-address="email"
        :is-favorite="isFavorite"
        @toggle-favorite="toggleFavorite(id)"
        @delete="deleteContact(id)"
      ></friend-contact>
      <friend-contact
        name="John"
        email-address="hi@hi.com"
        :is-favorite="true"
      ></friend-contact>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      friends: [
        {
          id: "manuel",
          name: "Manuel Lorenz",
          email: "manuel@localhost.com",
          isFavorite: false,
        },
        {
          id: "julie",
          name: "Julie Jones",
          email: "julie@localhost.com",
          isFavorite: true,
        },
      ],
    };
  },
  methods: {
    toggleFavorite(friendId) {
      const friend = this.friends.find((friend) => friend.id === friendId);
      friend.isFavorite = !friend.isFavorite;
    },
    addContact(name, email) {
      const newFriendContact = {
        id: new Date().toISOString(),
        name: name,
        email: email,
        isFavorite: false,
      };
      this.friends.push(newFriendContact);
    },
    deleteContact(friendId) {
      this.friends = this.friends.filter((friend) => friend.id !== friendId);
    },
  },
};
</script>

<style></style>
```

- Then in child, access it through `props` property array using `camelCase` convention. Or if we want to enforce prop types, we can use object. You will see the `warnings` in the browser console.
- Then in methods, we can access it by `this.` and in template, we can access it just by its name. So make sure that you don't have any clashes between props, data, and computed properties.
- With `v-bind="person"` you pass all key-value pairs inside of person as props to the component. That of course requires person to be a JavaScript object.
- Also, for documentation purposes, it is better to add `emits` property. With a validator for its data. That would be shown in the console in case of any errors.
- `FriendContact` component:

```vue
<template>
  <li>
    <h2>{{ name }} {{ isFavorite ? "*" : "" }}</h2>
    <button @click="toggleDetails">
      {{ detailsAreVisible ? "Hide" : "Show" }} Details
    </button>
    <button @click="toggleFavorite">Toggle Favorite</button>
    <button @click="$emit('delete')">Delete</button>
    <ul v-if="detailsAreVisible">
      <li>
        <strong>Email:</strong>
        {{ emailAddress }}
      </li>
    </ul>
  </li>
</template>

<script>
export default {
  // props: ["name", "emailAddress", "isFavorite"],
  props: {
    name: { type: String, required: true },
    emailAddress: {
      type: String,
      required: true,
      validator: function (value) {
        return value.includes("@");
      },
    },
    isFavorite: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
  // emits: ["toggle-favorite", "delete"],
  emits: {
    "toggle-favorite": function (data) {
      if (data) {
        console.warn("No data is required");
        return false;
      }

      return true;
    },
    delete: function () {
      return true;
    },
  },
  data() {
    return {
      detailsAreVisible: false,
    };
  },
  methods: {
    toggleDetails() {
      this.detailsAreVisible = !this.detailsAreVisible;
    },
    toggleFavorite() {
      this.$emit("toggle-favorite"); // use kebab-case for events
    },
  },
};
</script>
```

- `NewFriend` component:

```vue
<template>
  <form @submit.prevent="submitData">
    <div>
      <label>Name</label>
      <input type="text" v-model="enteredName" />
    </div>
    <div>
      <label>Phone</label>
      <input type="tel" v-model="enteredPhone" />
    </div>
    <div>
      <label>E-Mail</label>
      <input type="email" v-model="enteredEmail" />
    </div>
    <div>
      <button>Add Contact</button>
    </div>
  </form>
</template>

<script>
export default {
  emits: ["add-contact"],
  data() {
    return {
      enteredName: "",
      enteredPhone: "",
      enteredEmail: "",
    };
  },
  methods: {
    submitData() {
      this.$emit(
        "add-contact",
        this.enteredName,
        this.enteredPhone,
        this.enteredEmail
      );
    },
  },
};
</script>
```

- IMPORTANT: You should never change (mutate) a prop in the child component. If you want to change it, you have two options:

  1. You can add a new data property (with a different name than the prop) in the child component and set the initial value to the prop: `newDataProp = this.targetProp`. Then use that `newDataProp` in your template and logic.
  2. You can notify the parent to change the prop. By emiting an event with `this.$emit("custom-event");` and then use a listener attribute on child element in the parent component with `@custom-event="doSomething"`. You can also use React style, to pass the event handler as a prop to the child component.

- You can send as many values after the name of the event, when you call the `$emit` method. They will be provided to the callback function in the parent component.
- As a side note, you can use `@click` on a component and it will fall through to the top level html tag of that component. VERY IMPORTANT: The same is through for other attributes, such as `type="submit"`.

# Props Drilling: Provide + Inject

- If there is a `pass-through component` between parent and grand child.

- `App.vue`:

```vue
<template>
  <child :tasks="tasks" @select-task="toggleSelected"></child>
</template>

<script>
export default {
  data() {
    return {
      tasks: [
        {
          id: "1",
          title: "Task 1",
          selected: false,
        },
        {
          id: "2",
          title: "Task 2",
          selected: true,
        },
      ],
    };
  },
  methods: {
    toggleSelected(taskId) {
      const task = this.tasks.find((task) => task.id === taskId);
      task.selected = !task.selected;
    },
  },
};
</script>

<style></style>
```

- `Child.vue`:

```vue
<template>
  <section>
    <h2>Select a Task</h2>
    <grand-child
      :tasks="tasks"
      @select-task="$emit('select-task', $event)"
    ></grand-child>
  </section>
</template>

<script>
export default {
  props: ["tasks"],
  emits: ["select-task"],
};
</script>
```

- `GrandChild.vue`:

```vue
<template>
  <ul>
    <li
      v-for="task in tasks"
      :key="task.id"
      @click="$emit('select-task', task.id)"
    >
      {{ task.title }} {{ task.selected ? "*" : "" }}
    </li>
  </ul>
</template>

<script>
export default {
  props: ["tasks"],
  emits: ["select-task"],
};
</script>
```

- It is bad because, because neither the props is used in `Child` component and nor the event is generated. So we will `provide` it in the `parent` component and `inject` it in the `grand child` component. You can only inject things that have been defined on ancestors not on neighbors.
- Note that, we don't emit events anymore, instead we, provide the event handler from the parent to the target component.

```vue
<template>
  <child></child>
</template>

<script>
export default {
  data() {
    return {
      tasks: [
        {
          id: "1",
          title: "Task 1",
          selected: false,
        },
        {
          id: "2",
          title: "Task 2",
          selected: true,
        },
      ],
    };
  },
  provide() {
    return { tasks: this.tasks, selectTask: this.toggleSelected };
  },
  methods: {
    toggleSelected(taskId) {
      const task = this.tasks.find((task) => task.id === taskId);
      task.selected = !task.selected;
    },
  },
};
</script>

<style></style>
```

```vue
<template>
  <section>
    <h2>Select a Task</h2>
    <grand-child></grand-child>
  </section>
</template>

<script>
export default {};
</script>
```

```vue
<template>
  <ul>
    <li
      v-for="{ id, title, selected } in tasks"
      :key="id"
      @click="selectTask(id)"
    >
      {{ title }} {{ selected ? "*" : "" }}
    </li>
  </ul>
</template>

<script>
export default {
  inject: ["tasks", "selectTask"],
};
</script>
```

- Note that, do not over-use this. Instead your default should be `props` and `custom events` because the code is easier to read. But if you have 1 or more pass-through components, use `provide` and `inject`.
- Also, note that if you provide an array or an object, you have to `mutate` it if you want to change its state. Because you have provided a data property and if you change the memory address of it by setting it to a new array, the injected array is still reading from the old data property. So instead of `this.tasks = this.tasks.filter...`, you have to use `this.tasks.splice(index, 1);`.
- Also, note that you can (you shouldn't) mutate the array inside the component that it is injected!

![](/md/179.jpg)

# Registering Components

- We can register components with `component` method on app in the `main.js` file. Components will be available `globally`. You can use such components globally in every template in the Vue app.
  - The problem is when loading our app initially, the browser has to download all components at once.
  - The list can get very long and we don't know where those components have been used.
- We can register components `locally` which can be used only in the parent component by importing the component in the `script` section and registering it in `components` property as an object:

```vue
<script>
import MyComponent from "./MyComponent.vue";
export default {
  components: {
    "my-component": MyComponent,
    // MyComponent: MyComponent,
    // MyComponent, // <--- This is the best way
  },
};
</script>
```

- If you register the component with `PascalCase`, you can use it with `self-closing tag` in the template: `<MyComponent />`. You still can use `my-component` in this case (but you have to use two tags) and Vue will translate it automatically.
- If a component has been used on many places, register them globally, otherwise locally.

# Scoped Styling

- If you define your styles, in the `<style>...</style>` section of any components, it will be available `globally`. This is good for `App.vue` component.
- You can add `scoped` like `<style scoped>...</style>` to scope the style. It will only affect the current component. Not even the child components. Vue will add a random attribute to the component and then target that.

# Slots

- To receive dynamic html content in a component (just like children props in React), we will use `<slot></slot>`:

```vue
<template>
  <div>
    <slot></slot>
  </div>
</template>

<script>
export default {};
</script>

<style scoped>
div {
  ...;
}
</style>
```

- If you want to have multiple slots, you must name them (`named slots`). You can have maximum of one un-named slot (the default slot).

```vue
<template>
  <div>
    <header>
      <slot name="header">
        <h2>The default content for the slot (you can remove this line)</h2>
      </slot>
    </header>
    <slot></slot>
  </div>
</template>

<script>
export default {};
// Note that if it is empty like above, you can just remove the whole script section (it will be created automatically and you can import this component)
</script>

<style scoped>
div {
  ...;
}
</style>
```

- Then to use it:

```vue
<template>
  <section>
    <wrapper-component>
      <template v-slot:header>
        <h3>Hi: will go to slot with the name of header</h3>
      </template>
      <template v-slot:default>
        <p>This will go to the default slot</p>
      </template>
    </wrapper-component>
  </section>
</template>
```

- Shorthand for `v-slot:` is `#`: `<template #header>...<template #default>`.
- Note that, `template` has no html counterpart.
- Also note that, we can omit `<template #default>` and its closing tag. Because all the content that its corresponding slot is not specified will go to default slot.
- IMPORTANT: If you provide content between the `slot` tag in the wrapper component, it will be shown as default if the parent component that uses the wrapper component do not provide anything for that slot.
- We can have logic to examine if the wrapper component has received any content for a specific slot or not:

```vue
<template>
  <div>
    <header v-if="$slots.header">
      <slot name="header"></slot>
    </header>
    <slot></slot>
  </div>
</template>

<script>
export default {
  mounted() {
    console.log(this.$slots.header);
    console.log(this.$slots.default);
  },
};
</script>
```

## Scoped Slots

- If you want to pass props to the slots:

```vue
<template>
  <ul>
    <li v-for="task in tasks" :key="task">
      <slot :item="task" another-prop="..."></slot>
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      task: ["Task 1", "Task 2"],
    };
  },
};
</script>

<style scoped>
div {
  ...;
}
</style>
```

- Then to use it, you can use a `template` and add the slot props. Note that for slot props, we won't get automatic camelCasing like props for the component:

```vue
<template>
  <section>
    <wrapper-component>
      <template #default="slotProps">
        <h3>{{ slotProps.item }}</h3>
        <p>{{ slotProps["another-prop"] }}</p>
      </template>
    </wrapper-component>
  </section>
</template>
```

- Another simplification is to get rid of `template` and add `#default="slotProps"` to the `wrapper-component` (if the markup is only going to go to the default slot).

# Dynamic Components

- It is a cool feature. When you want to switch components based on a condition, you can use many `v-if="selectedComponent === 'target-component'"` and repeat for all the components. But you can use dynamic `component` special tag provided by Vue:

```vue
<template>
  <div>
    <the-header></the-header>
    <!-- <TheHeader /> -->
    <button @click="setSelectedComponent('active-goals')">Active Goals</button>
    <button @click="setSelectedComponent('manage-goals')">Manage Goals</button>
    <!-- <active-goals v-if="selectedComponent === 'active-goals'"></active-goals>
    <manage-goals v-if="selectedComponent === 'manage-goals'"></manage-goals>-->
    <component :is="selectedComponent"></component>
  </div>
</template>
```

- Where, `selectedComponent` is a data property and `setSelectedComponent` is a method.
- When switching, the previous component will be unmounted and will lose all its state. So if you want to keep it alive, you can wrap `component` with `keep-alive` component.

```vue
<template>
  <div>
    <the-header></the-header>
    <button @click="setSelectedComponent('active-goals')">Active Goals</button>
    <button @click="setSelectedComponent('manage-goals')">Manage Goals</button>
    <keep-alive>
      <component :is="selectedComponent"></component>
    </keep-alive>
  </div>
</template>
```

- Note that, if one of the components in the dynamic component needs props, you can use `provide and inject`. Or the other option is:

```vue
<template>
  <component :is="selectedComponent" v-bind="currentProperties"></component>
</template>

<script>
data: function () {
  return {
    selectedComponent: 'active-goals',
  }
},
computed: {
  currentProperties() {
    if (this.selectedComponent === 'active-goals') {
      return ["Goal 1", "Goal 2"]
    }
  }
},
methods:{
  setSelectedComponent(cmp) {
    this.selectedComponent = cmp
  }
}
</script>
```

# Teleport and Modals

- You can have a modal component with slot and conditionally render it inside a parent component but it is not good semantically because that modal component is an overlay and belongs to a higher position in html structure.
- You can use `teleport` Vue-provided element. `to` attribute is receiving a `css selector`.
- The `ModalAlert` code:

```vue
<template>
  <dialog open>
    <slot></slot>
  </dialog>
</template>

<style scoped>
dialog {
  margin: 0;
  position: fixed;
  top: 20vh;
  left: 30%;
  width: 40%;
  background-color: white;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 1rem;
}
</style>
```

```vue
<template>
  <div>
    <h2>Manage Tasks</h2>
    <input type="text" ref="task" />
    <button @click="setTask">Set Task</button>
    <teleport to="body">
      <modal-alert v-if="isModalOpen">
        <h2>Input is invalid!</h2>
        <p>Please enter at least a few characters ...</p>
        <button @click="closeModal">Okay</button>
      </modal-alert>
    </teleport>
  </div>
</template>

<script>
import ModalAlert from "./ModalAlert.vue";

export default {
  components: {
    ModalAlert,
  },
  data() {
    return {
      isModalOpen: false,
    };
  },
  methods: {
    setTask() {
      const enteredValue = this.$refs.task.value;
      if (enteredValue === "") {
        this.isModalOpen = true;
      }
    },
    closeModal() {
      this.isModalOpen = false;
    },
  },
};
</>
```

# Fragments

- In Vue 3.x, the `template` can have multiple top-level elements. This feature is called `fragments`.

# Folder Structure

- One component per file.
- Components that will be used in many places should be like: `BaseButton`, `AppButton` or `VButton`.
- Single instance components should be named like: `TheHeader`.
- You can have sub-folders in `components` directory to group for example `base` folder to house base components. or `layout` to house `TheHeader`. You can also have `feature` folders.

![](/md/180.jpg)

# Forms

- We have three options:

  1. listen for `@input="setThatInput"` for example on inputs and do something such as validation on typing.
  2. if we are only interested in the final value, we can add `ref="thatInput"` to the input and then extract the value in a method by `this.$refs.thatInput.value`.
  3. if we want to manipulate the data on inputs (resetting, capitalizing,...), we can have two-way binding by
     a. Using `:value="thatInput" @input="setThatInput"` and then having a data property with `thatName` or
     b. Using `v-model="thatInput"` and then having a data property with `thatName`.

- We have a modifier for `v-model.trim="userName"` to trim the data property automatically.
- When working with number inputs, note that `v-model` returns a number automatically but accessing by `ref` returns a string. If the type was text, you could force this conversion by `v-model.number="userAge"`.
- `v-model` can be bound to `dropdowns` as well (on select element).
- For `checkboxes` and `radiobuttons`, we give the same name to options and we give the `v-model="nameOfCheckOrRadio"` to each input, the `v-model` will bind the `checkbox` to `array of string` which you can set tp empty array and for `radiobutton` you can set it to `null` at the beginning. Don't forget to give unique value to each option.
- If you have a single checkbox, you can bind it to a boolean (you shouldn't give any value for it).
- For basic validation, inspect the code below (add blur event on input type, add invalid class dynamically based on a data property, and conditionally render an error message).

```vue
<template>
  <form @submit.prevent="submitForm">
    <div :class="{ invalid: userNameValidity === 'invalid' }">
      <label for="user-name">Your Name</label>
      <input
        id="user-name"
        name="user-name"
        type="text"
        v-model.trim="userName"
        @blur="validateInput"
      />
      <p v-if="userNameValidity === 'invalid'">Please enter a valid name!</p>
    </div>

    <div>
      <label for="age">Your Age (Years)</label>
      <input
        id="age"
        name="age"
        type="number"
        v-model="userAge"
        ref="ageInput"
      />
    </div>

    <div>
      <label for="referrer">How did you hear about us?</label>
      <select id="referrer" name="referrer" v-model="referrer">
        <option value="google">Google</option>
        <option value="wom">Word of mouth</option>
        <option value="newspaper">Newspaper</option>
      </select>
    </div>

    <div>
      <h2>What are you interested in?</h2>
      <div>
        <input
          id="interest-news"
          name="interest"
          type="checkbox"
          value="news"
          v-model="interest"
        />
        <label for="interest-news">News</label>
      </div>
      <div>
        <input
          id="interest-tutorials"
          name="interest"
          type="checkbox"
          value="tutorials"
          v-model="interest"
        />
        <label for="interest-tutorials">Tutorials</label>
      </div>
      <div>
        <input
          id="interest-nothing"
          name="interest"
          type="checkbox"
          value="nothing"
          v-model="interest"
        />
        <label for="interest-nothing">Nothing</label>
      </div>
    </div>

    <div>
      <h2>How do you learn?</h2>
      <div>
        <input
          id="how-video"
          name="how"
          type="radio"
          value="video"
          v-model="how"
        />
        <label for="how-video">Video Courses</label>
      </div>
      <div>
        <input
          id="how-blogs"
          name="how"
          type="radio"
          value="blogs"
          v-model="how"
        />
        <label for="how-blogs">Blogs</label>
      </div>
      <div>
        <input
          id="how-other"
          name="how"
          type="radio"
          value="other"
          v-model="how"
        />
        <label for="how-other">Other</label>
      </div>
    </div>

    <div>
      <rating-control v-model="rating"></rating-control>
    </div>

    <div>
      <input
        id="confirm-terms"
        name="confirm-terms"
        type="checkbox"
        v-model="confirm"
      />
      <label for="confirm-terms">Agree to terms of use?</label>
    </div>

    <div>
      <button>Save Data</button>
    </div>
  </form>
</template>

<script>
import RatingControl from "./RatingControl.vue";

export default {
  components: {
    RatingControl,
  },
  data() {
    return {
      userName: "",
      userAge: null,
      referrer: "wom",
      interest: [],
      how: null,
      confirm: false,
      rating: null,
      userNameValidity: "pending",
    };
  },
  methods: {
    submitForm() {
      console.log("Username: " + this.userName);
      this.userName = "";
      console.log("User age:");
      console.log(this.userAge); // this is number
      console.log(this.$refs.ageInput.value); // this is string
      this.userAge = null;
      console.log("Referrer: " + this.referrer);
      this.referrer = "wom";
      console.log("Checkboxes");
      console.log(this.interest);
      console.log("Radio buttons");
      console.log(this.how);
      this.interest = [];
      this.how = null;
      console.log("Confirm?");
      console.log(this.confirm);
      this.confirm = false;
      console.log("Rating");
      console.log(this.rating);
      this.rating = null;
    },
    validateInput() {
      if (this.userName === "") {
        this.userNameValidity = "invalid";
      } else {
        this.userNameValidity = "valid";
      }
    },
  },
};
</script>
```

## Using v-model on Custom Components

- Vue will set `modelValue` prop if you use `v-model` on a custom component.
- Also, it will listen to a very specific event `update:modelValue`.
- The custom component is:

```vue
<template>
  <ul>
    <li :class="{ active: modelValue === 'poor' }">
      <button type="button" @click="activate('poor')">Poor</button>
    </li>
    <li :class="{ active: modelValue === 'average' }">
      <button type="button" @click="activate('average')">Average</button>
    </li>
    <li :class="{ active: modelValue === 'great' }">
      <button type="button" @click="activate('great')">Great</button>
    </li>
  </ul>
</template>

<script>
export default {
  props: ["modelValue"],
  emits: ["update:modelValue"],
  methods: {
    activate(option) {
      this.$emit("update:modelValue", option);
    },
  },
};
</script>
```

# HTTP Requests

- You can use `fetch` or `axios`.
- The best place to load data for a component is `mounted` hook.

# Router

- Install `npm i vue-router`.
- Create a `router.js` file in the root of `src`.
- In the `routes` array property of the router, we register all the routes and their configurations (each object is one route).
- By importing components in `router.js`, we don't need to register them globally or locally anymore.
- `router-view` in the `App.vue` is a placeholder to render the components based on their routes. We can have more than one (we should name them. One can be unnamed - the default one) in the same component and therefore we need to give different components to different `router-view` for each route. If we give it just one (like the `NotFound` in the example below), it will go to the default one.
- We don't use anchor tags and instead we use `router-link` (built-in component in vue-router) with the `to` prop (it will be compiled to anchor tag).

## Styling

- When clicking on `router-link`, `router-link-active` and `router-link-exact-active` classes will be added to the anchor tag which can be used to style. `router-link-exact-active` will only be applied if the current path is an exact match. `router-link-active` will be applied to parents as well (refer to nesting few lines below). Note that if `/teams` and `/teams/t1` are not related if they are both root routes.
- You can modify the default name (`router-link-active`) by `linkActiveClass` property in the `createRouter` object.

## Programatic Navigation

- To `navigate programatically`, we have access to `this.$router` inside any components. We can use `this.$router.push("/teams");`. We have access to other methods such as `back()` and `forward()` on `this.$router` as well.
- If you don't want to go back (like forms), use `.replace` method instead of `push`.

## Nested Routing

- Most of the time we need `{ path: "/teams/:teamId", component: TeamDetails}` after `{ path: "/teams", component: TeamsList}` in the `routes` property of the `createRouter` object to render it in the `router-view` in the `App`. But if we want to render `TeamDetails`, in a `router-view` inside `TeamsList`, we have to nest it as `children` and create a `router-view` in its parent (`TeamsList`).

## Dynamic Params as Props

- Note that, if we set `props: true` on route object above, the `teamId` is accessible in the `TeamDetails` as `props` (actually vue-router will pass all the dynamic parameters as props). Another way is that since this component is loaded through the router, we can inspect `this.$route.params.teamId`.
- Note that if the component had a wrapper, then we couldn't access the `this.$route` and this is why the first option is better. Then the `TeamDetails` is re-usable and is not bound to routing.

## A Problem

- If in `TeamDetails`, we want to go to another `TeamDetails` with a different Id, since Vue does not destory the component, the route will be changed but the methods in `created` to load the team details won't be executed.
- **solution**: using `watch` to watch `$route` or the `teamId` prop. Another solution is using computed -> because when props is changed, if props has used in a computed, it will be run too.

## Alias

- You can have a `alias: "/"` property on the route object to load the componet(s) on a different route. Note that in this case the page will be accessible by two paths; so redirect is a better option.

## Naming Routes

- You can name your routes and then when navigating to them (by `to` or by `push`), instead of a lengthy string, you can use an object like: `{ name: "team-details", params: { teamId: this.id } }` instead of `'/team/' + this.id `. It is a better approach because you change your paths without changing the code everywhere. In this way, you can use `query parameter` easily: `{ name: "team-details", params: { teamId: this.id }, query: { sort: 'asc' } }`.

## Query Params

- You can see the current path as string by `this.$route.path` and query parameters as an object by `this.$route.query`. Note that the query parameters won't set as props even if the `props: true`.

## Scroll Behavior

- The `scrollBehavior` method is being called by `to`, `from`, and `savedPosition` objects and what it return (`{left: ..., top: ...}`) will specify the scroll behavior when the page changes.
- The `savedPosition` is the scroll coordinates when we changed to a new page and then came back to that page by back button. So most of the time, we want to scroll top for the new pages and if it is by back button, we want to go back to it.

## Route Guards

- Methods that will be called automatically by vue-router when a navigation action is started.
- `beforeEach` is the global `beforeEach` and receives a function which receives `to`, `from`, and `next` as parameters.
  - If we call `next(false);` it will cancel the navigation.
  - `next()` or `next(true)` will proceed.
  - `next('/teams')` or `next({ name: 'team-details', params: { teamId: this.id } })` will redirect.
- If you want to have logic before a certain route, you can check that `if (to.name === "blah") {` in the global `beforeEach` or you can use `beforeEnter` in that specific route. Or even you can have `beforeRouteEnter` hook on the component level. Which will be executed in this order: global -> route config level -> component level.
- `beforeRouteUpdate` is another component level hook, when the route is updated but the component stays like what is happening for `TeamDetails`.
- We have global `afterEach` function as well. It runs when the navigation has been confirmed, so you don't have access to `next`. So it can be used to send analytic data to your backend.
- If we want to cancel a navigation from inside a component, we can make use of `beforeRouteLeave` hook. Examples are when you have a form and you want to prompt the user if he is sure to leave. It will be called before `beforeEach` and `beforeEnter` guards.

## Redirecting Based on Origin

- Just pass a query parameter and then programatically route the user based on the existence of a query param: `const redirectURL = '/' + (this.$route.query.redirect || 'coaches');`.

## Organizing

- When using routings, next to `components` folder, create a `pages` folder and insert components that have been loaded through vue-router.

```js
import { createRouter, createWebHistory } from "vue-router";

import TeamsList from "./pages/TeamsList.vue";
import TeamsFooter from "./pages/TeamsFooter.vue";
import TeamDetails from "./components/teams/TeamDetails.vue";

import UsersList from "./pages/UsersList.vue";
import UsersFooter from "./pages/UsersFooter.vue";

import NotFound from "./pages/NotFound.vue";

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: "/", redirect: "/teams" }, // To redirect!
    {
      name: "teams",
      path: "/teams", // the root routes start with /
      meta: { needsAuth: true },
      components: { default: TeamsList, footer: TeamsFooter }, // If App component has more than one router-view
      children: [
        // children takes an array of route objects like the root routes
        {
          name: "team-details",
          path: ":teamId",
          component: TeamDetails,
          props: true,
        }, // /teams/t1
      ],
    }, // our-domain.com/teams => TeamsList
    {
      path: "/users",
      components: {
        default: UsersList,
        footer: UsersFooter,
      },
      beforeEnter(to, from, next) {
        console.log("users beforeEnter");
        console.log(to, from);
        next();
      },
    },
    { path: "/:notFound(.*)", component: NotFound }, // or you can redirect. This line should come last
  ],
  linkActiveClass: "active", // we can change the default name (router-link-active) to what we want to define
  scrollBehavior(_, _2, savedPosition) {
    // to satisfy IDE not to complain about un-used variables
    // console.log(to, from, savedPosition);
    if (savedPosition) {
      return savedPosition;
    }
    return { left: 0, top: 0 };
  },
});

router.beforeEach(function (to, from, next) {
  console.log("Global beforeEach");
  console.log(to, from);

  if (to.meta.needsAuth) {
    console.log("Needs auth!");
  }

  next();
});

router.afterEach(function (to, from) {
  // sending analytics data
  console.log("Global afterEach");
  console.log(to, from);
});

export default router;
```

- Connect router to in the `main.js`:

```js
import { createApp } from "vue";

import App from "./App.vue";
import router from "./router.js";

const app = createApp(App);

app.use(router);

app.mount("#app");
```

- `App.vue`:

```vue
<template>
  <the-navigation></the-navigation>
  <main>
    <router-view></router-view>
    <!-- this component is availabe because of vue-router package -->
  </main>
  <footer>
    <router-view name="footer"></router-view>
  </footer>
</template>

<script>
import TheNavigation from "./components/nav/TheNavigation.vue";

export default {
  components: {
    TheNavigation,
  },
  data() {
    return {
      teams: [
        { id: "t1", name: "Frontend Engineers", members: ["u1", "u2"] },
        { id: "t2", name: "Backend Engineers", members: ["u1", "u2", "u3"] },
        { id: "t3", name: "Client Consulting", members: ["u4", "u5"] },
      ],
      users: [
        { id: "u1", fullName: "Max Schwarz", role: "Engineer" },
        { id: "u2", fullName: "Praveen Kumar", role: "Engineer" },
        { id: "u3", fullName: "Julie Jones", role: "Engineer" },
        { id: "u4", fullName: "Alex Blackfield", role: "Consultant" },
        { id: "u5", fullName: "Marie Smith", role: "Consultant" },
      ],
    };
  },
  provide() {
    return {
      teams: this.teams,
      users: this.users,
    };
  },
};
</script>
```

- `TheNavigation.vue`:

```vue
<template>
  <header>
    <nav>
      <ul>
        <li>
          <router-link to="/teams">Teams</router-link>
        </li>
        <li>
          <router-link to="/users">Users</router-link>
        </li>
      </ul>
    </nav>
  </header>
</template>
```

- `TeamsList.vue`:

```vue
<template>
  <router-view></router-view>
  <ul>
    <teams-item
      v-for="team in teams"
      :key="team.id"
      :id="team.id"
      :name="team.name"
      :member-count="team.members.length"
    ></teams-item>
  </ul>
</template>

<script>
import TeamsItem from "../components/teams/TeamsItem.vue";

export default {
  components: {
    TeamsItem,
  },
  inject: ["teams"],
};
</script>
```

- `TeamDetails.vue`:

```vue
<template>
  <section>
    <h2>{{ teamName }}</h2>
    <ul>
      <user-item
        v-for="member in members"
        :key="member.id"
        :name="member.fullName"
        :role="member.role"
      ></user-item>
    </ul>
    <router-link to="/teams/t2">Go to Team 2</router-link>
  </section>
</template>

<script>
import UserItem from "../users/UserItem.vue";

export default {
  inject: ["users", "teams"],
  props: ["teamId"],
  components: {
    UserItem,
  },
  data() {
    return {
      teamName: "",
      members: [],
    };
  },
  methods: {
    loadTeamDetails(teamId) {
      const selectedTeam = this.teams.find((team) => team.id === teamId);
      const members = selectedTeam.members;
      const selectedMembers = [];
      for (const member of members) {
        const selectedUser = this.users.find((user) => user.id === member);
        selectedMembers.push(selectedUser);
      }
      this.members = selectedMembers;
      this.teamName = selectedTeam.name;
    },
  },
  created() {
    // this.$route.path // /teams/t1
    this.loadTeamDetails(this.teamId); // or (this.$route.params.teamId)
    console.log(this.$route.query);
  },
  beforeRouteUpdate(to, from, next) {
    console.log("TeamDetails Cmp beforeRouteUpdate");
    console.log(to, from);
    // this.loadTeamDetails(to.params.teamId); // If we write this, we don't need to use watch but again we bound our component to routing; so not good
    next();
  },
  watch: {
    teamId(newId) {
      // or `$route(newRoute) {` if you have used this.$route.params.teamId
      this.loadTeamDetails(newId); // or this.loadTeamDetails(newRoute.params.teamId);
    },
  },
};
</script>
```

- `UsersList.vue`:

```vue
<template>
  <button @click="confirmInput">Confirm</button>
  <button @click="saveChanges">Save Changes</button>
  <ul>
    <user-item
      v-for="user in users"
      :key="user.id"
      :name="user.fullName"
      :role="user.role"
    ></user-item>
  </ul>
</template>

<script>
import UserItem from "../components/users/UserItem.vue";

export default {
  components: {
    UserItem,
  },
  inject: ["users"],
  data() {
    return { changesSaved: false };
  },
  methods: {
    confirmInput() {
      // do something
      this.$router.push("/teams");
    },
    saveChanges() {
      this.changesSaved = true;
    },
  },
  beforeRouteEnter(to, from, next) {
    console.log("UsersList Cmp beforeRouteEnter");
    console.log(to, from);
    next();
  },
  beforeRouteLeave(to, from, next) {
    console.log("UsersList Cmp beforeRouteLeave");
    console.log(to, from);

    if (this.changesSaved) {
      next();
    } else {
      const userWantsToLeave = confirm(
        "Are you sure? You got unsaved changes!"
      );
      next(userWantsToLeave);
    }
  },
  unmounted() {
    console.log("unmounted");
  },
};
</script>
```

# Transitions and Animations

- You can define an animated class and based on a data property conditionally give it to a block and then on that animated class define a rule like `transform: translateX(100px);` and in the base class of that element `transition: transform 0.3s ease-out;`.
- For more advanced animations:

```css
.block {
}

.animated {
  animation: slide-scale 0.3s ease-out forwards;
}

@keyframes slide-scale {
  0% {
    transform: translateX(0) scale(1);
  }
  70% {
    transform: translateX(100px) scale(1.1);
  }
  100% {
    transform: translateX(150px) scale(1);
  }
}
```

- The problem is that it is hard to animate disappearance of elements, because the element is gone by Vue and the class will not have any effects.
- When you wrap the element with `transition` built-in component, Vue will add three CSS utility classes for appearance of elements and three for disappearance of them. Then you can define those classes. Vue will analyze your code for those classes, find out the duration and remove the element when your animations have finished.
- `transition` must have only one direct child (except one exception).

![](/md/181.jpg)

- For example:

```vue
<template>
  <div class="container">
    <transition>
      <p v-if="paraIsVisible">This is only sometimes visible...</p>
    </transition>
    <button @click="toggleParagraph">Toggle Paragraph</button>
  </div>
</template>

<script>
...
</script>

<style>
.v-enter-from {
  opacity: 0;
  transform: translateY(-30px);
}

.v-enter-active {
  animation: all 2s ease-out;
}

.v-enter-to {
  opacity: 1;
  transform: translateY(0);
}

.v-leave-from {
  opacity: 1;
  transform: translateY(0);
}

.v-leave-active {
  transition: all 0.3s ease-in;
}

.v-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>
```

- If you want to have animation (instead of transition), you don't need `*-from` and `*-to` and just use `*-active`. Note that we don't need to add `forwards` to animation as well, because that `*-active` class will be removed from the element:

```vue
<style>
.v-enter-active {
  animation: slide-scale 0.3s ease-out;
}
.v-leave-active {
  animation: slide-scale 0.3s ease-in;
}
</style>
```

- If you have multiple `transition` components in a component, you should name them (if you want different types of animation) and corresponding classes will be added instead of `v-`:
- Also note that, the second transition has two children but since only one of them is visible at a time, it is OK. But in order to make Vue understand that, we should use `v-else` and not two `v-if`s. Also, because we don't want to have both switch animations on the same time, we set `mode="out-in"`; so first the first component will be removed and then the new component will be added.

```vue
<template>
  <div class="container">
    <transition>
      <p v-if="paraIsVisible">This is only sometimes visible...</p>
    </transition>
    <button @click="toggleParagraph">Toggle Paragraph</button>
  </div>
  <div class="container">
    <transition name="fade-button" mode="out-in">
      <button @click="showUsers" v-if="!usersAreVisible">Show Users</button>
      <button @click="hideUsers" v-else>Hide Users</button>
    </transition>
  </div>
</template>

<style>
/* ... */
.fade-button-enter-from,
.fade-button-leave-to {
  opacity: 0;
}

.fade-button-enter-active {
  transition: opacity 0.3s ease-out;
}

.fade-button-leave-active {
  transition: opacity 0.3s ease-in;
}

.fade-button-enter-to,
.fade-button-leave-from {
  opacity: 1;
}
</style>
```

- You also can modify the dynamic class names if you set `<transition enter-to-class="some-class" enter-active-class="another-class"><p v-if="paraIsVisible">...</p></transition>`. This is beneficial if you use third party CSS libraries.
- A _potential problem_ is that if you use `transition` to wrap a user-defined component and it has two root elements (like modal that has backdrop and a card), those classes will be fall through the first element and might cause unexpected behavior. So better to use `transition` inside that user-defined component.
- Using `transition` component will give us some hooks to use transition events. These callbacks will be called by the element and we can use them to control animation with JS instead of CSS:

```vue
<template>
  <div class="container">
    <transition
      name="para"
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter"
      @before-leave="beforeLeave"
      @leave="leave"
      @after-leave="afterLeave"
      @enter-cancelled="enterCancelled"
      @leave-cancelled="leaveCancelled"
    >
      <p v-if="paraIsVisible">This is only sometimes visible...</p>
    </transition>
    <button @click="toggleParagraph">Toggle Paragraph</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      paraIsVisible: false,
    };
  },
  methods: {
    toggleParagraph() {
      this.paraIsVisible = !this.paraIsVisible;
    },
    beforeEnter(el) {
      console.log('beforeEnter');
      console.log(el);
    },
    enter(el) {
      console.log('enter');
      console.log(el);
    },
    afterEnter(el) {
      console.log('afterEnter');
      console.log(el);
    },
    beforeLeave(el) {
      console.log('beforeLeave');
      console.log(el);
    },
    leave(el) {
      console.log('leave');
      console.log(el);
    },
    afterLeave(el) {
      console.log('afterLeave');
      console.log(el);
    },
    enterCancelled(el) {
      console.log('enterCancelled');
      console.log(el);
    },
    leaveCancelled(el) {
      console.log('leaveCancelled');
      console.log(el);
    },
}
</script>
```

- So if you want to leverage JS (for example when you want an animation library), remove all those classes `*-enter-to`, ... and rely on hooks.
- Note that, the second parameter for `enter` and `leave` is `done` function which will tell Vue, when the animation is done and it is a cue to run `after-enter` or `after-leave` methods.
- If we remove the component before its entering animation is finished, the `enter-cancelled` event will be emitted.
- Note that `:css="false"` is optional but by setting this, we are telling Vue that don't even look for those CSS animation classes. But it is better to remove them.

```vue
<template>
  <div class="container">
    <transition
      :css="false"
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter"
      @before-leave="beforeLeave"
      @leave="leave"
      @after-leave="afterLeave"
      @enter-cancelled="enterCancelled"
      @leave-cancelled="leaveCancelled"
    >
      <p v-if="paraIsVisible">This is only sometimes visible...</p>
    </transition>
    <button @click="toggleParagraph">Toggle Paragraph</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      paraIsVisible: false,
      enterInterval: null,
      leaveInterval: null
    };
  },
  methods: {
    toggleParagraph() {
      this.paraIsVisible = !this.paraIsVisible;
    },
    enterCancelled(el) {
      clearInterval(this.enterInterval);
    },
    leaveCancelled(el) {
      clearInterval(this.leaveInterval);
    },
    beforeEnter(el) {
      el.style.opacity = 0;
    },
    enter(el, done) {
      let round = 1;
      this.enterInterval = setInterval(() => { // this must be arrow
        el.style.opacity = round * 0.01;
        round++;
        if (round > 100) {
          clearInterval(this.enterInterval);
          done();
        }
      }, 20);
    },
    beforeLeave(el) {
      el.style.opacity = 1;
    },
    leave(el, done) {
      let round = 1;
      this.leaveInterval = setInterval(() => { // this must be arrow
        el.style.opacity = 1 - round * 0.01;
        round++;
        if (round > 100) {
          clearInterval(this.leaveInterval);
          done();
        }
      }, 20);
    },
};
</script>
```

## Animated Lists

- To animate when an item is removed or added, we can't wrap `transition` because it needs one direct child and we have multiple list items so we will wrap it inside `transition-group` instead.
- Unlike `transition`, `transition-group` will be rendered to the DOM, so you should define the tag by `tag` prop.
- Note that we will have access to another CSS class which is `*-move` which animates the moving of other elements when adding or deleting an element.

```vue
<template>
  <transition-group tag="ul" name="user-list">
    <li v-for="user in users" :key="user" @click="removeUser(user)">
      {{ user }}
    </li>
  </transition-group>
  <div>
    <input type="text" ref="userNameInput" />
    <button @click="addUser">Add User</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: ["Max", "Manu", "Julie", "Angela", "Michael"],
    };
  },
  methods: {
    addUser() {
      const enteredUserName = this.$refs.userNameInput.value;
      this.users.unshift(enteredUserName);
    },
    removeUser(user) {
      this.users = this.users.filter((usr) => usr !== user);
    },
  },
};
</script>

<style scoped>
.user-list-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.user-list-enter-active {
  transition: all 1s ease-out;
}

.user-list-enter-to,
.user-list-leave-from {
  opacity: 1;
  transform: translateX(0);
}

.user-list-leave-active {
  transition: all 1s ease-in;
  position: absolute; /* If you omit this other elements will jump on remove */
}

.user-list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.user-list-move {
  transition: transform 0.8s ease;
}
</style>
```

## Animated Routes

- Let's say we have one `router-view` inside our `App.vue`. We can't wrap it with `transition` anymore; instead we should use a pattern like below (since `router-view` uses slots under the hood, we can get the `slotProps` and we are interested in `Component` part of it):

```vue
<template>
  <router-view v-slot="slotProps">
    <transition name="route" mode="out-in">
      <component :is="slotProps.Component"></component>
    </transition>
  </router-view>
</template>

<style>
.route-enter-from,
.route-leave-to {
  opacity: 0;
}

.route-enter-active {
  transition: opacity 0.3s ease-out;
}

.route-leave-active {
  transition: opacity 0.3s ease-in;
}

.route-enter-to,
.route-leave-from {
  opacity: 1;
}
</style>
```

- Note that if you do not want animation on the startup, you can use:

```js
router.isReady().then(function () {
  app.mount("#app");
});
```

- If you want to transition between routes, you need to ensure that your route components have exactly one root element. Because you must not forget that `<transition>` needs exactly one child element (with the special exceptions you learned about in this module). If your route component has multiple root elements, `<transition>` in the end has multiple children and that is the problem.

# Vuex

- Replacing `provide/inject`. Because it has some drawbacks like
  - getting some components really large (fat components like App.vue with a lot of data and logic) and also
  - we saw that we might have some weird (unpredictable) issues with `provide/inject`.
  - We can edit an injected array inside the component that it is injected into.

![](/md/182.jpg)

- Install `npm i vuex`.
- By connecting store to the app, we will have access to `$store` everywhere. To read data, we can use `$store.state.isLoggedIn` for example.
- We can (shouldn't) change the state in store from any component easily by `this.$store.state.isLoggedIn = true;` for example.
- So we can use state everywhere without any props or providing.

## Mutations

- We should change the state via mutations; because we will make sure that all components use the same mechanism to change data (for example App.vue and ChangeCounter here). It is more maintainable to have the logic for changing state in one place.

![](/md/183.jpg)

- So apart from `state` method, we have `mutations` in `createStore` which receives an object of methods which we can change the data with.
- No payload:
  - Each mutation method receives the current `state` as the first parameter. So inside such `mutations` method, we can have `login(state) { state.login = true; }`.
  - Then to use it in a component, we will write `this.$store.commit("login");`.
- With payload:

  - If we want to pass data, for example we have an _increase_ method which receives the amount of increment; `mutations methods` will receive a second parameter which can be anything that we pass: `increase(state, payload) {state.counter += payload.value; }`.
  - Then to use it, `this.$store.commit("increase", { value: 10 });`.
  - There is an alternative: `this.$store.commit({ type: "increase", value: 10 });`.

## Getters

- We should get the state via getters; because if we do some computations on the data in the store, we will make sure that all the components will use the same mechanism which is more maintainable (for example TheCounter and FavoriteValue here).
- They are like computed properties defined in the store.

![](/md/184.jpg)

- So apart from `state` method and `mutations` object, we will add a `getters` in `createStore` which receives an object of methods which we can read the data with.
- Each getter method receives the current `state` as the first parameter and other `getters` as the second parameter (if we want to get access to other getters. Like finalCounter and normalizedCounter.). So inside such `getters` method, we can have `finalCounter(state) { return state.counter * 2; }`.
- Then in a component, we will read it like: `this.$store.getters.finalCounter`.

## Actions

- Mutations need to be instantaneous. Because each mutation receives the latest state. And if a mutation is committed but not finished, it will lead to error.
- To deal with `async` code, we have to use actions between components and mutations. As a best practice, even when dealing with `sync` code, it is still better to use actions:

![](/md/185.jpg)

- So apart from `state` method and `mutations` and `getters` objects, we will add an `actions` in `createStore` which receives an object of methods which we can `dispatch` an action with.
- Note that most of the time, the name of `action` and `mutation` methods are the same.
- Each action method receives a `context` object as the first parameter and also a second parameter which can be anything that we pass (`payload`). So inside such `actions` method, we can have `increase(context, payload) { context.commit("increase", payload); }`.
- Note that you can run async code in actions and most probably finally you will commit a mutation with the same name as the action method.
- Then in your components: `this.$store.dispatch("increase", { value: 10 });` or alternatively `this.$store.dispatch({ type: "increase", value: 10 });`.
- Note that the `context` is actually the store and therefor we have access to `dispatch` (so we can dispatch an action inside another action), `getters` and `state`. But never manipulate state inside an action. Always use a mutation for that.
- A very important `difference` with `Redux`: here, actions (unlike action-creators in redux which returns an object or a function (mostly async funcs)) can be async. So you don't need to define central state for `isLoading` or `error` states. You simply can have a local state (`isLoading`) and await the async action and after it, set isLoading to false.
- So in general, you don't handle errors in actions in Vue.

## Mapper Helpers

- To access the data from store, instead of defining a computed property and return `this.$store.getters...`, you can `import { mapGetters } from 'vuex';`. and then spread it inside the computed: `computed: {...mapGetters(["finalCounter"])}`. You can give as many getters that you want and they will be defined as computed properties.
- If you want to rename the computer props, instead of an array, pass an object. The keys will be the new names and the values should be the getters in store.
- To access the actions, you can `import { mapActions } from 'vuex';`. and then spread it inside the methods: `methods: {...mapActions(['increment', 'increase'])}`. You can give as many actions that you want and they will be defined as methods.
- If you want to rename the methods, instead of an array, pass an object. The keys will be the new names and the values should be the actions in store.

## Modules

- To keep your code managable, you can add modules for different features. Some state can stay on the `root` module (or not, up to you).
- Here we outsourced counter into a separate module.
- We have to give it an identifier in the `modules` property of the main createStore.
- All modules will be merged into the root/global store on the same levels of other things in the store.
- **IMPORTANT** note that the `state` for each module is scoped to that module (In getters, mutations, and actions (by context.state)).
- If you want to get access to other modules' state or root state, for example
  - in `getters`' methods, you can have access to them by receiving more arguments `aGetterMethod(state, getters, rootState, rootGetters)`.
  - Also, in `actions`' methods, we have context which has access to `context.rootState` and `context.rootGetters["moduleIdentifier/getterMethod"]`.
  - In `mutations` methods, I think we don't have access and we should use `actions` instead.
- If you set `namespaced: true` on the module, not only the state but also getters, actions, and mutations will be scoped. So modules will be completely separated from each other. So you won't have any name clashes. Note that if you namespace the module, you have to specify the identifier of the module to be able to use its getters or actions.

  - In getters: `computed: { finalCounter () { return this.$store.getters['counter/finalCounter']; } }` or `computed: { ...mapGetters("counter", ["finalCounter"]) }`.
  - In actions: `methods: { incrementWithNewName() { this.$store.dispatch('counter/increment'); } }` or `methods: { ...mapActions("counter", { incrementWithNewName: "increment" } ) }`.

- For `auth` it is better not to put it in a module or if you put it in a module, do not namespace it.

## Use Store in Route Guards

- You simply can import store and use `store.getters.isAuthenticated` in `beforeEach` route guard. But this way, you might introduce cyclic dependency between store and router:
  - `export const appRouter = app.$router;`: This way you can import appRouter rather than the full router config (alternatively you can access `router.app.$store` in route guards which has some considerations).
  - Do not do routing inside actions and do it inside components. Instead define a new state in store and when you want to do routing, change that state inside your actions. Then in your `App.vue`, create a computed property out of that piece of store (by defining a getter of course) and then watch it for any change and route based on it. Since you might not handle errors in actions, maybe you don't want to do routing in actions too.

## Local Storage

- In `login` or `signup` action, before commiting the mutation, set the local storage (the token, userId, and the expiration date). Also, set a timeout to dispatch logout action, when token expires.
- Create an action `tryLogin` which reaches to local storage to read data and commits the mutation. Also, it sets the timer to automatically logout as well and if there is no time left, just return.
- In `created` method of the `App`, we dispatch the `tryLogin` action.
- In `logout` action, clear the local storage and clear the timer (it should have defined outside the actions object to have global scope).

- Create `store/index.js`:

```js
import { createStore } from "vuex";

import counterModule from "./modules/counter/index.js";

import rootMutations from "./mutations.js";
import rootGetters from "./getters.js";
import rootActions from "./actions.js";

const store = createStore({
  modules: {
    counter: counterModule,
  },
  state() {
    return {
      // this is the application-wide data
      isLoggedIn: false,
    };
  },
  mutations: rootMutations,
  actions: rootActions,
  getters: rootGetters,
});

export default store;
```

- `mutations.js`:

```js
export default {
  setAuth(state, payload) {
    state.isLoggedIn = payload.isAuth;
  },
};
```

- `getters.js`:

```js
export default {
  userIsAuthenticated(state) {
    return state.isLoggedIn;
  },
};
```

- `actions.js`:

```js
export default {
  login(context) {
    context.commit("setAuth", { isAuth: true });
  },
  logout(context) {
    context.commit("setAuth", { isAuth: false });
  },
};
```

- In the `modules` folder inside `store` folder, create a `counter` folder. Inside that `index.js`:

```js
import counterMutations from "./mutations.js";
import counterGetters from "./getters.js";
import counterActions from "./actions.js";

export default {
  namespaced: true,
  state() {
    return {
      counter: 0,
    };
  },
  mutations: counterMutations,
  getters: counterGetters,
  actions: counterActions,
};
```

- `mutations.js`:

```js
export default {
  increment(state) {
    state.counter = state.counter + 2;
  },
  increase(state, payload) {
    console.log(state);
    state.counter = state.counter + payload.value;
  },
};
```

- `getters.js`:

```js
export default {
  testAuth(state) {
    return state.isLoggedIn; // this will not work because the state in counter module is scoped
  },
  finalCounter(state) {
    return state.counter * 3;
  },
  normalizedCounter(_, getters) {
    const finalCounter = getters.finalCounter;
    if (finalCounter < 0) {
      return 0;
    }
    if (finalCounter > 100) {
      return 100;
    }
    return finalCounter;
  },
};
```

- `actions.js`:

```js
export default {
  increment(context) {
    setTimeout(function () {
      context.commit("increment");
    }, 2000);
  },
  increase(context, payload) {
    console.log(context);
    context.commit("increase", payload);
  },
  login() {}, // since it is namespaced, we can have actions with the same name as the root store
};
```

- Then wire it in the `main.js`:

```js
import { createApp } from "vue";

import App from "./App.vue";
import store from "./store/index.js";

const app = createApp(App);

app.use(store);

app.mount("#app");
```

- Then to use it inside `App.vue`:

```vue
<template>
  <base-container title="Vuex" v-if="isAuth">
    <the-counter></the-counter>
    <favorite-value></favorite-value>
    <button @click="addOne">Add 10</button>
    <change-counter></change-counter>
  </base-container>
  <base-container title="Auth">
    <user-auth></user-auth>
  </base-container>
</template>

<script>
import BaseContainer from "./components/BaseContainer.vue";
import TheCounter from "./components/TheCounter.vue";
import ChangeCounter from "./components/ChangeCounter.vue";
import FavoriteValue from "./components/FavoriteValue.vue";
import UserAuth from "./components/UserAuth.vue";

export default {
  components: {
    BaseContainer,
    TheCounter,
    ChangeCounter,
    FavoriteValue,
    UserAuth,
  },
  computed: {
    isAuth() {
      return this.$store.getters.userIsAuthenticated;
    },
  },
  methods: {
    addOne() {
      // this.$store.dispatch('counter/increase', { value: 10 });
      this.$store.dispatch({
        type: "counter/increase",
        value: 10,
      });
    },
  },
};
</script>
```

- and in `ChangeCounter.vue`:

```vue
<template>
  <button @click="incrementWithNewName">Add 2</button>
  <button @click="increase({ value: 11 })">Add 11</button>
</template>

<script>
import { mapActions } from "vuex";

export default {
  methods: {
    // incrementWithNewName() {
    //   this.$store.dispatch('counter/increment');
    // }
    // ...mapActions("counter", ['increment', 'increase'])
    ...mapActions("counter", {
      incrementWithNewName: "increment", // you can rename it here
      increase: "increase",
    }),
  },
};
</script>
```

- In `TheCounter` component:

```vue
<template>
  <h3>{{ finalCounter }}</h3>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  computed: {
    // finalCounter() {
    //   return this.$store.getters['counter/finalCounter'];
    // },
    ...mapGetters("counter", ["finalCounter"]),
  },
};
</script>
```

- and in `FavoriteValue` component:

```vue
<template>
  <h3>{{ counter }}</h3>
  <p>We do more...</p>
</template>

<script>
export default {
  computed: {
    counter() {
      return this.$store.getters["counter/normalizedCounter"];
    },
  },
};
</script>
```

- For `UserAuth` component:

```vue
<template>
  <button @click="login" v-if="!isAuth">Login</button>
  <button @click="logout" v-if="isAuth">Logout</button>
  <p>{{ isTestAuth }}</p>
</template>

<script>
export default {
  methods: {
    login() {
      this.$store.dispatch("login");
    },
    logout() {
      this.$store.dispatch("logout");
    },
  },
  computed: {
    isAuth() {
      return this.$store.getters.userIsAuthenticated;
    },
    isTestAuth() {
      return this.$store.getters.testAuth; // the getter is defined in the counter module
    },
  },
};
</script>
```

# Optimization and Build

- Instead of importing all components (which makes the built JS file, bigger and more time-consuming to load), we can lazily load the components that we don't need on all other pages such as modal. So instead of `import BaseModal from "./components/Base/BaseModal.vue";` use `const BaseModal = defineAsyncComponent(()=> import("./components/Base/BaseModal.vue"));`. You should import `defineAsyncComponent` from `vue`.
- Specially this is useful in `router.js` which we can lazily import many pages that the user hardly visit. BUT, It turns out, that it's NOT recommended to use async components for routing (you may use them to conditionally load and fetch component code when working with v-if etc. though). For routing, simply change the syntax for all routes from

```js
const CoachDetail = defineAsyncComponent(() =>
  import("./pages/coaches/CoachDetail.vue")
);
```

to

```js
const CoachDetail = () => import("./pages/coaches/CoachDetail.vue");
```

- After optimization,

```
npm run build
```

- It will create a `dist` folder which we can put onto a static hosting server.
- In server, we need to configure it to forward all paths to /index.html.

# The Composition API

- The `Options API` is exporting the configuration object (data, methods, computed, ...) from each component.
  - The problem is that the code that logically is related to each other can split across multiple options (data, methods, ...). But with this we can put them close each other (no more scrolling).
  - The second problem is re-using logic between components can be tricky or cumbersome.
- With the composition API, we merge **data, methods, computed, and watch** into `setup` method. But other options such as props, emits, components, ... will remain the same. Lifecycle methods will change too.

## ref and reactive

- `ref` function will return an object which its initial `value` will be what we pass and it will be reactive.
- Note that the `setup` method will run once at the startup of the component.
- What we return from `setup` will be available inside template.

```vue
<template>
  <h1>{{ name }}</h1>
</template>

<script>
import { ref } from "vue";

export default {
  setup() {
    const name = ref("Ben");

    setTimeout(() => {
      name.value = "John";
    }, 2000);

    return { name };
  },
  // data(){
  //   return {
  //     name:"Ben"
  //   }
  // },
  // created(){
  //   setTimeout(()=>{
  //     this.name = "John"
  //   }, 2000)
  // }
};
</script>
```

- Just like options API, you can mutate an object or set it to an entirely new value with `ref`.
- If the value that you want to pass is an object, it is better to use `reactive` instead of `ref`, then you don't need to `.value` and Vue will wrap it inside a proxy and you can use it easier. Note that `ref` will accept any type but `reactive` is only for objects.
- Just note that when returning, you should return the entire object in both `ref` ad `reactive` and not drill into its properties and return new variables from those properties to the template. The entire object is reactive and not its properties. If you insist, you can use `const objectRefs = toRefs(anObject);` and then all properties of that `objectRefs` are refs and therefore reactive to changes.

## Replacing Methods

- Methods in Options API can be replaced by regular functions in Composition API:

```vue
<template>
  <h1>{{ name }}</h1>
  <button @click="changeName">Change Name</button>
</template>

<script>
import { ref } from "vue";

export default {
  setup() {
    const name = ref("Ben");

    const changeName = () => {
      name.value = "John";
    };

    return { name, changeName };
  },
};
</script>
```

## Computed

- You can replace computed properties with `computed` function call.
- **Note** that under the hood, a computed variable is just a ref (it has .value property) but you cannot assign a new value to it (they are read-only).

```vue
<template>
  <h1>{{ fullName }}</h1>
  <button @click="changeName">Change Name</button>
</template>

<script>
import { ref, computed } from "vue";

export default {
  setup() {
    const firstName = ref("Ben");
    const lastName = ref("Abe");

    const fullName = computed(() => `${firstName.value} ${lastName.value}`);

    const changeName = () => {
      firstName.value = "John";
      lastName.value = "Doe";
    };

    return { fullName, changeName };
  },
};
</script>
```

## v-model

- `v-model` also accepts refs and reactives:

```vue
<template>
  <h1>{{ name }}</h1>
  <input type="text" v-model="name" />
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    const name = ref("");

    return { name };
  },
};
</script>
```

## watch

- To use `watch`ers:

```vue
<template>
  <h1>{{ name }}</h1>
  <input type="text" v-model="name" />
</template>

<script>
import { ref, watch } from "vue";
export default {
  setup() {
    const name = ref("");

    watch(name, (newValue, oldValue) => {
      console.log(newValue);
      console.log(oldValue);
    });

    return { name };
  },
};
</script>
```

- You can also pass an array of dependencies to watch -> more flexible than the options API.
- In this case, the callback function will also be executed with array of new values and array of old values.
- VERY IMPORTANT: If you want to watch a `prop`, you shouldn't use `watch(props.someProp, ()=>{...});` because props is reactive but its properties are not. So either you have to watch for `props` if you have just one prop, or use `toRefs` function to `const propsRefs = toRefs(props);` and then watch for `watch(propsRefs.someProp, ()=>{...});`.

## Use Template Refs

- We don't have access to `this` in setup method (setup method is called too early to have access) -> so no access to `this.$refs`. Instead (exactly like React!):

```vue
<template>
  <h1>{{ name }}</h1>
  <input type="text" ref="nameInput" />
  <button @click="setName">Set Name</button>
</template>

<script>
import { ref } from "vue";
export default {
  setup() {
    const name = ref("");
    const nameInput = ref(null);

    const setName = () => {
      name.value = nameInput.value.value;
    };

    return { name, nameInput, setName };
  },
};
</script>
```

## Components and Props

- Parent can use composition API but child can use options API -> nothing wrong with it.
- But how to use `props` in a component with composition API (since we don't have access to `this` and we know that props are outside `setup` method)? -> of course `setup(props, context)` receives props ;) which is an object
- context is also an object with three important properties: `attrs, emit, and slots`.
  - `attrs` is optional/fall-through props if the props haven't been provided.
  - If you have slots in your component, you can access them through slots.
  - If you want to emit something in setup method, you can do it through context.

## Provide and Inject

- We need to import `provide` and `inject` function from `vue`. In Parent:

```vue
<template>
  <h1>Parent</h1>
  <Child />
</template>

<script>
import { ref, provide } from "vue";
import Child from "./components/Child.vue";

export default {
  components: { Child },
  setup() {
    const name = ref("Ben");

    provide("nameKey", name);

    return {};
  },
};
</script>
```

- In Child:

```vue
<template>
  <section>
    <h2>Child</h2>
    <GrandChild />
  </section>
</template>

<script>
import GrandChild from "./GrandChild.vue";

export default {
  components: { GrandChild },
};
</script>
```

- In GrandChild:

```vue
<template>
  <h2>GrandChild</h2>
  <p>
    {{ name }}
  </p>
</template>

<script>
import { inject } from "vue";
export default {
  setup() {
    const name = inject("nameKey");

    return { name };
  },
};
</script>
```

- Here, we injected a ref so note that you should not change its value inside where it is injected.

## Lifecycle Methods

- So in summary:

![](/md/186.jpg)

- For lifecycle methods, we import corresponding functions from `vue`:

![](/md/187.jpg)

```vue
<template>
  <h1>{{ name }}</h1>
  <button @click="changeName">Change</button>
</template>

<script>
import {
  ref,
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from "vue";

export default {
  setup() {
    const name = ref("Ben");

    const changeName = () => {
      name.value = "John";
    };

    onBeforeMount(() => {
      console.log("onBeforeMount");
    });
    onMounted(() => {
      console.log("onMounted");
    });
    onBeforeUpdate(() => {
      console.log("onBeforeUpdate");
    });
    onUpdated(() => {
      console.log("onUpdated");
    });
    onBeforeUnmount(() => {
      console.log("onBeforeUnmount");
    });
    onUnmounted(() => {
      console.log("onUnmounted");
    });

    return { name, changeName };
  },
};
</script>
```

## Gotchas with Routing and Vuex

- Since we don't have access to `this.$route` and `this.$router`,
  - we can set the props to true in the route config object and receive it as props.
  - But if we wanted to access `query` or other things, we can use hooks inside `setup` method: `import { useRoute, useRouter } from "vue-router";` and then inside setup: `const route = useRoute();` or `const router = useRouter();`.
- Since we don't have access to `this.$store`,
  - we can use hooks inside `setup` method: `import { useStore } from "vuex";` and then inside setup: `const store = useStore();`. Then you can use, `store.dispatch` inside your function or `store.getters.something` inside your computed callback function.

![](/md/188.jpg)

# Code Reusability

## Options API -> Mixins

- Create a `mixins` folder next to `components` and create a `.js` file which exports default the shared configurations of the two components.
- Note that only `components` cannot be shared through mixins and should stay in their original components.
- Import the mixin in both components: `import alertMixin from "../mixins/alert.js";`.
- Add `mixins` property which is an array of mixins and add it there.
- When you have mixins, the options in your mixin will be `merged` with the options in your component.
- If we had the same data properties in mixin and inside the component, the component would overwrite the options provided by the mixin. But for lifecycle methods, both will run but the component's one will run after the mixin one.
- You can have `global mixins` (limited usage) to be connected to all your components, then you can import it in `main.js` and register it by `app.mixin(loggerMixin);`.
- **Disadvantage**:
  - We can't see from where the variables in our templates are coming from -> Use composition API.

## Compositions API -> Custom Composition Functions (Custom Hooks)

- Create a `hooks` folder next to `components`.
- Create an `alert.js` file which you export default a function called `useAlert`.
- Paste repeated code inside the custom hook and import necessary things (like `ref`, ... from `vue`).
- Then in your component: `import useAlert from "../hooks/alert.js";`.
- Then you can de-structure what it has returned from calling our custom hook.
- Note that you can define as many parameters as you want as input for your custom hook to customize it more.
