
I have computed observables and app state which depends upon a Repository, which sources from a database via gateway. the observable is the internal state value which is accessed via getter and setter in the repository. if there is no call to the database and the repository is the arbiter of the data, meaning it maintains its own state and uses that state to update the observable
can you show me, based on this code, a dependency injection workflow where each of these observables has their own database? 
The end result I want is for the 

```ts
export function BooksComposer({ observable }: BooksComposerProps) {
  const title = "booksComposer same as booksChild data";
  
  const database = DatabaseFactory.createDatabase();
  const httpGateway = HttpGatewayFactory.createHttpGateway(database);
  const repository = RepositoryFactory.createRepository(observable, httpGateway);
  const booksPresenter = PresenterFactory.createPresenter(repository);

  const data = booksChild;
  const [dataValue, setDataValue] = React.useState([]);
  
  React.useEffect(() => {
    const dataSubscription = booksPresenter.load((value) => {
      setDataValue(value);
    });
    return () => {
      dataSubscription();
    };
  }, []);
  
  // ...
}

```