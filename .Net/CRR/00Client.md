# Client Skeleton

### Create React App

To make the app support Typescript, provide the following switch:

```Dos
npx create-react-app client --use-npm --typescript
```

And then:

```Dos
cd client
npm start
```

### Install axios

In the client folder install `axios`:

```Dos
npm i axios
```

### Install Semantic UI

In the client folder install `semantic-ui-react`:

```Dos
npm i semantic-ui-react
```

From [here](https://react.semantic-ui.com/usage) copy `Default theme (CDN)` and paste it in the head of `index.html`:

```html
<link
  rel="stylesheet"
  href="//cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.css"
/>
```

# Adding Redux and folders structure

### Adding Redux

In the client folder install `redux`, `redux-thunk`, `react-redux`, `redux-devtools-extension`:

```Dos
npm i redux redux-thunk react-redux redux-devtools-extension
```

```Dos
npm i @types/react-redux
```

### Folders structure

![](/md/client_folders.jpg)
