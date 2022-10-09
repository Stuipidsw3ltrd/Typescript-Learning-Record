# Apollo

## Create a client and provide it to the child component

```react
import { ApolloClient, ApolloProvider } from '@apollo/client';

const client = new ApolloClient({
  uri: "https://rickandmortyapi.com/graphql", // the graphQL api URL
  cache: new InMemoryCache() // if the query is same, then return the cache result
})

... // use ApolloProvider component to wrap the child component, and pass the client as context
    <ApolloProvider client={client}> 
    <App />
    </ApolloProvider>
...
```

## useQuery()

```react
import { useQuery, gql } from "@apollo/client"; // 可以直接调用Apollo Client中的方法，因为我们已经在Apollo的Context下

const GET_CHARACTERS = gql`
  query {
    characters {
      results {
        id
        name
        image
        gender
      }
    }
  }
`; 
```

所有用在Apollo Client中的graphQL语句，都可以用gql这个标识符加format字符串符号来定义，字符串的内容就是一个graphQL语句

```react
export function useCharacters() {
  const {error, data, loading} = useQuery(GET_CHARACTERS);
  
  return {
    error,
    data,
    loading
  }
}
```

接着，可以将定义好的graphQL语句传入到useQuery当中，useQuery会返回三个对象，error代表错误信息（没有的时候是undefined），data代表我们获取到的数据（没有的时候是undefined），loading是一个布尔值，代表当前是否正在获取数据的过程当中，可以用来显示加载Icon。

### 带参数的useQuery()

```react
const GET_CHARACTERS = gql`
  query GetLocation($name: String!) { // 这里形参的顺序和位置很重要
    characters(filter: { name: $name }) {
      results {
        location {
          name
        }
      }
    }
  }
`; // filter是在graphQL server端定义好的参数
```

在使用这个graphQL语句的时候，需要提供额外的variable参数，注意：这个variable中的参数，是按query函数当中定义的形参顺序来提供的

```react
export const useLocations = (roleName: string) => {
    const [getLocations, {error, data, loading}] = useLazyQuery(GET_CHARACTERS, {
        variables: {
            roleName // 如果不提供形参名字，那么需要按 GetLocation 函数中的形参顺序提供参数
        }
    })

    return [getLocations, {
        error,
        data,
        loading
    }]
}
```



## useLazyQuery()

useQuery会在组件一加载好的时候就去请求数据，有时候我们可能不想这样，而是在某些特定情况触发时才请求数据，这个时候我们就可以用useLazyQuery()这个hook

```react
import { gql, useLazyQuery } from "@apollo/client";
import React, { useState } from "react";

interface ILocation {
    location:{
        name: string;
    }

}

const GET_CHARACTERS = gql`
  query GetLocation($name: String!) {
    characters(filter: { name: $name }) {
      results {
        location {
          name
        }
      }
    }
  }
`;

export default function Search() {
  const [name, setName] = useState("");
  // useLazyQuery会返回一个额外方法，getLocation（名字是自己定的），调用这个方法，就可以触发数据请求
  const [getLocations, { error, data, loading }] = useLazyQuery(
    GET_CHARACTERS,
    {
      variables: { name },
    }
  );

  return (
    <div>
      <input
        placeholder="name..."
        onChange={(e) => {
          setName(e.target.value);
        }}
        value={name}
      ></input>
      <button
        onClick={() => {
          getLocations(); // 触发数据请求，触发之后，error,data和loading也会随之更新
        }}
      >
        Search
      </button>
      {console.log(data)}
      {loading && <div>loading...</div>}
      {error && <div>something wrong...</div>}
      {data && (
        <ul>
          {data.characters.results.map((c: ILocation) => {
            return <li>{c.location.name}</li>;
          })}
        </ul>
      )}
    </div>
  );
}

```

## useMutation()

useQuery()是用来查询服务端数据的，而useMutation()则是负责更改服务端的数据（添加、更新）

useMutation() 的使用方法跟useLazyQuery() 非常相似，会返回一个触发mutation函数的方法，以及`{error, loading, data}` 这些东西。

```react
const FEATURED_SPEAKER = gql`
  mutation markFeatured($speakerId: ID!, $featured: Boolean!) {
    markFeatured(speakerId: $speakerId, featured: $featured) {
      id
      featured
    }
  }
`;
```

上面这个gql语句中定义了一个mutation，这个mutation的作用是toggle一个speaker的标记状态（featured），它接收speakerId和featured两个参数，返回id和featured两个数据。

现在有了这个语句，我们可以有两种使用方法，一种是在使用useMutation hook的时候把参数传进去，另一种是调用触发函数时的提供参数：

init时传入参数：

```react
const [markFeatured, { loading, error }] = useMutation(
    FEATURED_SPEAKER,
    {
      variables: { speakerId: CONST_ID, featured: featured },
    }
  );

<button onclick={markFeatured}></button>
```

调用触发函数时传入参数：

```react
const [markFeatured, { loading, error }] = useMutation(
    FEATURED_SPEAKER
  );
<button
onClick={markFeatured({
       variables: {
          speakerId: speakerId,
          featured: true,
       }
     });
}></button>
```

useMutation还有一个onComplete参数，为mutation提供一个回调函数，可以在完成mutation的时候触发这个回调

```react
const [markFeatured] = useMutation(
    FEATURED_SPEAKER,
    {
      variables: { speakerId: CONST_ID, featured: featured },
      onComplete: (data) => {console.log(data)}
    }
  );
```

