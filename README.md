# reactNewsSite
This React app creates a news site with fetched stories using server-side pagination.

This app utilizes a 3rd party API to pull news source data from.

A general walkthrough of the pertinent React code is given below:


First the general structure of the useReducer is created. The reducer.js file is only a frame.
```React
const initialState = {}

const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState)
```

Next the fetch request is constructed starting first with loading.
```React
  const fetchStories = async (url) => {
    dispatch({type:'SET_LOADING'})
  }
```

Next a useEffect is setup to run the fetchStories() function on page loads.
```React
  useEffect(()=>{
    fetchStories()
  },[])
```

Next the load event is handled by the reducer. All of the properties from useReducer are passed into state using the spread operator. Currently isLoading is the only property being spread.
```React
const initialState = {
  isLoading: true,
};

 return <AppContext.Provider value={{...state}}>{children}</AppContext.Provider>;
```

Next App.js is setup with its components.
```React
function App() {
  return (
    <>
      <SearchForm />
      <Buttons />
      <Stories />
    </>
  );
}
```

Next import isLoading into Story.js from the global context to make the loading render during story fetching.
```React
const Stories = () => {
  const { isLoading } = useGlobalContext();

  if (isLoading) {
    return <div className="loading"></div>;
  }
  return <h2>stories component</h2>;
};

To prevent undefined for the loading state value the reducer is worked upon since a function that returns an empty object is labeled as undefined. Setting up the function will thus prevent that error.
```React
const reducer = (state, action) => {
  switch (action.type) {
    case "SET_LOADING":
      return { ...state, isLoading: true };
    default:
      throw new Error(`no matching ${action.type} action type`);
  }
};
```

Next the fetch is further worked upon to pull the stories data from the API.
```React
  const fetchStories = async (url) => {
    dispatch({ type: SET_LOADING })
    try{
      const response = await fetch(url)
      const data = await response.json()
      console.log(data);
      
    }catch(error){}
  };
```


Then the initial state is updated with the data labels to be deconstructed/used.
```React
const initialState = {
  isLoading: true,
  hits: [],
  query: 'react',
  page: 0,
  nbPages:0
};
```

Next the correct endpoint is constructed inside of fetchStories(). It is set up to be paginated.
```React
  useEffect(() => {
    fetchStories(`${API_ENDPOINT}query=${state.query}&page=${state.page}`);
  }, []);
```

Once data is successfully fetched another action is dispatched to change the value of the initial hits and nbPages. This results in an error since currently there is no action set up for SET_STORIES inside the reducer function.
```React
onst AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  const fetchStories = async (url) => {
    dispatch({ type: SET_LOADING });
    try {
      const response = await fetch(url);
      const data = await response.json();
      dispatch({
        type: SET_STORIES,
        payload: { hits: data.hits, nbPages: data.nbPages },
      });
    } catch (error) {
      console.log(error);
    }
  };
```
 
 Next the SET_STORIES action type is created in the reducer functions switch statement.
```React
   switch (action.type) {
    case SET_LOADING:
      return { ...state, isLoading: true };
    case SET_STORIES:
      return { ...state, isLoading: false };
    default:
```

Next the hits are sent over in the payload to give it a value instead of an [] as in the initial state. Number of pages too.
```React
    case SET_STORIES:
      return {
        ...state,
        isLoading: false,
        hits: action.payload.hits,
        nbPages: action.payload.nbPages,
      };
```

Next the stories are retreived from the global context (hits)
```React
const Stories = () => {
  const { isLoading, hits } = useGlobalContext();
```

Next the return for Stories is worked on when isLoading is false.
```React
  if (isLoading) {
    return <div className="loading"></div>;
  }
  return (
    <section className="stories">
      {hits.map((story) => {
        console.log(story);
        return <article className="story">single story</article>;
      })}
    </section>
  );
```

At this point the single story components should show up on the screen. Next the data object was destructured to make available the desired properties.
```React
   <section className="stories">
      {hits.map((story) => {
        const {objectID, title, num_comments, url, points, author} = story;
```

Next the return is created to render the deconstructed properties data onto the screen.
```React
        return (
          <article className="story" key={objectID}>
            <h4 className="title">{title}</h4>
            <p className="info">
              {points} points by <span>{author} | </span> {num_comments}{" "}
              comments
            </p>
            <div>
              <a href={url} className="read-link">
                read more
              </a>
              <button className="remove-btn">remove</button>
            </div>
          </article>
        );
```

Next the anchor tag is worked on to direct users to a new tab upon clicks to the Read More button.
```React
              <a
                href={url}
                className="read-link"
                target="_blank"
                rel="noopener noreferrer"
              >
                read more
              </a>
```

Next the functionality is set up for the Remove button.
```React
  const removeStory = id => {
    console.log(id);
  }
  
    <AppContext.Provider value={{ ...state, removeStory }}> 
    
  const { isLoading, hits, removeStory } = useGlobalContext();
  
                <button
                className="remove-btn"
                onClick={() => removeStory(objectID)}
              >
                remove
              </button>
```

Now whenever the Remove button is clicked the id of that story is displayed inside the console and can be used to delete the story off the screen.
```React
  const removeStory = (id) => {
    dispatch({ type: REMOVE_STORY, payload: id });
  };
```

Now the new action is set up inside of the reduce function.
```React
    case REMOVE_STORY:
      return {
        ...state,
        hits: state.hits.filter((story) => story.objectID !== action.payload),
      };
```

Next the search form is addressed starting with setting up handleSearch function which is created and exported via the Provider.
```React
 const handleSearch = (query) => {
    dispatch({type: HANDLE_SEARCH, payload: query})
  }
  
      <AppContext.Provider value={{ ...state, removeStory, handleSearch }}>
```

Next the SearchForm function is worked upon.
```React
const SearchForm = () => {
  const { query, handleSearch } = useGlobalContext();
  return (
    <form
      className="search-form"
      onSubmit={(e) => {
        e.preventDefault();
      }}
    >
      <h2>search hacker news</h2>
      <input type="text" className="form-input" value={query}  />
    </form>
  );
};
```

As the user types into the input the handleSearch should be invoked to constantly refetch new data. 
```React
      <input type="text" className="form-input" value={query} onChange={(e)=>handleSearch(e.target.value)} />
```

Next the reducer is addressed to deal with the handle search action to update the query during input.
```React
     case HANDLE_SEARCH:
        return {
          ...state, query:action.payload, page:0
        }
```

To actually render the newly fetched stories the useEffect dependecy array is updated to fetch new data whenever the value inside of query changes.
```React
  useEffect(() => {
    fetchStories(`${API_ENDPOINT}query=${state.query}&page=${state.page}`);
  }, [state.query]);
```


Next the prev and next buttons are addressed beginning with the handlePage() function.
```React
  const handlePage = (value) => {
    console.log(value);
    //dispatch
  }
  
    return (
    <AppContext.Provider value={{ ...state, removeStory, handleSearch, handlePage }}>
```

Next the buttons component is built up.
```React
const Buttons = () => {
  const { isLoading, page, nbPages, handlePage } = useGlobalContext();
  return (
    <div className="btn-container">
      <button disabled={isLoading} onClick={() => handlePage("dec")}>
        prev
      </button>
      <p>
        {page + 1} of {nbPages}
      </p>
      <button disabled={isLoading} onClick={() => handlePage("inc")}>
        next
      </button>
    </div>
  );
};
```

Now a new action is dispatched from context.js
```React
  const handlePage = (value) => {
    dispatch({ type: HANDLE_PAGE, payload: value });
  };
```

Then that new action is carried out in the reducer.
```React
    case HANDLE_PAGE:
      if (action.payload === "inc") {
        let nextPage = state.page + 1;
        if (nextPage > state.nbPages - 1) {
          nextPage = 0;
        }
        return { ...state, page: nextPage };
      }
 ```
 
 Lastly decrease is taken care of.
 ```React
       if (action.payload === "dec") {
        let prevPage = state.page - 1;
        if (prevPage < 0) {
          prevPage = state.nbPages - 1;
        }
        return { ...state, page: prevPage };
      }
 ```
 
 
Now a problem arises where the buttons do not fetch new data. This is because currently the only dependency array inside of the useEffect is that of the query, there is nothing telling the fetch request to be run on the button clicks. So by adding that property as a dependency in useEffect the buttons will gain their functionality.
```React
  useEffect(() => {
    fetchStories(`${API_ENDPOINT}query=${state.query}&page=${state.page}`);
  }, [state.query, state.page]);
```

***End walkthrough
