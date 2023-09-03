- 
```elm
type Msg = Increment | Decrement | FetchData | DataFetched String

fetchDataCmd : Cmd Msg
fetchDataCmd =
    -- Simulating an API call that will result in DataFetched "some data"
    Task.succeed "some data"
        |> Task.perform DataFetched

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        Increment ->
            (model + 1, Cmd.none)
        Decrement ->
            (model - 1, Cmd.none)
        FetchData ->
            (model, fetchDataCmd)
        DataFetched data ->
            (model, Cmd.none)  -- Normally you would update the model with the fetched data
```

-
```elm
type Msg = Increment | Decrement | FetchData | DataFetched String
type alias Model =
    { count : Int
    , data : String
    }

fetchDataCmd : Cmd Msg
fetchDataCmd =
    -- Simulating an API call that will result in DataFetched "some data"
    Task.succeed "some data"
        |> Task.perform DataFetched

update : Msg -> Model -> (Model, Cmd Msg)
update msg model =
    case msg of
        Increment ->
            ({ model | count = model.count + 1 }, Cmd.none)
        Decrement ->
            ({ model | count = model.count - 1 }, Cmd.none)
        FetchData ->
            (model, fetchDataCmd)
        DataFetched newData ->
            ({ model | data = newData }, Cmd.none)
```

