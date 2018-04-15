# hype

>  graphql in a react suspense era

All GraphQL client API's to date have a double declaration problem. Here's a sample adapted from [the urql example](https://github.com/FormidableLabs/urql/blob/6f9fa91dc2e003fba8bef1ce152f4029ed5f5726/example/src/app/home.tsx):

```js
const Home: React.SFC<{}> = () => (
  <Connect
    query={query(TodoQuery)}
    children={({data}: UrqlProps<ITodoQuery, ITodoMutations>) => {
      return (<div>
            <TodoList todos={data.todos} />
          </div>);
    }}
  />
);

const TodoQuery = `
query {
  todos {
    id
    text
  }
}
`;
```

Everything requested in the graphql query string is then repeated in the code. On top of the ease of creating malformed queries, it is difficult to keep the query and code in sync as data needs change. There has to be a better way.

Here's the proposed hype API (without typescript annotations because I have no idea what those will look like):

```js
const Home = () => (
  <Connect
    children={({query}) => {
      query.todos = ['id', 'text']
      return (<div>
            <TodoList todos={query.todos} />
          </div>);
    }}
  />
);
```

`query` is actually wrapped with ES6 Proxies that throw a Promise wrapping a graphql query when asked for properties it does not have. Once it resolves, React Suspense's behavior is to rerender and the query succeeds as it is stored in cache.
