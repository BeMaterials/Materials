# Introduction

- The output of the `render` method is a react element which is a simple JS object in memory which maps to a DOM element (so react elements creates a virtual DOM in memory) which is cheap to build in comparison to DOM elements.
- When we change the `state` of a component, we get a new react element (with all its children) which will be compared to the previous one and if it is changed, it will update part of the real DOM. So react reacts to the state change.
- If `props` change in the parent, the child won't re-rendered. Child will be re-rendered if its props is coming from state in of its ancestors.
- Props doesn't need to be always state in the parent. They can be anything.
- If the state of the parent changes, it re-renders and all the its children that their props have been changed (otherwise they won't) will be re-rendered.
- For React and Redux, because the props in each child is the state in parent (the HOC connect), when a value changes in the store, the component will be re-rendered.
- Whatever inside the function component (including useEffect hooks), will be re-run during each render.
- `useEffect hook` is like `componentDidMount` + `componentDidUpdate`.
- In `useEffect`, usually the dependency is the `state` or `prop`:
  - `state`:
    - for example if `filter`, `currentPage`, or `pageSize` (states in the component itself) change, go and fetch the records again (which changes the `currentRecords` -> which is a prop from redux).
    - or when the `formData` (state in the component) changes, clear the `errors` (state in the component).
  - `props`:
    - for example if the `apiError` (prop from redux) changes, set the `errors` (state in the component) to that error.

## Set up

- With `npx create-react-app my-react-app`, create a new react app.
- With `npm start`, run the app.
- The div with the id of root in `index.html` will contain react.
- Each component returns JSX from `render` method. Babel will compile this into call to `React.createElement()`.
- in the `src` folder, create a `index.js` file:

```js
import React from 'react'; // there is no direct call to React here. But because JSX will be compiled to React.createElement, it is needed.
import ReactDOM from 'react-dom';

const element = <h1>Hi there!</h1>; // it is a JS object that you can console.log:
console.log(element); // {key: null, ref: null, props: {children: "hi there!"}, type: "h1", ...}

ReactDOM.render(
    element
  document.querySelector('#root')
);
```

# Components

- in the `src` folder, create a `index.js` file:

```js
import React from "react";
import ReactDOM from "react-dom";

import App from "./components/App";

ReactDOM.render(<App />, document.querySelector("#root"));
```

- In the `src/components` folder, create `App.js`:
- Whenever we want to write JS variables (of type string, number, array; but not object) or functions that return those types in JSX to show something to the user, use `{}`. In style, we pass an object, but note that we don't show the object to the user.
- Use `className` instead of `class`.
- Use `onClick` instead of `onclick`.
- Use `style={{...}}`. Also, instead of kebab-casing, use camelCasing for CSS properties. So the inner object should be a JS object.
- Use `htmlFor` instead of `for`.
- If you want to set `innerHTML` inside a JSX element (take a string and render as html):

```js
import React from "react";

const App = () => {
  return (
    <div
      dangerouslySetInnerHTML={{
        __html: `<img src="asdf" onerror="document.body.innerHTML = '<h1>HAHAHA, I control this app now!!!</h1>';"></img>`,
      }}
    />
  );
};

export default App;
```

- In general, setting HTML from code is risky because it’s easy to inadvertently expose your users to a `cross-site scripting` (XSS) attack. So, you can set HTML directly from React, but you have to type out dangerouslySetInnerHTML and pass an object with a \_\_html key, to remind yourself that it’s dangerous.
- `state` is local to a component`.
- `props` is readonly and you cannot change the props. They are input to the component.
- The component that owns an state should be the one modifying it -> `raise and handle event`.
- If you want to share a state as props to a sibling, you have to `lift the state up`.
- There are different methods that will be run during different phases of a component: `lifecycle hooks`.

![](/md/112.jpg)

- To make sure to pass the props `npm i prop-types`. Then:

```js
import PropTypes from "prop-types";

//...

MyComponent.propTypes = {
  someProp1: PropTypes.number.isRequired,
  someProp2: PropTypes.string,
  handleSomething: PropTypes.func.isRequired,
};

export default MyComponent;
```

- To style components, create a css file with the same name as the component: `MyComponent.css` and import it like `import "./MyComponent.css";` in the component.
- Add a class to the component root element: `className="my-component"`. And define `.my-component {...}` in the css file.
- Instead of `<React.Fragment>` and `</React.Fragment>`, we can use `<>` and `</>`.

```js
import React, { Component, Fragment } from "react";
import FunctionChild from "./FunctionChild";
import ClassChild from "./ClassChild";

class App extends Component {
  state = {
    name: "Ben",
    age: 35,
    persons: [
      { id: 1, name: "Tom" },
      { id: 2, name: "Jane" },
      { id: 3, name: "Jim" },
    ],
    showChildren: true,
  };

  constructor(props) {
    super(props);
    // if you want to initialize state here in the constructor:
    // this.state = {...}.

    console.log("App constructor");
  }

  componentDidMount() {
    // Ajax calls and then this.setState({...})
    console.log("App componentDidMount");
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("App componentDidUpdate", prevProps, prevState);
  }

  componentWillUnmount() {
    console.log("App componentWillUnmount");
  }

  handleClick = (e) => {
    // if "handleClick (e) {...", we wouldn't have access to "this".
    // so we should either bind this in the constructor or use an arrow function like here,
    // or use an arrow function when calling in JSX.
    console.log(e.target);

    const value = this.state.name === "John" ? "Ben" : "John";
    this.setState({ name: value }, () => {
      console.log("state has been set", this.state);
    });
  };

  handleDelete = (user) => {
    const persons = this.state.persons.filter(
      (person) => person.id !== user.id
    );

    this.setState({ persons });
  };

  render() {
    console.log("App render");
    const { name, persons, showChildren } = this.state;

    return (
      <Fragment>
        <button
          onClick={this.handleClick}
          className={this.state.name === "John" ? "class1" : "class2"}
        >
          Toggle
        </button>

        <h1 style={{ fontWeight: "bold" }}>hi {name}</h1>

        {name === "John" && <p>The name is John!</p>}

        <ul>
          {persons.map((user) => (
            <li
              key={user.id}
              onClick={() => {
                this.handleDelete(user);
              }}
            >
              {user.name}
            </li>
          ))}
        </ul>

        {showChildren && (
          <FunctionChild
            age={this.state.age}
            handleClick={() => this.setState({ age: 40 })}
          >
            I am children
          </FunctionChild>
        )}

        {showChildren && <ClassChild message="hey" />}

        <button onClick={() => this.setState({ showChildren: false })}>
          remove children
        </button>
      </Fragment>
    );
  }
}

export default App;
```

- in the `src/components` folder, create `FunctionChild.js`:

```js
import React, { useEffect } from "react";

const FunctionChild = ({ age, children, handleClick }) => {
  useEffect(() => {
    console.log("FunctionChild componentDidMount");
  }, []);

  useEffect(() => {
    console.log("FunctionChild componentDidUpdate");
  }, [age]);

  useEffect(() => {
    return () => {
      console.log("FunctionChild componentWillUnmount");
    };
  }, []);

  useEffect(() => {
    console.log("Initial render or age was changed"); //// execute when age changed

    return () => {
      console.log("CLEANUP"); // // execute before age is changed -. this technique is perfect for clearTimeout(timeoutId); when we setTimeout to search the term (connect to the api) after 1 sec of not typing (obviously instead of age it would be term. So we throttled the API by de-bouncing technique.)
    };
  }, [age]); //when age changes, first the cleanup function from the last render will be ran ("CLEANUP") and then "Initial render or age was changed"

  useEffect(() => {
    console.log("FunctionChild render"); // it runs after the console.log below -> after every render
  });

  console.log("FunctionChild render"); // always runs -> with every render
  return (
    <div>
      <p>{age}</p>
      <p>{children.toUpperCase()}</p>
      <button onClick={handleClick}>Change state of parent</button>
    </div>
  );
};

export default FunctionChild;
```

- in the `src/components` folder, create `ClassChild.js`:

```js
import React, { Component } from "react";

export default class ClassChild extends Component {
  componentDidMount() {
    console.log("ClassChild componentDidMount");
  }

  componentDidUpdate(prevProps, prevState) {
    // we can compare it with current values, and if there is any change, we can make some Ajax calls.
    console.log("ClassChild componentDidUpdate", prevProps, prevState);
  }

  componentWillUnmount() {
    // This will be run just before unmounting phase (being removed from the DOM) -> best opportunity to do the cleanup to avoid memory leaks: timers, listeners ...
    console.log("ClassChild componentWillUnmount");
  }

  render() {
    console.log("ClassChild render");
    const { message } = this.props;

    return <div>{message}</div>;
  }
}
```

![](/md/113.jpg)

- When you use a piece of state in the useEffect hook, react warns that you should include it in the dependency array. Sometimes when you do, it introduces new bugs (it is recommended to do that to prevent some nasty bugs) but watch for bugs and if necessary introduce new pieces of states to combat those bugs (like `debouncedTerm` and in useEffect for `term` setTimeout to `setDebouncedTerm` with the value of term and a cleanup to clear the timeout whenever term changes before the specified duration and in the useEffect for `debouncedTerm` call the API).

```js
useEffect(() => {
  const timerId = setTimeout(() => {
    setDebouncedTerm(term);
  }, 500);

  return () => {
    clearTimeout(timerId);
  };
}, [term]);

useEffect(() => {
  const callAPI = async () => {
    const { data } = await axios.post("...");

    setResults(data);
  };

  callAPI();
}, [debouncedTerm]);
```

- When updating state of an array of objects, you should clone the array and also the object.

# Pagination, Filtering, and Sorting

- You can also use `react-infinite-scroller` for pagination.
- `App.js`:

```js
import React from "react";
import MovieList from "./MovieList";

const App = () => {
  return (
    <div>
      <MovieList />
    </div>
  );
};

export default App;
```

- `MovieList.js`:

```js
import React, { useState } from "react";

import GenericList from "./GenericList";
import { getMovies } from "./movieService";

const MovieList = () => {
  const [currentMovies, setCurrentMovies] = useState([]); // if using redux, this line as the one below is not necessary
  const [filteredMoviesCount, setFilteredMoviesCount] = useState(0);
  const [moviesSortColumn, setMoviesSortColumn] = useState({
    path: "title",
    order: "asc",
  });

  const columns = [
    {
      label: "Movie title",
      path: "title", // path is for sorting
      content: (movie) => <h3>{movie.title}</h3>,
    },
    { label: "Number in stock", path: "inStock" },
    {
      key: "log", // key is for unique key of the columns that shouldn't have path (not sortable columns)
      content: (movie) => (
        <button
          onClick={() => {
            console.log(movie);
          }}
        >
          Log
        </button>
      ),
    },
  ];

  const filterFields = [
    { label: "Movie title", path: "title" },
    { label: "Number in stock", path: "inStock" },
  ];

  return (
    <GenericList
      title="MOVIES"
      columns={columns}
      filterFields={filterFields}
      getRecords={getMovies}
      currentRecords={currentMovies}
      setRecords={setCurrentMovies}
      setFilteredCount={setFilteredMoviesCount}
      totalRecordsCount={filteredMoviesCount}
      sortColumn={moviesSortColumn}
      setSortColumn={setMoviesSortColumn}
      initialPageSize="3"
    />
  );
};

export default MovieList;
```

- `movieService.js`:

```js
import _ from "lodash";

let movies = [
  {
    title: "Die Hard",
    inStock: "3",
  },
  {
    title: "GodFather 1",
    inStock: "2",
  },
  {
    title: "GodFather 2",
    inStock: "4",
  },
  {
    title: "Shrek",
    inStock: "1",
  },
  {
    title: "Mr. Bean 1",
    inStock: "3",
  },
  {
    title: "Mr. Bean 2",
    inStock: "2",
  },
  {
    title: "Rambo",
    inStock: "3",
  },
  {
    title: "Rocky",
    inStock: "3",
  },
];

const getMovies = (skip, limit, filter, sortColumn) => {
  const filtered = movies.filter((movie) => {
    let flag = true;

    for (let key in filter) {
      if (filter[key] && movie[key] !== filter[key]) {
        flag = false;
      }
    }

    return flag;
  });

  const sorted = _.orderBy(filtered, [sortColumn.path], [sortColumn.order]);

  return {
    filteredCount: filtered.length,
    data: sorted.slice(skip, skip + limit),
  };
};

export { getMovies };
```

- `GenericList.js`:

```js
import React, { Fragment, useEffect, useState } from "react";

import Filter from "./Filter";
import Pagination from "./Pagination";
import TableHeader from "./TableHeader";
import TableBody from "./TableBody";

const GenericList = ({
  title,
  columns,
  filterFields,
  getRecords,
  setRecords,
  setFilteredCount,
  currentRecords,
  totalRecordsCount,
  sortColumn,
  setSortColumn,
  initialPageSize,
}) => {
  const [filter, setFilter] = useState({});
  const [currentPage, setCurrentPage] = useState(1);
  const [pageSize] = useState(parseInt(initialPageSize));

  useEffect(() => {
    const records = getRecords(
      (currentPage - 1) * pageSize,
      pageSize,
      filter,
      sortColumn
    );
    setRecords(records.data); // if using redux, this line as the one below is not necessary
    setFilteredCount(records.filteredCount);
  }, [
    getRecords,
    setFilteredCount,
    setRecords,
    currentPage,
    pageSize,
    filter,
    sortColumn,
  ]);

  const handlePageChange = (page) => {
    setCurrentPage(page);
  };

  const handleFilterChange = (filter) => {
    setCurrentPage(1);
    setFilter(filter);
  };

  return (
    <Fragment>
      <h3>
        {title} ({totalRecordsCount})
      </h3>
      <Filter
        filterFields={filterFields}
        handleFilterChange={handleFilterChange}
      />
      <div>
        <table>
          <TableHeader
            columns={columns}
            sortColumn={sortColumn}
            setSortColumn={setSortColumn}
          />
          <TableBody data={currentRecords} columns={columns} />
        </table>

        <Pagination
          itemsCount={totalRecordsCount}
          pageSize={pageSize}
          currentPage={currentPage}
          handlePageChange={handlePageChange}
        />
      </div>
    </Fragment>
  );
};

GenericList.defaultProps = {
  // if we do not pass that prop, it will use this (not related to prop-types package)
  title: "Default Title", // other way is to use something like: props.title || "Default Title" in the code.
};

export default GenericList;
```

- `Filter.js`:

```js
import React, { useState } from "react";

const Filter = ({ filterFields, handleFilterChange }) => {
  const initialFilterData = {};
  filterFields.forEach((field) => {
    initialFilterData[field.path] = "";
  });

  const [filterData, setFilterData] = useState(initialFilterData);

  const handleChange = (e) => {
    setFilterData({ ...filterData, [e.target.name]: e.target.value });
  };

  const handleFilterSubmit = (e) => {
    e.preventDefault();

    handleFilterChange(filterData);
  };

  return (
    <form onSubmit={handleFilterSubmit} noValidate="novalidate">
      {filterFields.map((field, index) => (
        <input
          name={field.path}
          placeholder={field.label}
          autoComplete="off"
          value={filterData[field.path]}
          onChange={handleChange}
          key={index}
        />
      ))}
      <button type="submit" value="Submit">
        Filter
      </button>
    </form>
  );
};

export default Filter;
```

- `Pagination.js`:

```js
import React from "react";
import _ from "lodash";

const Pagination = ({
  itemsCount,
  pageSize,
  currentPage,
  handlePageChange,
}) => {
  const pagesCount = Math.ceil(itemsCount / pageSize);
  if (pagesCount === 1) return null; // no need for pagination if there is only one page
  const pages = _.range(1, pagesCount + 1); // creates an array of numbers from start up to, but not including, end.

  return (
    <ul>
      {pages.map((page) => (
        <li
          key={page}
          style={{ fontWeight: page === currentPage ? "bold" : "normal" }}
        >
          <a href="/#" onClick={() => handlePageChange(page)}>
            {page}
          </a>
        </li>
      ))}
    </ul>
  );
};

export default Pagination;
```

- `TableHeader.js`:

```js
import React from "react";

const TableHeader = ({ columns, sortColumn, setSortColumn }) => {
  const raiseSort = (path) => {
    if (!path) return;

    const newSortColumn = { ...sortColumn };
    if (newSortColumn.path === path)
      newSortColumn.order = newSortColumn.order === "asc" ? "desc" : "asc";
    else {
      newSortColumn.path = path;
      newSortColumn.order = "asc";
    }
    setSortColumn(newSortColumn);
  };

  const renderSortIcon = (column) => {
    if (column.path !== sortColumn.path) return null;
    if (sortColumn.order === "asc") return "v";
    return "^";
  };

  return (
    <thead>
      {columns.length > 1 && (
        <tr>
          {columns.map((column, index) => (
            <th
              key={column.path || column.key}
              onClick={() => raiseSort(column.path)}
            >
              {column.label} {renderSortIcon(column)}
            </th>
          ))}
        </tr>
      )}
    </thead>
  );
};

export default TableHeader;
```

- `TableBody.js`:

```js
import React from "react";
import _ from "lodash";

const TableBody = ({ columns, data }) => {
  const renderCell = (item, column) => {
    if (column.content) {
      return column.content(item);
    }

    return _.get(item, column.path);
  };

  return (
    <tbody>
      {data.map((item, index) => (
        <tr key={index}>
          {columns.map((column, index) => (
            <td key={index}>{renderCell(item, column)}</td>
          ))}
        </tr>
      ))}
    </tbody>
  );
};

export default TableBody;
```

# Routing

- We can create our own `Route` and `Link` components:

```js
import { useEffect, useState } from "react";

const Route = ({ path, children }) => {
  const [currentPath, setCurrentPath] = useState(window.location.pathname);

  useEffect(() => {
    const onLocationChange = () => {
      setCurrentPath(window.location.pathname);
    };

    window.addEventListener("popstate", onLocationChange);

    return () => {
      window.removeEventListener("popstate", onLocationChange);
    };
  }, []);

  return currentPath === path ? children : null;
};

export default Route;
```

```js
import React from "react";

const Link = ({ className, href, children }) => {
  const onClick = (event) => {
    if (event.metaKey || event.ctrlKey) {
      //these are booleans for mac and windows (if ctrl is held down)
      return; // we want the browser do its normal process of opening a new tab
    }

    event.preventDefault(); // to prevent page reload
    window.history.pushState({}, "", href); // to change the URL

    const navEvent = new PopStateEvent("popstate"); // publish an event that URL has changed -> Route component will listen
    window.dispatchEvent(navEvent);
  };

  return (
    <a onClick={onClick} className={className} href={href}>
      {children}
    </a>
  );
};

export default Link;
```

- Then we can use it like:

```js
<Route path="/">
  <Accordion items={items} />
</Route>
```

```js
<Link href="/" className="item">
  Accordion
</Link>
```

- Install `npm i react-router-dom`.
- The `BrowserRouter` gives the `history` object as `props` to all its direct children (not nested ones).
- We can also create our own `history` object and pass it to `Router` (not the `BrowserRouter`) as you can see in the `Calling Backend Services` section (This is the recommended way). If you don't want to do it, you will have to pass history object to you action creators for programmatic navigation or pass a callback to them.
- You have access to `match`, `location`, and `history` object in all the components which is wrapped with `Route`.
- To get the params variables -> `match.params.id`. We could use a selection reducer instad of url parameter, but it is better. And also note that in the URL approach, the component need to be self-sufficent to make the Ajax request for itself.
- To get query strings -> `location.search`. You can pass it to `parse` method of `import queryString from 'query-string';`.
- `exact` attribute for the `Route` component checks if the current url is exactly the pattern. If we don't use it, multiple routes will be match to a given url.
- `Switch` component selects only first route that matches. So we should use a combination of exact and Switch.
- If you want to use `Switch` without `exact`, you should order from the most specific route to the most generic ones.
- With `Link` instead of `a`, we can have SPA and no additional requests for reloading html and js bundle will be made to the server. We just show and hide components based on the URL.
- The `Route` component should not be always in the App component. We can have `nested routing`.
- If we want to have access to `match`, `location`, or `history` in a component which is not rendered by Route component, we can `import { withRouter } from "react-router-dom";` and wrap the component with `withRouter(MyComponent);` at the end. Then, we have those objects as props. The new way is to `import { useHistory } from "react-router-dom";` and then `const history = useHistory();`.
- If you want a full reload, you can use `window.location = "/";`.
- `HashRouter` is easier for deployment because the server will only return the `index.html`. But if we want to use `BrowserRouter`, we should modify the server to send only the `index.html` for matched patterns and not give a 404. Some servers such as `github pages` won't allow that logic, so we should use `HashRouter`.
- To show a component always, we can put it outside the `BrowserRouter` or not inside a `Route`. If that element has some router components like `Link`, we should have it inside the `BrowserRouter` like a Navbar.

![](/md/126.jpg)

- `App.js`:

```js
import React, { useState } from "react";
import {
  BrowserRouter as Router,
  Route,
  Switch,
  Redirect,
} from "react-router-dom";

import Navbar from "./Navbar";
import Login from "./Login";
import Movies from "./Movies";
import Products from "./Products";
import Movie from "./Movie";
import NotFound from "./NotFound";
import Dashboard from "./Dashboard";
import ProtectedRoute from "./ProtectedRoute";

const App = () => {
  const [isLoggedIn] = useState(true);

  return (
    <Router>
      <div>
        <Navbar />
        <Switch>
          <Route path="/login" exact component={Login} />
          <Route
            path="/movies/:id" // By :id? we will make it optional and for movies route, this routes will be matched
            // so it not recommended. Instead use query strings and read it like `location.search` in the React app.
            // we can have multiple route params: /:id/:other -> then we have match.params.other as well.
            exact
            component={Movie}
          />
          <Route path="/movies" exact component={Movies} />
          <Route
            path="/products"
            exact
            render={(props) => <Products sortBy="newest" {...props} />}
            // whenever we want to pass additional props (like here where we passed sortBy="", we use render instead of component)
            // if you don't pass ...props -> then you don't have access to history, location, and match
          />
          <Route
            path="/films"
            exact
            render={() => <Redirect to="/movies"></Redirect>}
          />
          <Route path="/not-found" exact component={NotFound} />
          <ProtectedRoute
            isLoggedIn={isLoggedIn}
            path="/admin" //because of nested routing, we don't use exact
            component={Dashboard}
          />
          <Redirect to="/not-found" />
        </Switch>
      </div>
    </Router>
  );
};

export default App;
```

- `Navbar.js`:

```js
import React from "react";
import { Link } from "react-router-dom";

const Navbar = () => {
  return (
    <div>
      <Link to="/movies">Movies</Link>
      <br />
      <Link to="/products">Products</Link>
      <br />
      <Link to="/admin">Dashboard</Link>
      <br />
      <Link to="/login">Login</Link>
      <hr />
    </div>
  );
};

export default Navbar;
```

- `Login.js`:

```js
import React from "react";

const Login = () => {
  return <div>Login</div>;
};

export default Login;
```

- `Movies.js`:

```js
import React from "react";
import { Link } from "react-router-dom";

const Movies = () => {
  return (
    <div>
      Movies
      <Link to="/movies/1?hi=bye">Movie 1</Link>
      <Link to="/movies/2?hi=hi">Movie 2</Link>
    </div>
  );
};

export default Movies;
```

- `Movie.js`:

```js
import React from "react";

const Movie = ({ match, history, location }) => {
  return (
    <div>
      Movie {match.params.id} queries {location.search}
      <br />
      <button onClick={() => history.push("/movies")}>
        Go to Movies by Programmatic Navigation (with push so back is working)
      </button>
      <button onClick={() => history.replace("/movies")}>Go to Movies by Programmatic Navigation (with replace so back is not working)</button>
    </div>
  );
};

export default Movie;
```

- `Products.js`:

```js
import React from "react";

const Products = ({ sortBy, history, location, match }) => {
  console.log(history, location, match);

  return <div>Products {sortBy}</div>;
};

export default Products;
```

- `Dashboards.js`:

```js
import React from "react";
import { Route, Link } from "react-router-dom";
import Movies from "./Movies";
import Products from "./Products";

const Dashboard = () => {
  return (
    <div>
      Admin Dashboard
      <Link to="/admin/movies">Movies</Link>
      <Link to="/admin/products">Products</Link>
      <Route path="/admin/movies" component={Movies} />
      <Route path="/admin/products" component={Products} />
    </div>
  );
};

export default Dashboard;
```

- `NotFound.js`:

```js
import React from "react";

const NotFound = () => {
  return <div>NotFound</div>;
};

export default NotFound;
```

- `ProtectedRoute.js`:

```js
import React from "react";
import { Route, Redirect } from "react-router-dom";

const ProtectedRoute = ({
  component: Component,
  render,
  isLoggedIn,
  ...rest
}) => {
  return (
    <Route
      {...rest}
      render={(props) => {
        if (!isLoggedIn)
          return (
            <Redirect
              to={{
                pathname: "/login",
                state: {
                  from: props.location, // the location of the user before login page (then in the login action creator, we can take a look at location.state.from.pathname to figure out where were we before redirect)
                },
              }}
            />
          );
        return Component ? <Component {...props} /> : render(props);
      }}
    />
  );
};

export default ProtectedRoute;
```

# Ref

- Use `ref` when you want to access to DOM elements directly (use rarely when you know what you are doing) -> Focus on an input field, animations or third-party DOM libraries. Or when you want to know the height of images to tile them in a gird to know the numbers of rows they need to span.
- `useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument (initialValue). The returned object will persist for the full lifetime of the component.
- Essentially, `useRef` is like a “box” that can hold a mutable value in its `.current` property.

```js
import React, { useRef } from "react";

const TextInputWithFocusButton = () => {
  const inputEl = useRef(null);

  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
};
```

- In case of images tiling:

```js
import React, { useRef, useState, useEffect } from "react";

const ImageCard = ({ image }) => {
  const imageRef = useRef(null);

  const [spans, setSpans] = useState(0);

  useEffect(() => {
    imageRef.current.addEventListener("load", () => {
      // because it takes time for the image to load
      const height = imageRef.current.clientHeight;

      setSpans(Math.ceil(height / 10)); // 10 px is the height of each row in our grid (grid-auto-rows)
    });
  }, []);

  return (
    <div style={{ girdRowEnd: `span ${spans}` }}>
      <img ref={imageRef} alt={image.description} src={image.urls.regular} />
    </div>
  );
};

export default ImageCard;
```

- `ImageList.js`:

```js
import React from "react";

const ImageList = ({ images }) => {
  return (
    <div>
      {images.map((image) => {
        return <ImageCard key={image.id} image={image} />;
      })}
    </div>
  );
};

export default ImageList;
```

- `ImageList.css`:

```css
.image-list {
  display: grid;
  gird-template-columns: repeat(
    auto-fill,
    minmax(250px, 1fr)
  ); /* auto-fill means grid will determine the number of columns and minmax() means each column should be greater than and less than certain values. 1fr makes it equally-sized */
  grid-gap: 0 10px;
  grid-auto-rows: 10px;
}

.image-list img {
  width: 250px;
}
```

- Be aware that in React, all the event handlers in the html dom elements will run before the components' event handlers. So if we want to have a dropdown in such a way that when we click on elsewhere it should be closed, we have to add an even listener to the body element and check that the click event target is not inside the component (by using ref):

```js
const [open, setOpen] = useState(false);
const ref = useRef();

useEffect(() => {
  const onBodyClick = (e) => {
    if (!ref.current.contains(e.target)) {
      setOpen(false);
    }
  };

  document.body.addEventListener("click", onBodyClick);

  return () => {
    // if we don't un-register, when the component is not showing, by clicking on body, ref is null and an error will be thrown.
    document.body.removeEventListener("click", onBodyClick);
  };
}, []);
```

- The `ref` should be on the top-level element of this component`ref={ref}`.

# Forms

- `App.js`:

```js
import React, { useState } from "react";
import { BrowserRouter as Router, Route, Switch } from "react-router-dom";

import Login from "./Login";
import Dashboard from "./Dashboard";
import ProtectedRoute from "./ProtectedRoute";

const App = () => {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [error, setError] = useState({});

  return (
    <Router>
      <div>
        <Switch>
          <Route
            path="/login"
            exact
            render={(props) => (
              <Login
                isLoggedIn={isLoggedIn}
                setIsLoggedIn={setIsLoggedIn}
                serverError={error}
                setServerError={setError}
                {...props}
              />
            )}
          />
          <ProtectedRoute
            isLoggedIn={isLoggedIn}
            path="/"
            component={Dashboard}
          />
        </Switch>
      </div>
    </Router>
  );
};

export default App;
```

- `Login.js`:

```js
import React, { useState, useEffect } from "react";
import { Redirect } from "react-router-dom";
import Joi from "joi";

import Input from "./Input";
import { login } from "./authService";
import validate from "./validate";
import validateProperty from "./validateProperty";

const Login = ({ isLoggedIn, setIsLoggedIn, serverError, setServerError }) => {
  const [formData, setFormData] = useState({
    email: "",
    password: "",
  });
  const { email, password } = formData;
  const [errors, setErrors] = useState({});

  const schema = {
    email: Joi.string().email({
      minDomainSegments: 2,
      tlds: { allow: ["com", "net"] },
    }),
    password: Joi.string().min(3).pattern(new RegExp("^[a-zA-Z0-9]{3,30}$")),
  };

  useEffect(() => {
    setServerError({});
  }, [formData, setServerError]);

  const handleChange = ({ target: input }) => {
    if (errors[input.name]) {
      const clonedErrors = { ...errors };
      delete clonedErrors[input.name];
      setErrors(clonedErrors);
    }

    setFormData({ ...formData, [input.name]: input.value });
  };

  const handleBlur = ({ target: input }) => {
    const clonedErrors = { ...errors };
    const errorMessage = validateProperty(input, schema);
    if (errorMessage) clonedErrors[input.name] = [errorMessage];
    else delete clonedErrors[input.name];

    setErrors(clonedErrors);
  };

  const handleSubmit = (e) => {
    e.preventDefault();

    const errors = validate(formData, schema);

    if (errors) {
      setErrors(errors);
    } else {
      const isMatch = login(email, password);
      if (isMatch) {
        setIsLoggedIn(true);
      } else {
        setServerError({ msg: "Invalid credential." });
      }
    }
  };

  if (isLoggedIn) {
    return <Redirect to="/" />;
  }

  return (
    <div>
      <div>
        <h1>Login</h1>
        <form onSubmit={handleSubmit}>
          <Input
            label="Email"
            name="email"
            placeholder="your email address"
            autoComplete="username"
            value={email}
            errors={errors["email"]}
            onChange={handleChange}
            onBlur={handleBlur}
            valid={errors["email"] ? false : true}
          />
          <Input
            label="Password"
            type="password"
            name="password"
            placeholder="password"
            autoComplete="current-password"
            value={password}
            errors={errors["password"]}
            onChange={handleChange}
            onBlur={handleBlur}
            valid={errors["password"] ? false : true}
          />
          <p>{serverError.msg}</p>
          <button disabled={validate(formData, schema)}>LOGIN</button>
        </form>
      </div>
    </div>
  );
};

export default Login;
```

- `Input.js`:

```js
import React from "react";

const Input = ({ valid, label, errors, ...props }) => {
  return (
    <div>
      <label>
        {label}
        <input {...props} className={!valid ? "invalid" : ""} />
      </label>
      {errors &&
        errors.map((error, index) => {
          return <div key={index}>{error}</div>;
        })}
    </div>
  );
};

export default Input;
```

- `validate.js`:

```js
import Joi from "joi";

export default function (formData, schema) {
  const { error } = Joi.object(schema).validate(formData, {
    abortEarly: false,
  });

  if (!error) return null;

  const errors = {};
  for (let item of error.details) {
    if (errors[item.path[0]]) {
      errors[item.path[0]].push(item.message);
    } else {
      errors[item.path[0]] = [item.message];
    }
  }
  return errors;
}
```

- `validateProperty.js`:

```js
import Joi from "joi";

export default function ({ name, value }, schema) {
  const obj = { [name]: value };
  const subSchema = { [name]: schema[name] };

  const { error } = Joi.object(subSchema).validate(obj);

  return error ? error.details[0].message : null;
}
```

- `authService.js`:

```js
const login = (email, password) => {
  if (email === "test@test.com" && password === "1234") return true;
  return false;
};

export { login };
```

# Calling Backend Services

- With starting a node project and then installing `npm i json-server` and then adding this script `"start": "json-server -p 3001 -w db.json"` and then creating a `db.json` with a `{"myResource":[]}`. json-server creates a `RESTful API` for us by running `npm start`.
- We use axios interceptors to handle unexpected errors (and some expected but uniform behaviors) globally.
- We can use `sentry` to log the errors from client's frontend. To do that, `npm i raven-js`. Copy some configuration in the `index.js` and then `import Raven from 'raven-js';` and use `Raven.captureException(error);` instead of `console.log(error);`.
- It is better to extract a loggerService in a module -> `services/logService.js`:

```js
import Raven from "raven-js";

function init() {
  Raven.config("https://examplePublicKey@o0.ingest.sentry.io/0").install();
}

function log(error) {
  Raven.captureException(error);
}

export default {
  init,
  log,
};
```

- Then, use `logger.init();` in `index.js` and `logger.log(error);` elsewhere.
- `src/apis/agent.js`:

```js
import axios from "axios";
import { toast } from "react-toastify";

import { history } from "../index";

axios.defaults.baseURL = "https://jsonplaceholder.typicode.com"; // store it in an env variable: REACT_APP_API_URL and use it like process.env.REACT_APP_API_URL

axios.interceptors.request.use(
  (config) => {
    const token = window.localStorage.getItem("jwt");
    if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

axios.interceptors.response.use(undefined, (error) => {
  if (!error.response) {
    toast.error("Network error - make sure API is running!");
  } else {
    const { status } = error.response;
    if (status === 404) {
      toast.error("Not found!");
      history.push("/notfound");
    }

    if (status === 401) {
      window.localStorage.removeItem("jwt");
      toast.error("Your session has expired, please login again");
      history.push("/");
    }

    if (status === 500) {
      toast.error("Server error - check the terminal for more info!");
    }
  }

  return Promise.reject(error); // to throw it again to the action creator
});

const responseBody = (response) => response.data;

const http = {
  get: (url) => axios.get(url).then(responseBody),
  post: (url, body) => axios.post(url, body).then(responseBody),
  put: (url, body) => axios.put(url, body).then(responseBody),
  del: (url) => axios.delete(url).then(responseBody),
  postForm: (url, file) => {
    let formData = new FormData();
    formData.append("File", file);
    return axios
      .post(url, formData, {
        headers: { "Content-type": "multipart/form-data" },
      })
      .then(responseBody);
  },
};

const posts = {
  list: (params) => axios.get("/posts", { params: params }).then(responseBody),
  details: (id) => http.get(`/posts/${id}`),
  create: (post) => http.post("/posts", post),
  update: (post) => http.put(`/posts/${post.id}`, post),
  delete: (id) => http.del(`/posts/${id}`),
};

export default {
  posts,
};
```

- `index.js`:

```js
import React from "react";
import ReactDOM from "react-dom";
import { Router } from "react-router-dom";
import { createBrowserHistory } from "history";
import "react-toastify/dist/ReactToastify.min.css";

import App from "./components/App";

export const history = createBrowserHistory();

ReactDOM.render(
  <Router // We didn't use BrowserRouter
    history={history} // because we want to have access to history object in the api agent and action creators
  >
    <App />
  </Router>,
  document.querySelector("#root")
);
```

- `App.js`:

```js
import React from "react";
import { Route, Switch } from "react-router-dom";
import { ToastContainer } from "react-toastify";

import Post from "./Post";
import Posts from "./Posts";

const App = () => {
  localStorage.setItem("jwt", "myToken");

  return (
    <React.Fragment>
      <ToastContainer position="bottom-right" />
      <Switch>
        <Route path="/posts" exact component={Posts} />
        <Route path="/posts/:id" exact component={Post} />
      </Switch>
    </React.Fragment>
  );
};

export default App;
```

- `Posts.js`:

```js
import React, { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import { toast } from "react-toastify";

import api from "../apis/agent";

const Posts = () => {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    const exec = async () => {
      // we can't use async on the useEffect so we use this trick.
      const posts = await api.posts.list();

      setPosts(posts);
    };

    exec(); // we could have not defined the exec and immediately invoke it: (async()=>{await ...;})();
  }, []);

  const createPost = async (newPost) => {
    //In case of Redux: see the comments
    // dispatch(setSubmitting(true));
    try {
      const post = await api.posts.create(newPost);

      const newPosts = [post, ...posts]; //Pessimistic update
      setPosts(newPosts);
      // dispatch({
      //   type: NEW_POST,
      //   payload: post,
      // });
      // dispatch(setSubmitting(false));
      // history.push(`/posts/${post.id}`);
    } catch (ex) {
      // ex.response && console.log(ex.response.data);
      // dispatch(setSubmitting(false));
      // toast.error("Problem creating post");
    }
  };

  const deletePost = async (id) => {
    const originalPosts = posts;
    const newPosts = posts.filter((post) => post.id !== id);
    setPosts(newPosts); //Optimistic update

    try {
      await api.posts.delete(id);
      throw new Error("simulation of an error");
    } catch (ex) {
      toast.error("Something failed while deleting a post!");
      setPosts(originalPosts);
    }
  };

  const updatePost = async (post) => {
    post.title = "UPDATED";
    await api.posts.update(post);

    const newPosts = [...posts];
    const index = newPosts.indexOf(post);
    newPosts[index] = { ...post };
    setPosts(newPosts);
  };

  return (
    <div>
      <button
        onClick={() => createPost({ userId: 1, title: "hi", body: "there" })}
      >
        New Post
      </button>

      <ul>
        {posts.map((post) => (
          <div key={post.id}>
            <Link to={`/posts/${post.id}`}>{post.title}</Link>
            <button onClick={() => deletePost(post.id)}>Del</button>
            <button onClick={() => updatePost(post)}>Update</button>
          </div>
        ))}
      </ul>
    </div>
  );
};

export default Posts;
```

- `Post.js`:

```js
import React, { useEffect, useState } from "react";
import { Link } from "react-router-dom";

import api from "../apis/agent";

const Post = ({ match }) => {
  const [post, setPost] = useState({});

  useEffect(() => {
    const exec = async () => {
      const post = await api.posts.details(match.params.id);

      setPost(post);
    };

    exec();
  }, [match.params.id]);

  return (
    <div>
      {post.title}
      <Link to="/posts">Back</Link>
    </div>
  );
};

export default Post;
```

- There is another way (Stephen's way) to pre-configure an axios instance (give it some default baseURL, query params or headers) -> `apis/someAPI.js`:

```js
import axios from "axios";

export default axios.create({
  baseURL: "http://localhost:3001",
  headers: { "x-custom-header": "foobar" },
  params: {
    Key: "asdgfadfgsdfgh",
  },
});
```

- Note that in axios get (and post) method, you can pass an object like `{ params: { Key: "sdfsdf" }}` as the second (third for post) argument.

# Authentication

## JWT

- In real world, in `login` action creator in `user` actions file:

```js
export const login = (userData) => async (dispatch) => {
  dispatch(setAuthError(null)); // empties error in user piece of store

  try {
    const user = await agent.User.login(userData);

    dispatch(setUser(user)); // sets user in user piece of store

    dispatch(setToken(user.token)); // sets token in app piece of store

    dispatch(closeModal());

    history.push("/dashboard"); // we could use: `location.state ? location.state.from.pathname : '/dashboard'` instead of just '/dashboard' but to do that, we should make the location object accessible in this module.
  } catch (ex) {
    ex.response && dispatch(setAuthError(ex.response)); // sets error in user store
  }
};
```

- which the `setToken` action creator in `app` actions is:

```js
export const setToken = (token) => {
  if (token) {
    window.localStorage.setItem("jwt", token);
  } else {
    window.localStorage.removeItem("jwt");
  }

  return {
    type: TOKEN, // sets token in app piece of store
    payload: token,
  };
};
```

- In the `App.js`, we get the token from store; if there is one we get the user otherwise just set the app to loaded but without any user:

```js
useEffect(() => {
  token ? getUser() : setAppLoaded();
}, [getUser, setAppLoaded, token]);

if (!appLoaded) return <Loading content="Loading app..." />;
```

- `getUser` action creator in `user` actions is:

```js
export const getUser = () => async (dispatch) => {
  try {
    const user = await agent.User.current();

    dispatch(setUser(user));

    dispatch(setAppLoaded()); // sets appLoaded to true in app store
  } catch (ex) {
    ex.response && console.log(ex.response.data);
  }
};
```

- Be aware that the `appReducer` has this initial state:

```js
const initialState = {
  token: null,
  appLoaded: false,
};
```

- and the `userReducer` has this initial state:

```js
const initialState = {
  user: null,
  isLoggedIn: false,
  error: null,
  loading: false,
};
```

- For logging out, `logout` action creator in `user` actions is:

```js
export const logout = () => (dispatch) => {
  dispatch(setUser(null));

  dispatch(setToken(null));

  history.push("/");
};
```

- Make sure that in the login page, check that if the user is logged in, redirect it to the home page.

## Google OAuth

![](/md/127.jpg)

- For use-case number 2, we have to modify the scope in google console. But for authentication, we are dealing with use-case number 1.

![](/md/128.jpg)

- The flow:

![](/md/129.jpg)

![](/md/130.jpg)

![](/md/131.jpg)

![](/md/132.jpg)

- We don't need OAuth secret (It is used for setting OAuth on the server).

![](/md/133.jpg)

- Add this to the head section of `public/index.html` (there is no npm package for it):

```html
<script src="https://apis.google.com/js/api.js"></script>
```

- By including that script, we have access to `gapi` library on the window scope.

- The `GoogleAuth` component:

```js
import React, { useEffect } from "react";
import { connect } from "react-redux";
import { signIn, signOut } from "../actions";

let auth;

const GoogleAuth = ({ isSignedIn, signIn, signOut }) => {
  useEffect(() => {
    window.gapi.load("client:auth2", () => {
      //gapi is multi-purpose -> so first we have to load what other js code we want from google
      window.gapi.client
        .init({
          clientId:
            "468939135624-v3npphun428p5qcock6f335rrsjo2rbk.apps.googleusercontent.com",
          scope: "email",
        }) // this shows the pop-up and asks the client
        .then(() => {
          auth = window.gapi.auth2.getAuthInstance(); // this is the auth object from google that we can do auth stuff with
          onAuthChange(auth.isSignedIn.get()); // run onAuthChange for the first time
          auth.isSignedIn.listen(onAuthChange); // whenever isSignedIn changes, run onAuthChange (the callback)
          // we could have re-written the code above  to follow redux conventions better
          // so that the actual login and logout of the user will be in the action creator,
          // but in that case the logic of this component will be spread out. But I think it would have been easier.
        });
    });
  }, []);

  const onAuthChange = (isSignedIn) => {
    if (isSignedIn) {
      signIn(auth.currentUser.get().getId()); // we use user's google Id to associate resources with the user
    } else {
      signOut();
    }
  };

  return (
    <div>
      {isSignedIn === null ? null : isSignedIn ? (
        <button
          className="ui red google button"
          onClick={() => {
            auth.signOut();
          }}
        >
          <i className="google icon"></i>
          Sign Out
        </button>
      ) : (
        <button
          className="ui green google button"
          onClick={() => {
            auth.signIn();
          }}
        >
          <i className="google icon"></i>
          Sign In with Google
        </button>
      )}
    </div>
  );
};

const mapStateToProps = (state) => {
  return {
    isSignedIn: state.auth.isSignedIn,
    userId: state.auth.userId,
  };
};

export default connect(mapStateToProps, { signIn, signOut })(GoogleAuth);
```

- `autReducer` and its action creators:

```js
const INITIAL_STATE = {
  isSignedIn: null,
  userId: null,
};

export default (state = INITIAL_STATE, action) => {
  switch (action.type) {
    case SIGN_IN:
      return { ...state, isSignedIn: true, userId: action.payload };
    case SIGN_OUT:
      return { ...state, isSignedIn: false, userId: null };
    default:
      return state;
  }
};

export const signIn = (userId) => {
  return {
    type: SIGN_IN,
    payload: userId,
  };
};

export const signOut = () => {
  return {
    type: SIGN_OUT,
  };
};
```

- Now we can import `GoogleAuth` to each component to see a button.

# Modals and Portals

- Usually for the modals, we don't need portals. In this case, the `index.js` file will be like:

```js
import React from "react";
import ReactDOM from "react-dom";
import { Router } from "react-router-dom";
import { createBrowserHistory } from "history";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import { composeWithDevTools } from "redux-devtools-extension";
import thunk from "redux-thunk";
import App from "./components/App";
import reducers from "./reducers";

const store = createStore(
  reducers,
  composeWithDevTools(applyMiddleware(thunk))
);

export const history = createBrowserHistory();

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <App />
    </Router>
  </Provider>,
  document.getElementById("root")
);
```

- We set the `Modal` component as a direct child of the `root`:

```js
import React, { Fragment } from "react";
import { Route, Switch } from "react-router-dom";

import Modal from "./common/Modal";
import HomePage from "./HomePage";
import Navbar from "./Navbar";
import Posts from "./Posts";
import Post from "./Post";
import NotFound from "./NotFound";

const App = () => {
  return (
    <Fragment>
      <Modal />
      <Route exact path="/" component={HomePage} />
      <Route
        path={"/(.+)"} // match for anything
        render={() => (
          <Fragment>
            <Navbar />
            <Switch>
              <Route exact path="/posts" render={() => <Posts />} />
              <Route exact path="/posts/:id" component={Post} />
              <Route
                component={NotFound} // Obviously, there is path for this component! And because there is no path, we cannot use exact and we must use Switch.
              />
            </Switch>
          </Fragment>
        )}
      />
    </Fragment>
  );
};

export default App;
```

- `Modal.js`:

```js
import React from "react";
import { connect } from "react-redux";
import { closeModal } from "../../actions";
import "./Modal.css";

const Modal = ({ closeModal, open, body }) => {
  return (
    open && (
      <div className="modal" onClick={closeModal}>
        <div className="modal-content" onClick={(e) => e.stopPropagation()}>
          {body}
        </div>
      </div>
    )
  );
};

const mapStateToProps = ({ modal: { open, body } }) => ({ open, body });

export default connect(mapStateToProps, {
  closeModal,
})(Modal);
```

- `Modal.css`:

```css
.modal {
  height: 100%;
  width: 100%;
  position: fixed;
  background-color: rgba(100, 100, 100, 0.5);
  left: 0;
  top: 0;
}

.modal-content {
  border: 1px solid black;
  background-color: white;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
```

- The modal action creators:

```js
import { OPEN_MODAL, CLOSE_MODAL } from "./types";

// Open Modal
export const openModal = (content) => ({
  type: OPEN_MODAL,
  payload: content,
});

// Close Modal
export const closeModal = () => ({
  type: CLOSE_MODAL,
});
```

- The modal reducer:

```js
import { OPEN_MODAL, CLOSE_MODAL } from "../actions/types";

const initialState = {
  open: false,
  body: null,
};

export default function (state = initialState, action) {
  const { type, payload } = action;

  switch (type) {
    case OPEN_MODAL:
      return { ...state, open: true, body: payload };

    case CLOSE_MODAL:
      return { ...state, open: false, body: null };

    default:
      return state;
  }
}
```

- To define the content of a modal:

```js
import React from "react";
import { connect } from "react-redux";
import { closeModal } from "../actions";

const DeletePostModal = ({ closeModal }) => {
  return (
    <React.Fragment>
      <h1>WARNING</h1>
      <p>Are you sure?</p>
      <button onClick={() => closeModal()}>Cancel</button>
      <button onClick={() => console.log("Deleted")}>Delete</button>
    </React.Fragment>
  );
};

export default connect(null, { closeModal })(DeletePostModal);
```

- Then to actually open a modal:

```js
import React from "react";
import { connect } from "react-redux";
import DeletePostModal from "./DeletePostModal";
import { openModal } from "../actions";

const Post = ({ match, openModal }) => {
  return (
    <React.Fragment>
      Post {match.params.id}
      <button onClick={() => openModal(<DeletePostModal />)}>Delete</button>
    </React.Fragment>
  );
};

export default connect(null, { openModal })(Post);
```

## Portals

- Another way for creating modals is to return modals instead of configuring a one single modal with action creators. So don't forget to remove the `<Modal />` component from `App.js`.
- Note that there is no global modal redux state for modals in this way.
- To do that we should add `<div id="modal"></div>` to the `public/index.html` next to the div with the id of root.
- Then, we create the modal component (With portals we can create elements not as a direct child of their parent component):

```js
import React from "react";
import ReactDOM from "react-dom";
import "./Modal.css";

const Modal = ({ body, closeModal }) => {
  return ReactDOM.createPortal(
    <div className="modal" onClick={closeModal}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {body}
      </div>
    </div>,
    document.querySelector("#modal")
  );
};

export default Modal;
```

- The difference is to open a modal here we don't call an action creator and instead we return JSX:

```js
import React, { useState } from "react";
import DeletePostModal from "./DeletePostModal";
import Modal from "./common/Modal";

const Post = ({ match }) => {
  const [showModal, setShowModal] = useState(false);

  return (
    <React.Fragment>
      Post {match.params.id}
      <button onClick={() => setShowModal(true)}>Delete</button>
      {showModal && (
        <Modal
          body={<DeletePostModal setShowModal={setShowModal} />}
          closeModal={() => setShowModal(false)}
        />
      )}
    </React.Fragment>
  );
};

export default Post;
```

- In which `DeletePostModal` is:

```js
import React from "react";

const DeletePostModal = ({ setShowModal }) => {
  return (
    <React.Fragment>
      <h1>WARNING</h1>
      <p>Are you sure?</p>
      <button onClick={() => setShowModal(false)}>Cancel</button>
      <button onClick={() => console.log("Deleted")}>Delete</button>
    </React.Fragment>
  );
};

export default DeletePostModal;
```

# Environment Variables

- Applications created with CRA, have built-in support for environment variables.
- In the root, create `.env` file and paste the env variables prefixed with `REACT_APP_` for all environments.
- To give environment-specific variables, use `.env.development`, `.env.production`, and `.env.test`.
- Access to the values in the app using, `process.env.REACT_APP_API_URL`.
- `npm run build` to build an optimized production build (then it uses production env variable).

# Higher Order Components

![](/md/139.jpg)

- Be aware that with custom hooks, some HOC can be written as custom hooks.
- To reuse logic for different components -> For example giving tooltip when hovering or loader icon when fetching from backend we wrap the components with HOC.
- Usually, when you wrap your component with a HOC, you get will get access to some `injected props`.
- `App.js`:

```js
import React from "react";
import Movie from "./Movie";

const App = () => {
  return <Movie title="Shrek" />;
};

export default App;
```

- `Movie.js`:

```js
import React from "react";
import withTooltip from "./withTooltip";

const Movie = ({ showTooltip, title }) => {
  return (
    <div>
      {title} {showTooltip && <div>Some tooltip</div>}
    </div>
  );
};

export default withTooltip(Movie);
```

- `withTooltip.js`:

```js
import React, { useState } from "react";

export default function withTooltip(WrappedComponent) {
  return (props) => {
    //returns a new functional component
    const [showTooltip, setShowTooltip] = useState(false);

    return (
      <div
        onMouseOver={() => {
          setShowTooltip(true);
        }}
        onMouseOut={() => {
          setShowTooltip(false);
        }}
      >
        <WrappedComponent
          showTooltip={showTooltip} // injected props (they are usually state or method)
          {...props} // don't forget to pass through the props
        />
      </div>
    );
  };
}
```

- Another interesting example of a HOC is:

```js
import React, { useEffect } from "react";
import { connect } from "react-redux";

export default function requireAuth(WrappedComponent) {
  function mapStateToProps(state) {
    return { auth: state.auth };
  }

  return connect(mapStateToProps)(({ auth, ...passThroughProps }) => {
    // Filter out extra props that are specific to this HOC
    useEffect(() => {
      if (!auth) {
        passThroughProps.history.push("/");
      }
    }, [auth]);

    return <WrappedComponent {...passThroughProps} />;
  });
}
```

# Custom Hooks

- Building your own Hooks lets you extract component logic into reusable functions.
- When we want to share logic between two JavaScript functions, we extract it to a third function. Both components and Hooks are functions, so this works for them too!
- Custom hooks are to reuse everything on top of JSX.

![](/md/114.jpg)

![](/md/115.jpg)

![](/md/116.jpg)

```js
import { useState, useEffect } from "react";

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline; // you can return an array like primitive hooks or an object.
}

export default useFriendStatus;
```

- Now use it here:

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? "green" : "black" }}>{props.friend.name}</li>
  );
}
```

- and there:

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}
```

# Context

- `context` is introduced to combat with `prop drilling`. With context, we can use some kind of central store concept like `redux`. In the parent (provider), we create a context for shared data and in grand children, we consume that context.
- Steps:

  1. Create the context in a separate file like `contexts/UserContext.js`:

  ```js
  import React from "react";

  export default React.createContext(); // if we pass something, it will be the default value of the context
  ```

  2. Import context in the parent component and wrap the JSX with `UserContext.Provider` component and set `value` prop to what we want to pass (what we usually pass is state or prop):

  ```js
  import React, { useState } from "react";
  import Child from "./Child";
  import UserContext from "../contexts/UserContext";

  const App = () => {
    const [currentUser, setCurrentUser] = useState({ name: "Ben" });

    return (
      <UserContext.Provider // we don't need to wrap the whole parent component with the provider, just we compass the target components
        value={{ currentUser, setCurrentUser }} // very important: we can have two providers of the same context but since they are different instances, they can have different values at the same type
        // so separate pipe of information
      >
        <Child />
      </UserContext.Provider>
    );
  };

  export default App;
  ```

  3. Import context in the grand-child component and add `const userContext = useContext(UserContext);`. Now use it

  ```js
  import React, { useContext } from "react";
  import UserContext from "../contexts/UserContext";

  const GrandChild = () => {
    const userContext = useContext(UserContext);

    return (
      <div>
        GrandChild {userContext.currentUser.name}
        <button onClick={() => userContext.setCurrentUser({ name: "John" })}>
          Change User
        </button>
      </div>
    );
  };

  export default GrandChild;
  ```

  ```js
  import React from "react";
  import GrandChild from "./GrandChild";

  const Child = () => {
    return (
      <div>
        Child
        <GrandChild />
      </div>
    );
  };

  export default Child;
  ```

- So as you can see to update the state, we should pass the handle as well.
- If you have multiple contexts, you can nest them in the parent component (and pass a `value` prop to each) and use multiple `useContext` in the grand-child component.

# Redux

![](/md/143.jpg)

![](/md/117.jpg)

![](/md/118.jpg)

- Action creator (a function) creates an `action` with `type` and `payload`.
- The `action` will be `dispatched` by the `dispatch` function to all the `reducers` along with their slice of data from the `store` -> `store.dispatch(action);`.
- The `reducer` will look at the type and if it cares, it will update its slice of data from the store and return it.

## Action creator

- With `redux-thunk middleware` , action creators can return a plain JS object with `type` and `payload`, `=> ({type:..., payload:...});`, or a function which is called with dispatch and getState functions `=> async (dispatch, getState) => {...; dispatch({type:..., payload:...});}`.
- The `dispatch` function will send that plain JS object or function to the first middleware (redux-thunk).
- If it is a function, `redux-thunk` will call the function (which usually contains a call to dispatch so we will go back again to dispatch). If it is an object just give it to next middleware or to all the reducers. This technique is useful for async stuff.
- With `redux-thunk middleware`, you can always (even when not async) return a function `=> dispatch => {dispatch({type:..., payload:...});}`. This technique is useful to dispatch multiple actions or even another action creator: `dispatch(setLoadingInitial(true));` in an action creator.

![](/md/119.jpg)

![](/md/140.jpg)

![](/md/120.jpg)

![](/md/121.jpg)

![](/md/122.jpg)

- Create `types.js` in the `actions` folder to host all the action types:
- For naming actions, name it imperatively: `ADD_POST`.

```js
// Action Types
export const TOKEN = 'TOKEN';
export const APP_LOADED_STATUS = 'APP_LOADED_STATUS';
...
```

- `React-Redux` `connect` function performs a `shallow equality check` (That means that the top-level values of two objects are compared to see if each pair of values is === equal, rather than comparing if the objects themselves are the same) `on each value within the props object` (the return from mapStateToProps), not on the props object itself. So if a piece of state (A property in a reducer not the reducer itself; because the reducer will be a property in the total store object) is an object with a property of an array and we get that piece of state in the action creator by getState (we need three levels -> getState().post.post), we shouldn't use push to change its array property.
- `connect` HOC communicates with the `Provider` component through use of react `context`.
- For each category of data, create an action creator file to host all action creators in the `actions` folder, like `app.js`:

```js
import { TOKEN, APP_LOADED_STATUS } from "./types"; // If there are lots of actions: import * as actions from "./types";

// Set Token
export const setToken = (token) => {
  if (token) {
    window.localStorage.setItem("jwt", token);
  } else {
    window.localStorage.removeItem("jwt");
  }

  return {
    type: TOKEN,
    payload: token,
  };
};

// Set App Loaded
export const setAppLoaded = () => ({
  type: APP_LOADED_STATUS,
});
```

- `user.js`:

```js
import { USER, AUTH_ERROR } from "./types";
import { setToken, setAppLoaded } from "./app";
import agent from "../apis/agent";
import { history } from "./../index";

// Register
export const register = (userData) => async (dispatch) => {
  dispatch(setAuthError(null));

  try {
    const user = await agent.User.register(userData);

    dispatch(setUser(user));

    dispatch(setToken(user.token));

    history.push("/posts");
  } catch (ex) {
    ex.response && dispatch(setAuthError(ex.response));
  }
};

// Get User
export const getUser = () => async (dispatch) => {
  try {
    const user = await agent.User.current();

    dispatch(setUser(user));

    dispatch(setAppLoaded());
  } catch (ex) {
    ex.response && console.log(ex.response.data);
  }
};

// Login
export const login = (userData) => async (dispatch) => {
  dispatch(setAuthError(null));

  try {
    const user = await agent.User.login(userData);

    dispatch(setUser(user));

    dispatch(setToken(user.token));

    history.push("/posts");
  } catch (ex) {
    ex.response && dispatch(setAuthError(ex.response));
  }
};

// Logout
export const logout = () => (dispatch) => {
  dispatch(setUser(null));

  dispatch(setToken(null));

  history.push("/");
};

// Set User
export const setUser = (user) => ({
  type: USER,
  payload: user,
});

// Set Auth Error
export const setAuthError = (err) => ({
  type: AUTH_ERROR,
  payload: err,
});
```

- `modal.js`:

```js
import { OPEN_MODAL, CLOSE_MODAL } from "./types";

// Open Modal
export const openModal = (content) => ({
  type: OPEN_MODAL,
  payload: content,
});

// Close Modal
export const closeModal = () => ({
  type: CLOSE_MODAL,
});
```

- `post.js`:

```js
import { toast } from "react-toastify";

import agent from "../apis/agent";
import { history } from "./../index";
import {
  POSTS_LIST,
  CURRENT_POST,
  NEW_POST,
  EDIT_POST,
  DELETE_POST,
  POST_PAGE,
  POST_LOADING_STATUS,
  POSTS_LOADING_STATUS,
  POST_SUBMITTING_STATUS,
} from "./types";
import { getAxiosParams } from "./../utils/getAxiosParams";
import { getTotalPage } from "./../utils/getTotalPage";

// Load Posts
export const loadPosts = () => async (dispatch, getState) => {
  // dispatch and getState is run on store actually: store.dispatch(action); and store.getState();.

  // To compare with cache
  // const { lastFetch } = getState().post;
  // const diffInMin = moment().diff(moment(lastFetch), "minutes");
  // if (diffInMin < 10) return;

  dispatch(setLoadingInitial(true));

  try {
    const params = getAxiosParams(getState().post.page, getState().post.filter); // getState gives the state of all the store so we need to go to post and then filter
    const { items: posts, totalItems: postCount } = await agent.posts.list(
      params
    );

    dispatch({
      type: POSTS_LIST,
      payload: {
        posts,
        postCount,
        totalPages: getTotalPage(postCount),
      },
    });
    dispatch(setLoadingInitial(false));
  } catch (ex) {
    ex.response && console.log(ex.response.data);
    dispatch(setLoadingInitial(false));
    toast.error("Problem loading posts");
  }
};

// Load Post
export const loadPost = (id) => async (dispatch, getState) => {
  const post = getState().post.posts.find((post) => post.id === id);
  if (post) {
    dispatch({
      type: CURRENT_POST,
      payload: post,
    });
  } else {
    dispatch(setLoadingInitial(true));
    try {
      const post = await agent.posts.details(id);

      dispatch({
        type: CURRENT_POST,
        payload: post,
      });
      dispatch(setLoadingInitial(false));
    } catch (ex) {
      ex.response && console.log(ex.response.data);
      dispatch(setLoadingInitial(false));
      toast.error("Error!");
    }
  }
};

// Create Post
export const createPost = (newPost) => async (dispatch) => {
  // we could use getState here and associate the post author from the user piece of store
  dispatch(setSubmitting(true));

  try {
    await agent.posts.create(newPost);

    dispatch({
      type: NEW_POST,
      payload: newPost,
    });
    dispatch(setSubmitting(false));

    history.push(`/posts/${newPost.id}`);
  } catch (ex) {
    ex.response && console.log(ex.response.data);
    dispatch(setSubmitting(false));
    toast.error("Error!");
  }
};

// Edit Post
export const editPost = (id, updatedPost) => async (dispatch) => {
  dispatch(setSubmitting(true));

  try {
    await agent.posts.update(updatedPost);

    dispatch({
      type: EDIT_POST,
      payload: { id, updatedPost },
    });
    dispatch(setSubmitting(false));

    history.push(`/posts/${updatedPost.id}`);
  } catch (ex) {
    ex.response && console.log(ex.response.data);
    dispatch(setSubmitting(false));
    toast.error("Error!");
  }
};

// Delete Post
export const deletePost = (id) => async (dispatch) => {
  dispatch(setSubmitting(true));

  try {
    await agent.Posts.delete(id);

    dispatch({
      type: DELETE_POST,
      payload: id,
    });
    dispatch(setSubmitting(false));
  } catch (ex) {
    ex.response && console.log(ex.response.data);
    dispatch(setSubmitting(false));
    toast.error("Error!");
  }
};

// Set Page
export const setPage = (page) => ({
  type: POST_PAGE,
  payload: page,
});

// Set Loading
export const setLoading = (loading) => ({
  type: POST_LOADING_STATUS,
  payload: loading,
});

// Set Initial Loading
export const setLoadingInitial = (loadingInitial) => ({
  type: POSTS_LOADING_STATUS,
  payload: loadingInitial,
});

// Set Submitting
export const setSubmitting = (submitting) => ({
  type: POST_SUBMITTING_STATUS,
  payload: submitting,
});
```

- Then, in an `index.js` file in the `actions` folder:

```js
export * from "./app";
export * from "./user";
export * from "./modal";
export * from "./post";
```

## Reducer

![](/md/123.jpg)

- Reducers will be called once during the initialization phase to initialize their state:

![](/md/124.jpg)

![](/md/125.jpg)

- Reducers should not mutate the state and must return a new state.
- The fourth rule is not for redux. It is for react-redux and notifying react to re-render.
- Apart from the whole state of a reducer that should not be mutated, If a piece of state (a property) is an array, we should return a new array by destructuring or filter. If it is an object, we should return an object with its top-level values be different from the previous state (because of react-redux shallow-compare). If you return just a new object with the same top-level values it won't cause re-rendering (shallow-compare).
- Note that usually, the state of each reducer is an object. But it can be an array or string (like Stephen Grider's).

- For each category of data, create a reducer in the `reducers` folder, like `appReducer.js`:

```js
import { APP_LOADED_STATUS, TOKEN } from "../actions/types";

const initialState = {
  token: null,
  appLoaded: false,
};

export default function (state = initialState, action) {
  const { type, payload } = action;

  switch (type) {
    case APP_LOADED_STATUS:
      return { ...state, appLoaded: true };

    case TOKEN:
      return { ...state, token: action.payload };

    default:
      return state;
  }
}
```

- `userReducer.js`:

```js
import { USER, AUTH_ERROR } from "../actions/types";

const initialState = {
  user: null,
  isLoggedIn: false,
  error: null,
  loading: false,
};

export default function (state = initialState, action) {
  const { type, payload } = action;

  switch (type) {
    case USER:
      return {
        ...state,
        user: payload,
        isLoggedIn: !!payload,
        error: null,
      };

    case AUTH_ERROR:
      return {
        ...state,
        error: payload,
      };

    default:
      return state;
  }
}
```

- `modalReducer.js`:

```js
import { OPEN_MODAL, CLOSE_MODAL } from "../actions/types";

const initialState = {
  open: false,
  body: null,
};

export default function (state = initialState, action) {
  const { type, payload } = action;

  switch (type) {
    case OPEN_MODAL:
      return { ...state, open: true, body: payload };

    case CLOSE_MODAL:
      return { ...state, open: false, body: null };

    default:
      return state;
  }
}
```

- `postReducer.js`:

```js
import {
  POSTS_LIST,
  CURRENT_POST,
  NEW_POST,
  EDIT_POST,
  DELETE_POST,
  POST_FILTER,
  POST_PAGE,
  POST_LOADING_STATUS,
  POSTS_LOADING_STATUS,
  POST_SUBMITTING_STATUS,
} from "../actions/types";

const initialState = {
  posts: [], // instead of posts being an array we can have it as an object { [id]: {id, title}, ... }. Then, the reducer's manipulations would have been much easier (and we could use key interpolation) -> better performance too!
  // return {...state, posts: {...state.posts, [action.payload.id]: action.payload}}
  post: null,
  loadingInitial: false,
  loading: false,
  // lastFetch: null, // for caching
  submitting: false,
  postCount: 0,
  page: 0,
  totalPages: 0,
  filter: { all: true, publishDate: new Date() },
};

export default function (state = initialState, action) {
  const { type, payload } = action;

  switch (type) {
    case POSTS_LIST:
      return {
        ...state,
        posts: [...state.posts, ...payload.posts], // if posts were object {...state.posts, ..._.mapKeys(payload.posts, 'id') }; because what we got from the API is still array and we want to turn it to object.
        // In the component in this case, in the mapStateToProps we should use posts: Object.values(state.post.posts).
        postCount: payload.postCount,
        totalPages: payload.totalPages,
        // lastFetch: Date.now(), // for caching
      };

    case CURRENT_POST:
      return {
        ...state,
        post: payload,
      };

    case NEW_POST:
      return {
        ...state,
        posts: [...state.posts, payload],
        post: payload,
      };

    case EDIT_POST:
      return {
        ...state,
        posts: [
          ...state.posts.filter((post) => post.id !== payload.id),
          payload.updatedPost,
        ],
        post: payload.updatedPost,
      };

    case DELETE_POST:
      return {
        ...state,
        posts: state.posts.filter((post) => post.id !== payload), // if the posts were an object: { ...state.posts, [payload]: undefined } or we could use _.omit(state.posts, payload)
        post: null,
      };

    case POST_FILTER:
      return {
        ...state,
        filter: payload,
      };

    case POST_PAGE:
      return {
        ...state,
        page: payload,
      };

    case POST_LOADING_STATUS:
      return { ...state, loading: payload };

    case POSTS_LOADING_STATUS:
      return { ...state, loadingInitial: payload };

    case POST_SUBMITTING_STATUS:
      return { ...state, submitting: payload };

    default:
      return state;
  }
}
```

- Then, in the `index.js` in the `reducers` folder, combine all the reducers:

```js
import { combineReducers } from "redux";
import appReducer from "./appReducer";
import userReducer from "./userReducer";
import modalReducer from "./modalReducer";
import postReducer from "./postReducer";

export default combineReducers({
  app: appReducer, // keys here will be keys in the state object
  user: userReducer,
  modal: modalReducer,
  post: postReducer,
});
```

## Connect to React

- The main `index.js` should be something like:

```js
import React from "react";
import ReactDOM from "react-dom";

import { Router } from "react-router-dom";
import { createBrowserHistory } from "history";

import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";
import { composeWithDevTools } from "redux-devtools-extension"; // to see redux store in the browser (You should install Redux DevTools extension on your browser)
import { Provider } from "react-redux";
import reducers from "./reducers";

import App from "./components/App";

const store = createStore(
  reducers,
  { app: { token: window.localStorage.getItem("jwt"), appLoaded: false } },
  composeWithDevTools(applyMiddleware(thunk)) // with redux DevTools, you can "jump" around each action and restore the store at that time
);

export const history = createBrowserHistory();

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <App />
    </Router>
  </Provider>,
  document.getElementById("root")
);
```

- In a react component:

```js
import { connect } from "react-redux";
import { actionCreator } from "../actions"; // import action creators -> but you can't use it right away. You have to pass it to connect function to become available to the component as props

const MyComponent = ({ actionCreator, submitError, user }) => {
  // pass action creators and states from the redux store as props
  return (
    // some JSX
    <div></div>
  );
};

const mapStateToProps = (state, ownProps) => {
  return {
    submitError: state.user.error,
    user: state.users.find((user) => user.id === ownProps.userId), //ownProps is useful for some pre-calculation and extract the props that we care about. It is not good to give the component the list of whole users.
  };
};

export default connect(mapStateToProps, {
  actionCreator, // pass the action creators to actually go through dispatch function
})(MyComponent); // call to connect returns a function so the second () is invoking that function
```

### Hooks

- You can `import {useDispatch, useSelector} from "react-redux";`, and instead of `connect` and `mapStateToProps`, inside the functional component (on top where you define useState hooks):

```js
const dispatch = useDispatch();
const posts = useSelector((state) => state.post.posts);
```

- But whenever you want to dispatch an action, wrap it in dispatch like `dispatch(loadPosts());`.

## Memoizing

- If in one action creator, we fetch posts and in another we fetch users (authors) of that posts, in order to not send multiple calls to the backend for the same user, we have two solutions:

  - `memoize` function of `lodash` to memoize action creator (if we memoize a function, the original function will be called once with the unique arguments and what is returned from that first call will be returned of the memoize function each time later):

  ```js
  export const fetchUser = (id) => (dispatch) => {
    _fetchUser(id, dispatch);
  };

  const _fetchUser = _.memoize(async (id, dispatch) => {
    const { data } = await api.get(`/users/${id}`);

    dispatch({ type: "FETCH_USER", payload: data });
  });
  ```

  - Fetch posts and users in one action creator. We will keep `fetchPosts` and `fetchUser` but in `fetchPostsAndUsers` we will dispatch those action creators (action creator in action creator).

  ```js
  export const fetchPostsAndUsers = () => async (dispatch, getState) => {
    await dispatch(fetchPosts()); //fetchPosts returns a function -> redux-thunk returns what is returning from calling the inner function of fetchPosts, which is a promise -> we will await it

    const userIds = _.uniq(_.map(getState().posts, "userId"));

    userIds.forEach((id) => dispatch(fetchUser(id))); // we don't have to await each dispatch but if we had some logic afterwards,
    // we would write: await Promise.all(userIds.map(id => dispatch(fetchUser(id))))

    // _.chain(getState().posts)
    //   .map("userId")
    //   .uniq()
    //   .forEach((id) => dispatch(fetchUser(id)))
    //   .value(); // <-- better way (more compact)
  };
  ```

## @reduxjs/toolkit

- It is a library that you can use to import `{configureStore, createAction, createReducer, createSlice}` from it.
- `configureStore()`: wraps createStore to provide simplified configuration options and good defaults. It can automatically combine your slice reducers, adds whatever Redux middleware you supply, includes `redux-thunk` by default, and enables use of the Redux DevTools Extension.
- `createReducer()`: that lets you supply a lookup table of action types to case reducer functions, rather than writing switch statements. In addition, it automatically uses the `immer` library to let you write simpler immutable updates with normal **mutative** code, like `state.todos[3].completed = true`.
- `createAction()`: generates an action creator function for the given action type string. The function itself has toString() defined, so that it can be used in place of the type constant.
- `createSlice()`: accepts an object of reducer functions, a slice name, and an initial state value, and automatically generates a slice reducer with corresponding action creators and action types. With this, you don't need createReducer and createAction.
- So we create a `todos.js` in the `src/store` folder:

```js
import { createSlice } from "@reduxjs/toolkit";

let lastId = 0;

const todosSlice = createSlice({
  name: "todos",
  initialState: [],
  reducers: {
    addTodo: {
      reducer(todos, action) {
        todos.push({
          id: ++lastId,
          text: action.payload.text,
          completed: false,
        });
      },
    },
    toggleTodo(todos, action) {
      const todo = todos.find((todo) => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
  },
});

export const { addTodo, toggleTodo } = todosSlice.actions;

export default todosSlice.reducer;
```

- Then, we can import `import { addTodo } from "./store/todos";` and dispatch an action like `store.dispatch(addTodo({ text: "Buy milk" }));`.

## Redux Form

- If you want to store the state of your form in the redux store so if user goes to another component and then comes back to the form, the data will be still there (mostly we don't want that), use `redux-form` library.
- redux-form is overkill for a simple form with few fields and a submit button (in that case, using local state for formData and error is recommended) but great for complex forms such as wizard forms (because in the next page we want to access the form data).
- `npm i redux-form`.
- In redux-form, reducer, action creators and mapStateToProps have been implemented already.
- In `reducers/index.js`, import the reducer from redux-form and connect it to our store:

```js
import { reducer as formReducer } from "redux-form";

export default combineReducers({
  form: formReducer, // the key has to be form
});
```

- Then, we create a component like `SomethingForm.js`:

```js
import React from "react";
import { reduxForm, Field } from "redux-form";
import SomethingField from "./SomethingField";

const SomethingForm = ({ handleSubmit, onSomethingSubmit }) => {
  // handleSubmit is provided automatically by redux-form and it invokes its inner function (written by us) with all the form values so we don't have to do e.preventDefault(); anymore.
  return (
    <div>
      <form onSubmit={handleSubmit(onSomethingSubmit)}>
        <Field
          name="title" // this will be the name of the variable in the redux store (form -> SomethingForm -> values -> title)
          label="Enter title: " // each added props (like this one) to the Field component will automatically be forwarded to the component {SomethingField}
          component={SomethingField} // redux-form passes input object (which has a lot of event handlers: onChange, onBlur,...) and meta
          // also that input object makes the input a controlled element by having value and onChange
        />
        <Field
          name="description"
          label="Enter description: "
          component={SomethingField}
        />
        <button>Submit</button>
      </form>
    </div>
  );
};

const validate = (formData) => {
  const errors = { title: [], description: [] }; // the keys in errors object should be matched with the name of Field component

  if (!formData.title) {
    errors.title.push("You must enter a title");
  }

  if (!formData.description) {
    errors.description.push("You must enter a description");
  }
  return errors;
};
// you can write a helper function and use a package like Joi to validate the fields:
// const validate = formData => {
//   const schema = ...;

//   return validateAgainstSchema(formData, schema);
// };

export default reduxForm({
  // an HOC which provides our component with handleSubmit and initialValues for example
  validate, // it is automatically ran with the form values whenever the form data is changed or form is going to be submitted or even any interaction (blur, focus, ...).
  // if this function responds with {"fieldName": ["error1", "error2"], ...}, it will update the meta for each field
  // and ALSO PREVENTS THE EXECUTION OF THE FUNCTION PASSED TO THE handleSubmit
  form: "SomethingForm", // it creates a fraction of state with the key of this name to store form data
  destroyOnUnmount: false, // when the user wants to review and then come back, all the form values exist
})(SomethingForm);
```

- `SomethingField` contains logic to render a single label and text input:

```js
import React from "react";

export default ({ input, meta: { error: errors, touched }, label }) => {
  return (
    <div>
      <label>{label}</label>
      <input {...input} />
      <div>
        {errors &&
          touched && // touched means the user clicked in and out of a redux-form Field
          errors.map((error) => <p key={error}>{error}</p>)}
      </div>
    </div>
  );
};
```

- Then for `SomethingCreate.js`:

```js
import React from "react";
import { connect } from "react-redux";
import { createSomething } from "../../actions";
import SomethingForm from "./SomethingForm";

const SomethingCreate = ({ createSomething }) => {
  return (
    <div>
      <h3>Create something</h3>
      <SomethingForm
        onSomethingSubmit={(formData) => {
          // this function receives the formData so we can do whatever we want
          createSomething(formData);
        }}
      />
    </div>
  );
};

export default connect(null, { createSomething })(SomethingCreate);
```

- and for `SomethingEdit.js`:

```js
import React, { useEffect } from "react";
import { connect } from "react-redux";
import { fetchSomething, editSomething } from "../../actions";
import SomethingForm from "./SomethingForm";

const SomethingEdit = ({ something, match, fetchSomething, editSomething }) => {
  useEffect(() => {
    fetchSomething(match.params.id);
  }, []);

  if (!something) return <div>Loading...</div>;

  return (
    <div>
      <h3>Edit something</h3>
      <SomethingForm
        initialValues={{
          // this prop will be consumed by reduxForm HOC
          // as you can see, not the whole `something` is sent to the redux form, because otherwise, the whole object will be sent back to the API as formData
          // but we don't want to send id for example
          // we could use {_.pick(something, "title", "description")}
          title: something.title,
          description: something.description,
        }}
        onSomethingSubmit={(formData) => {
          editSomething(match.params.id, formData);
        }}
      />
    </div>
  );
};

const mapStateToProps = (state, ownProps) => {
  return {
    something: state.somethings[ownProps.match.params.id],
  };
};

export default connect(mapStateToProps, { fetchSomething, editSomething })(
  SomethingEdit
);
```

## Middleware functions

- Middleware functions can log, modify or stop actions that have been returned from action creators before sending them to the reducers.

![](/md/141.jpg)

- It is better to design middlewares in such a way that they dispatch another actions so that we don't care much about the order of middlewares.

### Implementation of `redux-promise` middleware

![](/md/142.jpg)

- In the `src` folder, create a folder called `middlewares`. Then `async.js`:

```js
export default ({ dispatch }) =>
  (next) =>
  (action) => {
    // it is how authors of redux decided to design middlewares
    // Check to see if the action
    // has a promise on its 'payload' property
    // If it does, then wait for it to resolve
    // If it doesn't, then send the action on to the
    // next middleware
    if (!action.payload || !action.payload.then) {
      // !action.payload.then is a way to check if something is a promise
      return next(action);
    }

    // We want to wait for the promise to resolve
    // (get its data!!) and then create a new action
    // with that data and dispatch it
    action.payload.then(function (response) {
      const newAction = { ...action, payload: response };
      dispatch(newAction);
    });
  };
```

- Then inside `createStore` and in `applyMiddleware(...)`, import and pass the above middleware.

### Type-checking middleware

- Let's create a middleware that checks to see if the payload is of the correct type (TS is a better solution), `stateValidator.js`:

```js
import tv4 from "tv4";
import stateSchema from "./stateSchema";

export default ({ dispatch, getState }) =>
  (next) =>
  (action) => {
    next(action); // this middleware needs to be called after all the middlewares

    if (!tv4.validate(getState(), stateSchema)) {
      // after the reducer has updated the state
      console.warn("Invalid state schema detected");
    }
  };
```

- `tv4` is a tiny validator which validates a json string against a `JSON Schema` (which can be easily generated by visiting `jsonschema.net` and giving an example -> `stateSchema.js`):

```js
export default {
  $id: "http://example.com/example.json",
  type: "object",
  definitions: {},
  $schema: "http://json-schema.org/draft-07/schema#",
  properties: {
    comments: {
      $id: "/properties/comments",
      type: "array",
      items: {
        $id: "/properties/comments/items",
        type: "string",
        title: "The 0th Schema ",
        default: "",
        examples: ["Comment #1", "Comment #2"],
      },
    },
    auth: {
      $id: "/properties/auth",
      type: "boolean",
      title: "The Auth Schema ",
      default: false,
      examples: [true],
    },
  },
};
```

### Parametrized middleware

- If we want to pass an argument to a middleware, we should change the signature to `param => (store) => (next) => (action) => {` and then call the middleware when passing in applyMiddleware so it will return a middleware with the old signature.

### API call middleware

```js
import axios from "axios";
import { CALL_API } from "../actions/types";
import { apiCallSuccess, apiCallFailed } from "../actions/api.js";

const api =
  ({ dispatch }) =>
  (next) =>
  async (action) => {
    if (action.type !== CALL_API) return next(action);

    const { url, method, data, onStart, onSuccess, onError } = action.payload;

    if (onStart) dispatch({ type: onStart });

    next(action);

    try {
      const response = await axios.request({
        baseURL: "http://localhost:9001/api",
        url,
        method,
        data,
      });
      // General
      dispatch(apiCallSuccess(response.data));
      // Specific
      if (onSuccess) dispatch({ type: onSuccess, payload: response.data });
    } catch (error) {
      // General
      dispatch(apiCallFailed(error.message));
      // Specific
      if (onError) dispatch({ type: onError, payload: error.message });
    }
  };

export default api;
```

- Then we can dispatch actions like this in an action creator like `loadPosts` (we have such action creator in its section above):

```js
dispatch({
  type: CALL_API,
  payload: {
    url: "/posts",
    onSuccess: POSTS_LIST,
  },
});
```

# useReducer and useContext instead of Redux

- An alternative to useState (it is very similar to `useState`). Accepts a reducer of type `(state, action) => newState`, and returns the current `state` paired with a `dispatch` method:

```js
const [state, dispatch] = useReducer(reducer, initialState);
```

- So we create a `src/context/createDataContext.js`:

```js
import React, { useReducer } from "react";

export default (reducer, actions, defaultValue) => {
  const Context = React.createContext();

  const Provider = ({ children }) => {
    const [state, dispatch] = useReducer(reducer, defaultValue);

    const boundActions = {};
    for (let key in actions) {
      boundActions[key] = actions[key](dispatch);
    }

    return (
      <Context.Provider value={{ state, ...boundActions }}>
        {children}
      </Context.Provider>
    );
  };

  return { Context, Provider };
};
```

- Then, inside that `context` folder, for each piece of state, we create a new context in which we write its reducer and all its action creators, for example a `BlogContext.js`:

```js
import createDataContext from "./createDataContext";
import jsonServer from "../apis/jsonServer";

const blogReducer = (state, action) => {
  const { type, payload } = action;

  switch (type) {
    case "GET_BLOGPOSTS":
      return payload;
    case "EDIT_BLOGPOSTS":
      return state.map((blogPost) => {
        return blogPost.id === payload.id ? payload : blogPost;
      });
    case "DELETE_BLOGPOSTS":
      return state.filter((blogPost) => blogPost.id !== payload);
    default:
      return state;
  }
};

const getBlogPosts = (dispatch) => async () => {
  // here, unlike Redux, the first function input is always dispatch and
  // the second function input is for the action creators inputs
  const response = await jsonServer.get("/blogposts");

  dispatch({ type: "GET_BLOGPOSTS", payload: response.data });
};

const addBlogPost = (dispatch) => async (title, content, callback) => {
  await jsonServer.post("/blogposts", { title, content });

  if (callback) {
    // a technique to navigate to a different page
    callback();
  }
};

const deleteBlogPost = (dispatch) => async (id) => {
  await jsonServer.delete(`/blogposts/${id}`);

  dispatch({ type: "DELETE_BLOGPOSTS", payload: id });
};

const editBlogPost = (dispatch) => async (id, title, content, callback) => {
  await jsonServer.put(`/blogposts/${id}`, { title, content });

  dispatch({
    type: "EDIT_BLOGPOSTS",
    payload: { id, title, content },
  });
  if (callback) {
    callback();
  }
};

export const { Context, Provider } = createDataContext(
  blogReducer,
  { addBlogPost, deleteBlogPost, editBlogPost, getBlogPosts },
  []
);
```

- Then we have to make the state available to all components so in `index.js`:

```js
import { Provider as BlogProvider } from './context/BlogContext';
import { Provider as AuthProvider } from './context/AuthContext';
...

<BlogProvider>
  <AuthProvider>
    <App />
  </AuthProvider>
</BlogProvider>
```

- To use the context in each component:

```js
import React, { useContext } from "react";
import { Context } from "../context/BlogContext";

const MyComponent = () => {
  const { state, deleteBlogPost, getBlogPosts } = useContext(Context);

  // use state and action creators in the component
};

export default MyComponent;
```

- The problem is that sharing data between different contexts will be challenging. In redux, in an arbitrary action creator we can get the entire state object by `getState()` and have access to all the data inside redux. So if we want an action creator to have some state form another context, we should give it to the action creator as a parameter. And also there is no middleware.

# useCallback and useMemo

- Both are for performance optimization. So don't use it early!
- On every render, everything that's inside a functional component will run again. If a child component has a dependency on a function from the parent component, the child will re-render every time the parent re-renders even if that function "doesn't change" (the reference changes, but what the function does won't).
- `useCallback` is used for optimization by avoiding unnecessary renders from the child, making the function change the reference only when dependencies change. You should use it when a function is a dependency of a side effect e.g. useEffect.
- Very important: `useCallback` does not run the memoized function.
- `useCallback` returns a `memoized callback`. Normally, if you have a child component that receives a function prop, at each re-render of the parent component, this function will be re-executed; by using useCallback() you ensure that this function is only re-executed when any value on it's dependency array changes:

```js
const Parent = () => {
  const [count, setCount] = React.useState(0);
  const [another, setAnother] = React.useState(0);

  const memoizedCallback = React.useCallback(() => {
    console.log("It is running");
    return count;
  }, [count]);

  return (
    <div>
      <Child callbackFunction={memoizedCallback} />
      <button onClick={() => setCount(count + 1)}>Change callback</button>

      <button onClick={() => setAnother(another + 1)}>
        Do not change callback
      </button>
    </div>
  );
};

const Child = ({ callback }) => {
  const [childValue, setChildValue] = React.useState(0);

  React.useEffect(() => {
    callback(); // useCallback doesn't actually run the function,
    // it simply memoizes it so that between renders the same function isn't recreated.
    // so if we omit this line, there will be no console.log
    setChildValue(childValue + 1);
  }, [callback]);

  return <p>Child value: {childValue}</p>;
};
```

- `useMemo` runs with every re-render, but with cached values. It will only use new values when certain dependencies change. It's used for optimization when you have expensive computations.Note that, `useEffect` will run after each re-render.
- `useMemo` will return a `memoized value` that is the result of the passed parameter. It means that useMemo will make the calculation for some parameter once and it will then return the same result for the same parameter from a cache.
- `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`.
- Doing heavy processing and want to memoize (cache) the results? Use useMemo. `const result = useMemo(heavyProcessFunc(val, val2),[val,val2]);`.

```js
const Parent = () => {
  const [count, setCount] = React.useState(0);
  const [another, setAnother] = React.useState(0);

  const memoizedValue = React.useMemo(() => {
    // Do some heavy processing with the parameter
    console.log(`Cached memo: ${count}`);
    return count;
  }, [count]);

  return (
    <div>
      <Child value={memoizedValue} />
      <button onClick={() => setCount(count + 1)}>Change memo</button>
      <button onClick={() => setAnother(another + 1)}>
        Do not change memo
      </button>
    </div>
  );
};

const Child = ({ value }) => {
  const [childValue, setChildValue] = React.useState(0);

  React.useEffect(() => {
    setChildValue(childValue + 1);
  }, [value]);

  return <p>Child value: {childValue}</p>;
};
```

# Testing

![](/md/137.jpg)

- As a result of creating a react project with `create-react-app`, we will have `jest` installed which is an automated test runner.

![](/md/134.jpg)

- The file `App.test.js` that we get for free:

```js
import React from "react";
import { render } from "@testing-library/react";
import App from "./App";

it("renders learn react link", () => {
  const { getByText } = render(<App />);
  const linkElement = getByText("/learn react/i");
  expect(linkElement).toBeInTheDocument();
});
```

## Enzyme

- With `Enzyme`, we can say that a rendered component has an instance of another component for example.
- Enzyme is competitor of `@testing-library`. They are not test runners (jest is).
- `npm i --save-dev enzyme enzyme-adapter-react-16`.
- In the `src` folder, create a file `setupTests.js` which is very special because `jest` will run it before any tests (if jest finds it) and it is the perfect place for some configurations.
- There are three methods to create instances of our component (these methods return an object that we can write some tests around):

![](/md/135.jpg)

- With `static`, we get only `html` (we can't do anything with it like clicking or interacting). It is not used very often. Usually, `shallow` and `mount` are used.
- `shallow` is perfect for testing a component in isolation.

- Create a folder `__tests__` in the components directory. Then, create an `App.test.js`:
- To test that a component renders another component:

```js
import React from "react";
import { shallow } from "enzyme";
import App from "../App";
import CommentBox from "../CommentBox";

it("shows a comment box", () => {
  const wrapped = shallow(<App />); // the naming is to communicate the fact that the returned object has other functionality. We could call it component.
  expect(wrapped.find(CommentBox).length).toEqual(1); // find method finds every node in the render tree that matches the provided selector. -> returns an array
});
```

- You can use `absolute imports` by configuring the node project (refer to node):

```js
import React from "react";
import { shallow } from "enzyme";
import App from "components/App";
import CommentBox from "components/CommentBox";
import CommentList from "components/CommentList";

it("shows a comment box", () => {
  const wrapped = shallow(<App />);

  expect(wrapped.find(CommentBox).length).toEqual(1);
});

it("shows a comment list", () => {
  const wrapped = shallow(<App />);

  expect(wrapped.find(CommentList).length).toEqual(1);
});
```

- For common setup before each tests `only inside the module`, use `beforeEach`:

```js
let wrapped;

beforeEach(() => {
  wrapped = shallow(<App />);
});

it("shows a comment box", () => {
  expect(wrapped.find(CommentBox).length).toEqual(1);
});

it("shows a comment list", () => {
  expect(wrapped.find(CommentList).length).toEqual(1);
});
```

- To test the `CommentBox`, because it has no children, we can use shallow renderer but for the sake of learning, we will use `mount` here (`CommentBox.test.js`).
- Note that unlike shallow or static rendering, full rendering actually mounts the component in the DOM, which means that tests can affect each other if they are all using the same DOM. Keep that in mind while writing your tests and, if necessary, use `.unmount()` or something similar as cleanup.

```js
import React from "react";
import { mount } from "enzyme";
import CommentBox from "components/CommentBox";

let wrapped;

beforeEach(() => {
  wrapped = mount(<CommentBox />);
});

afterEach(() => {
  wrapped.unmount(); // unmounts the component from fake DOM (JSDOM).
});

it("has a text area and a button", () => {
  expect(wrapped.find("textarea").length).toEqual(1); // find can be used to find normal html elements (by CSS selectors) as well.
  expect(wrapped.find("button").length).toEqual(1);
});
```

- To test that the user can enter an input:

![](/md/136.jpg)

```js
it("has a text area that the user can type in", () => {
  wrapped.find("textarea").simulate("change", {
    target: { value: "new comment" },
  });

  // wrapped.update(); // because updating state is not instantaneous, we have to force it.
  // actually it is not needed. So I commented it out.

  expect(wrapped.find("textarea").prop("value")).toEqual("new comment"); // this prop is not a react prop
});
```

- Note that when `.prop(key)` called on a shallow wrapper, `.prop(key)` will return values for props on the root node that the component renders, not the component itself. To return the props for the entire React component, use `wrapper.instance().props`.

```js
it("should be emptied when the user submits the form", () => {
  wrapped.find("textarea").simulate("change", {
    target: { value: "new comment" },
  });

  wrapped.find("form").simulate("submit");

  expect(wrapped.find("textarea").prop("value")).toEqual("");
});
```

- As you can see the last two tests have a same code for setup, so we can group these two with a `describe` block and have another `beforeEach` for it:

```js
describe("the text area", () => {
  beforeEach(() => {
    wrapped.find("textarea").simulate("change", {
      target: { value: "new comment" },
    });
  });

  it("has a text area that the user can type in", () => {
    expect(wrapped.find("textarea").prop("value")).toEqual("new comment");
  });

  it("should be emptied when the user submits the form", () => {
    wrapped.find("form").simulate("submit");

    expect(wrapped.find("textarea").prop("value")).toEqual("");
  });
});
```

### Redux

- When we add `redux` our tests will fail because of the `Provider` wrapper (we have the connect but there is no provider). To solve the problem, we export a component `Root.js` which accepts a component and wraps it inside a provider and we import that in every test file (and also in `index.js`):

```js
import React from "react";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import reduxPromise from "redux-promise"; // something like redux-thunk.
// with this, as before, you return an action from your action creators (payload is a promise) and not a function like redux-thunk.
// It is limited compared to redux-thunk, because in redux-thunk we can as many async stuff as we want
// and we can dispatch any number of actions as we want
import reducers from "./reducers";

export default ({ children, initialState = {} }) => {
  const store = createStore(
    reducers,
    initialState,
    applyMiddleware(reduxPromise)
  );

  return <Provider store={store}>{children}</Provider>;
};
```

- Then, you just need to import it to the test file and wrap the component with it:

```js
// ...
import Root from "Root";

let wrapped;

beforeEach(() => {
  wrapped = mount(
    <Root>
      <CommentBox />
    </Root>
  );
});
// ...
```

- We should also test our reducers by creating `__test__` inside `reducers` folder. Then `commentsReducer.test.js`:

```js
import commentsReducer from "reducers/commentsReducer";
import { SAVE_COMMENT } from "actions/types";

it("handles actions of type SAVE_COMMENT", () => {
  const action = {
    type: SAVE_COMMENT,
    payload: "New Comment",
  };

  const newState = commentsReducer([], action);

  expect(newState).toEqual(["New Comment"]);
});

it("handles action with unknown type", () => {
  const newState = commentsReducer([], { type: "LKAFDSJLKAFD" });

  expect(newState).toEqual([]);
});
```

- For testing actions, create `__tests__` inside actions and then `comments.test.js`:

```js
import { saveComment } from "actions";
import { SAVE_COMMENT } from "actions/types";

describe("saveComment", () => {
  it("has the correct type", () => {
    const action = saveComment();

    expect(action.type).toEqual(SAVE_COMMENT);
  });

  it("has the correct payload", () => {
    const action = saveComment("New Comment");

    expect(action.payload).toEqual("New Comment");
  });
});
```

- To test a component with some props from redux store (`CommentList.test.js`):

```js
import React from "react";
import { mount } from "enzyme";

import CommentList from "components/CommentList";
import Root from "Root";

let wrapped;

beforeEach(() => {
  const initialState = {
    comments: ["Comment 1", "Comment 2"],
  };

  wrapped = mount(
    <Root initialState={initialState}>
      <CommentList />
    </Root>
  );
});

it("creates one LI per comment", () => {
  expect(wrapped.find("li").length).toEqual(2);
});

it("shows the text for each comment", () => {
  expect(wrapped.render().text()).toContain("Comment 1"); // render method returns a CheerioWrapper (something like jQuery element) around the rendered HTML of the single node's subtree.
  expect(wrapped.render().text()).toContain("Comment 2"); // text method is a cheerio method which returns all the text inside of a component (no html).
});
```

### Integration tests

![](/md/138.jpg)

- `npm i --save-dev moxios`.
- `moxios` will mock the response, because we don't have access to outside API in command line environment for axios. So moxios will respond axios instantly with fake data.
- We create a `__tests__` in the `src` folder and then `integrations.test.js`:

```js
import React from "react";
import { mount } from "enzyme";
import moxios from "moxios";
import Root from "Root";
import App from "components/App";

beforeEach(() => {
  moxios.install(); // intercept any request that axios is trying to issue
  moxios.stubRequest("http://jsonplaceholder.typicode.com/comments", {
    // response for any matching request URL
    status: 200,
    response: [{ name: "Fetched #1" }, { name: "Fetched #2" }],
  });
});

afterEach(() => {
  moxios.uninstall(); // because other requests from other tests might have other responses
});

it("can fetch a list of comments and display them", (done) => {
  // because jest doesn't have any idea of delayed events, we use this done callback.
  const wrapped = mount(
    <Root>
      <App />
    </Root>
  );

  wrapped.find(".fetch-comments").simulate("click");

  moxios.wait(() => {
    // it is like: setTimeout(() => { ... done(); wrapped.unmount();}, 100);
    // but because we don't know that if 100 ms is enough or not, we use moxios.wait.
    wrapped.update(); // force update

    expect(wrapped.find("li").length).toEqual(2);

    done();
    wrapped.unmount();
  });
});
```

# React and Redux with Typescript

## React

- In React, we use Typescript instead of PropTypes (legacy). In the `car.ts`:

```ts
export interface ICar {
  color: string;
  model: string;
  topSpeed?: number;
}
```

In the parent component (`CarList.tsx`):

```tsx
import React from "react";
import CarItem from "./CarItem";
import { ICar } from "./car";

const car1: ICar = {
  color: "blue",
  model: "BMW",
  topSpeed: 200,
};

const car2: ICar = {
  color: "white",
  model: "Benz",
};

const cars = [car1, car2];

const CarList: React.FC = () => {
  return (
    <ul>
      {cars.map((car) => (
        <CarItem car={car} />
      ))}
    </ul>
  );
};

export default CarList;
```

In the child component (`CarItem.tsx`):

```tsx
import React from "react";
import { ICar } from "./car";

interface IProps {
  car: ICar;
}

const CarItem: React.FC<IProps> = ({ car }) => {
  return (
    <li>
      {car.color} {car.model}
    </li>
  );
};

export default CarItem;
```

## Redux

- `src/actions/types/index.ts`:

```js
export enum ActionTypes {
  OPEN_MODAL = "OPEN_MODAL",
  CLOSE_MODAL = "CLOSE_MODAL",
  ...
}
```

- `src/actions/types/modalActions.ts`:

```js
import { ActionTypes } from ".";

export type IModalActions = IOpenModalAction | ICloseModalAction;

export interface IOpenModalAction {
  type: ActionTypes.OPEN_MODAL;
  payload: any;
}

export interface ICloseModalAction {
  type: ActionTypes.CLOSE_MODAL;
}
```

- `src/actions/modal.ts`:

```js
import { ActionTypes } from "./types";
import { IOpenModalAction, ICloseModalAction } from "./types/modalActions";

// Open Modal
export type IOpenModal = (content: any) => void;
export const openModal = (content: any): IOpenModalAction => ({
  type: ActionTypes.OPEN_MODAL,
  payload: content,
});

// Close Modal
export type ICloseModal = () => void;
export const closeModal = (): ICloseModalAction => ({
  type: ActionTypes.CLOSE_MODAL,
});
```

- `src/actions/index.ts`:

```js
export * from "./modal";
...
```

- `src/reducers/modalReducer.ts`:

```js
import { IStore } from "./index";
import { ActionTypes } from "../actions/types";
import { IModalActions } from "../actions/types/modalActions";

const initialState: IStore["modal"] = {
  open: false,
  body: null,
};

export default function (
  state = initialState,
  action: IModalActions
): IStore["modal"] {
  switch (action.type) {
    case ActionTypes.OPEN_MODAL:
      return { ...state, open: true, body: action.payload };

    case ActionTypes.CLOSE_MODAL:
      return { ...state, open: false, body: null };

    default:
      return state;
  }
}
```

- `src/reducer/index.ts`:

```ts
import { combineReducers } from "redux";
import modalReducer from "./modalReducer";
import appReducer from "./appReducer";

export interface IStore {
  modal: {
    open: boolean;
    body: any;
  };
  app: {
    token: string | null;
    appLoaded: boolean;
  };
}
export default combineReducers <
  IStore >
  {
    modal: modalReducer,
    app: appReducer,
  };
```

- To use it:

```js
import React from "react";
import { connect } from "react-redux";

import { actionCreator, IActionCreator } from "../../../actions";
import { IStore } from "../../../reducers";

interface IProps {
  actionCreator: IActionCreator;
  submitError: AxiosResponse | null;
}

const MyComponent: React.FC<IProps> = ({ actionCreator, submitError }) => {
  return (
    // some JSX
    <div></div>
  );
};

const mapStateToProps = ({ user: { error } }: IStore) => ({
  submitError: error,
});

export default connect(mapStateToProps, {
  actionCreator,
})(MyComponent);
```

## New course on React with TS

- To initialize a project:

```
npx create-react-app <appName> --template typescript
```

- Every file that has .jsx needs to be replaced by .tsx and the rest just .ts.
- For props, use an interface with `React.FC<IProps>` and for states, if type inference doesn't work, `const [list, setList] = useState<string[]>([]);`.
- If you define event handlers before using (not inline), you have to annotate them, for example for `onChange` event: `event: React.ChangeEvent<HTMLInputElement>`. For `onDragStart` event: `event: React.DragEvent<HTMLDivElement>`.
- If you want to use refs, you have to specify the type of the HTML element that the ref will be assigned to eventually:

```js
const inputRef = (useRef < HTMLInputElement) | (null > null);
```

Also, because you might forget to assign the ref to an HTML element:

```js
useEffect(() => {
  if (!inputRef.current) {
    return;
  }
  inputRef.current.focus();
}, []);
```

### Redux

- For each piece of store (reducer), it is good to have `data`, `loading` and `error`.
- Put redux stuff in one folder (such as `state`) and have a `index.ts` to communicate with redux.
- A better way to use action creators inside React is, creating a `useActions` hook:

```js
import { useDispatch } from "react-redux";
import { bindActionCreators } from "redux";
import { actionCreators } from "../state";

export const useActions = () => {
  const dispatch = useDispatch();

  return bindActionCreators(actionCreators, dispatch);
};
```

Then, we can:

```js
const { myActionCreator } = useActions();
```

- To use `useSelector`, in the file that we define our combined reducer:

```js
export type RootState = ReturnType<typeof reducer>;
```

Then, we create `useSelector.ts` hook in the `hooks` folder:

```js
import {
  useSelector as _useSelector,
  TypedUsedSelectorHook,
} from "react-redux";
import { RootState } from "../state";

export const useSelector: TypedUsedSelectorHook<RootState> = _useSelector;
```

Then we use this `useSelector` instead of what is coming from `react-redux`.

- Use `immer` for mutating in reducers. To do that, `import produce form "immer";` and then wrap your reducer with produce function call.
- If you have asynchronous calculation (calling an API, doing intensive calculations, ...), store the results in the redux (like a cache?). Do not use useSelector (mapStateToProps to do async stuff -> just sync stuff).

![](/md/173.jpg)
