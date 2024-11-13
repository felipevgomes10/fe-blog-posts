<!-- Padrões de busca de dados no React -->

<!-- 
    Nos aplicativos da web de hoje, a busca eficiente de dados é crucial para fornecer experiências excepcionais ao usuário. Este guia abrangente explora padrões avançados de busca de dados usando TanStack Query (anteriormente React Query), com foco em estratégias práticas para melhorar o desempenho percebido e real do aplicativo.
 -->

 <!-- https://utfs.io/a/oqi3glmmqm/WfWc1HX19bacfgrcXJZNz1yDPrIUm6tXVKWsRvwL7xh3c49n -->

## Introdução

Padrões de data fetching são estratégias que podem ser utilizadas para melhorar a performance percebida da aplicação. Ao utilizar esses padrões, proporcionamos aos usuários a sensação de que a aplicação é mais rápida do que realmente é. Existem várias maneiras de alcançar isso em uma aplicação client-side, cada uma igualmente importante e servindo seu propósito único de melhorar a Experiência do Usuário e até mesmo a Experiência do Desenvolvedor.

## Formas de caching

![kylo](https://utfs.io/a/oqi3glmmqm/WfWc1HX19bac2DgDwOWICGfsXt1UAzZBmRJd7OWolP4TLjkq)

Cache em memória é um dos mecanismos mais importantes em aplicações client-side que pode ser usado para melhorar a performance percebida pelos usuários. No entanto, é importante mencionar que o cache em memória é diferente do cache do navegador. Enquanto o cache do navegador é algo habilitado por padrão nos navegadores modernos, o cache em memória é controlado pela própria aplicação e pode ser feito manualmente ou usando bibliotecas de terceiros, além de poder ser persistido usando as opções nativas de armazenamento disponíveis nos navegadores.

Cache em memória é uma estratégia que consiste em salvar os dados buscados em memória, geralmente usando um Map para maior eficiência. Uma maneira simples de implementar isso seria:

```ts
// Implementão simples de cache em memória
const cache = new Map()

// Busca no cache
const getCachedData = (key) => {
  const item = cache.get(key)
  
  if (!item) return null
  
  const now = Date.now()
  
  if (now > item.expiry) {
    cache.delete(key)
    
    return null
  }
  
  return item.data
}

// Grava no cache
const setCacheData = (key, data, ttlSeconds = 300) => {
  cache.set(key, {
    data,
    expiry: Date.now() + (ttlSeconds * 1000)
  })
}

// Uso
const loadData = async (id) => {
  const cachedData = getCachedData(id)

  if (cachedData) return cachedData

  const { data } = await myService.getById(id)
  setCacheData(id, data)

  return data
}
```

Embora esta implementação possa parecer suficiente, existem situações que precisam ser consideradas como: tratamento de erros, robustez, performance, condições de corrida, dados desatualizados, gerenciamento de estado e várias outras questões que podem surgir. Portanto, uma abordagem mais conveniente seria usar algo que já está sendo utilizado e testado por milhões de outros desenvolvedores, como o [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview).

## Tanstack Query

Basicamente, [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview) é um wrapper em torno de requisições HTTP que gerencia queries e mutations em aplicações client-side. Esta ferramenta permite que os desenvolvedores se concentrem em construir ótimas experiências enquanto deixam a complexidade de lidar com o cache para a própria ferramenta.

Embora o Tanstack Query gerencie o cache por conta própria, ele também fornece um controle refinado sobre o cache armazenado em memória. Além disso, possui uma série de métodos integrados que permitem que a aplicação intervenha e assuma o controle do que está acontecendo nos bastidores.

A configuração básica de uma instância Tanstack Query para uma aplicação React seria:

```tsx
import {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { getTodos, postTodo } from '../my-api'

// Cria a instância
const queryClient = new QueryClient()

function App() {
  return (
    // Fornece a instância pro contexto
    <QueryClientProvider client={queryClient}>
      <Todos />
    </QueryClientProvider>
  )
}

function Todos() {
  // Acessa instância 
  const queryClient = useQueryClient()

  // Busca os dados
  const query = useQuery({ queryKey: ['todos'], queryFn: getTodos })

  // Muda os dados
  const mutation = useMutation({
    mutationFn: postTodo,
    onSuccess: () => {
      // Invalida e busca novamente
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  return (
    <div>
      <ul>{query.data?.map((todo) => <li key={todo.id}>{todo.title}</li>)}</ul>

      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now(),
            title: 'Do Laundry',
          })
        }}
      >
        Add Todo
      </button>
    </div>
  )
}

render(<App />, document.getElementById('root'))

// Fonte: https://tanstack.com/query/latest/docs/framework/react/quick-start
```

O exemplo acima ilustra o básico de como trabalhar com Tanstack Query. Ele cobre a criação de um [query client estável](https://tanstack.com/query/latest/docs/eslint/stable-query-client), envolvendo sua aplicação com um QueryClientProvider, acessando o query client de dentro de um componente em qualquer lugar na árvore do React, fazendo uma query, realizando uma mutation e invalidando os dados desatualizados de uma query específica.

## Como funciona o cache do Tanstack Query?

O cache do Tanstack Query funciona em torno de um conceito chamado [query key](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys). Cada query deve ter uma query key associada, que é simplesmente um array de valores usado para distinguir uma query de outra. A query key é usada como chave para aquela query específica dentro da estrutura de dados do cache em memória. Com essa chave, é fácil para a aplicação manipular os dados de uma query específica.

```tsx
import { useQuery } from '@tanstack/react-query'
import { getUser } from '../my-api'

function User() {
  // Query key usada para cachear os dados
  const queryKey = ['user', '1']
  
  const query = useQuery({ queryKey, queryFn: getTodos })

  return (
    <div>
	  <span>{query.data?.name}</span>
	  <span>{query.data?.address}</span>
    </div>
  )
}
```

Agora, os dados para aquele usuário específico estão em cache, e sempre que a aplicação precisar desses dados, ela usará sua versão em cache e revalidará em segundo plano, o que permite que os usuários tenham uma melhor experiência, pois não precisam esperar por dados que já foram buscados anteriormente. Além disso, agora os dados em cache podem ser acessados e manipulados usando:

- [`getQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientgetquerydata)
- [`getQueriesData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientgetqueriesdata)
- [`setQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientsetquerydata)
- [`setQueriesData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientsetqueriesdata)

Também é possível [invalidar](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation) queries baseado em suas query keys:

- [`invalidateQueries`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientinvalidatequeries)

Outro aspecto importante de caching são os dados desatualizados (stale data). Os dados são considerados desatualizados quando a aplicação não possui mais sua versão mais atualizada. O Tanstack Query tem uma opção de stale time, que é importante para gerenciar por quanto tempo os dados buscados pelas queries são considerados fresh (novos). O stale time tem valor padrão de 0 milissegundos, o que significa que as queries serão atualizadas em segundo plano toda vez que o componente for montado ou quando a janela for refocada. O stale time pode ser personalizado de acordo com as necessidades da aplicação para otimizar a performance e reduzir buscas desnecessárias de dados.

```ts
const { data } = useQuery({ 
  queryKey: ['todos'], 
  queryFn: fetchTodos, 
  staleTime: 60 * 1000, // 60 segundos 
});
```

No exemplo acima, toda vez que uma query com uma query key definida como `['todos']` for feita dentro de 60 segundos, o Tanstack Query usará os dados armazenados dentro do cache em vez de chamar a API novamente.

Alternativamente, um stale time padrão pode ser definido em um query client e compartilhado com todas as queries, a menos que seja sobrescrito pela própria query.

```ts
const queryClient = new QueryClient({ 
  defaultOptions: { 
    queries: { 
      staleTime: 60 * 1000, // 60 segundos 
    }, 
  }, 
});
```

Para mais informações sobre caching no Tanstack Query:

- [Padrões importantes](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [Exemplos de caching](https://tanstack.com/query/latest/docs/framework/react/guides/caching)
- [Opções de query](https://tanstack.com/query/latest/docs/framework/react/guides/query-options)

## Dados iniciais vs dados placeholder

As opções de dados iniciais e dados placeholder são duas maneiras diferentes de informar como a query precisa se comportar quando não há dados armazenados em cache. 

A opção de dados iniciais fornece à query os dados (não parciais) que serão exibidos inicialmente na aplicação, pulando o estado de carregamento e cacheando seu valor, sendo útil quando os dados já estão disponíveis de alguma forma. No entanto, como o stale time tem valor padrão de 0 milissegundos, a query será revalidada imediatamente, então para evitar isso o seguinte pode ser feito:

```ts
// Mostra initialTodos imediatamente, mas não irá buscar novamente os dados até que outro evento de interação seja encontrado após 1000 ms
const result = useQuery({
  queryKey: ['todos'],
  queryFn: () => fetch('/todos'),
  initialData: initialTodos,
  staleTime: 60 * 1000, // 1 minuto
  // Isso poderia ser 10 segundos atrás ou 10 minutos atrás
  initialDataUpdatedAt: initialTodosUpdatedTimestamp, // ex. 1608412420052
})

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/initial-query-data
```

Ao comparar a diferença entre `staleTime` e `initialDataUpdatedAt`, a query sabe se os dados iniciais estão novos ou não e precisam ser revalidados.

Outra opção é passar uma função como dados iniciais para a query. A função será executada apenas uma vez quando a query for inicializada.

```ts
const result = useQuery({
  queryKey: ['todo', todoId],
  queryFn: () => fetch(`/todos/${todoId}`),
  staleTime: 60 * 1000, // 1 minuto
  initialData: () => queryClient.getQueryData(['todos'])?.find((d) => d.id === todoId), // Usa um todo da query 'todos' como dados iniciais para esta query
  initialDataUpdatedAt: () => queryClient.getQueryState(['todos'])?.dataUpdatedAt // Obtém a última atualização da query 'todos'
})
```

Mais uma coisa importante ilustrada no exemplo acima é como o cache de outras queries pode ser usado para popular o cache de outra query com a opção de dados iniciais. Usando o query client e seus métodos integrados `getQueryData` e `getQueryState`, é possível acessar os dados e o estado de outras queries e usá-los conforme a aplicação necessitar.

Por outro lado, há a opção de dados placeholder que permite que as queries se comportem como se já tivessem dados, similar à opção de dados iniciais, mas não persiste os dados no cache. É útil quando há dados parciais ou falsos suficientes que podem ser exibidos antecipadamente enquanto os dados reais são buscados em segundo plano.

```ts
function Todos() {
  const result = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/todos'),
    placeholderData: placeholderTodos,
  })
}

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
```

Adicionalmente, a versão de função da opção de dados placeholder tem acesso aos dados anteriores daquela query, o que significa que é possível evitar mostrar estados de carregamento ao realizar buscas com queries dinâmicas como: buscar algo por seu id, ou uma [query paginada](https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries#better-paginated-queries-with-placeholderdata).

```ts
const result = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetch(`/todos/${id}`),
  placeholderData: (previousData, previousQuery) => previousData,
})

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/placeholder-query-data
```

ou simplesmente:

```ts
import { useQuery, keepPreviousData } from '@tanstack/react-query'

const result = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetch(`/todos/${id}`),
  placeholderData: keepPreviousData
})
```

## Prefetching de dados

Ao renderizar componentes, frequentemente existem casos onde um componente filho que precisa buscar algum dado necessita que a query do pai termine de carregar para que possa ser renderizado e então começar a buscar os dados que realmente precisa. Isso gera o que é chamado de [request waterfall](https://tanstack.com/query/latest/docs/framework/react/guides/request-waterfalls) e pode levar a uma experiência ruim do usuário e baixa performance. Por exemplo:

```tsx
function Article({ id }) {
  const { data: articleData, isPending } = useQuery({
    queryKey: ['article', id],
    queryFn: getArticleById,
  })

  if (isPending) {
    return 'Loading article...'
  }

  return (
    <>
      <ArticleHeader articleData={articleData} />
      <ArticleBody articleData={articleData} />
      <Comments id={id} />
    </>
  )
}

function Comments({ id }) {
  const { data, isPending } = useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
  })

  ...
}
  
// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Isso gera a seguinte cascata:

```
1. |> getArticleById()
2.   |> getArticleCommentsById()

Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Uma abordagem para contornar isso é fazer o prefetch dos dados assim:

```tsx
function Article({ id }) {
  const { data: articleData, isPending } = useQuery({
    queryKey: ['article', id],
    queryFn: getArticleById,
  })

  // Prefetch
  useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
    // Otimização opcional para evitar re-renderizações quando esta query muda:
    notifyOnChangeProps: [],
  })

  if (isPending) {
    return 'Loading article...'
  }

  return (
    <>
      <ArticleHeader articleData={articleData} />
      <ArticleBody articleData={articleData} />
      <Comments id={id} />
    </>
  )
}

function Comments({ id }) {
  const { data, isPending } = useQuery({
    queryKey: ['article-comments', id],
    queryFn: getArticleCommentsById,
  })

  ...
}

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Agora a cascata se parece com:

```
1. |> getArticleById()
1. |> getArticleCommentsById()

Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Além disso, é possível fazer prefetch de dados dentro da função `queryFn` de uma query e para isso é possível usar o query client:

```ts
const queryClient = useQueryClient()

const { data: articleData, isPending } = useQuery({
  queryKey: ['article', id],
  queryFn: (...args) => {
    queryClient.prefetchQuery({
      queryKey: ['article-comments', id],
      queryFn: getArticleCommentsById,
    })

    return getArticleById(...args)
  },
})

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Note que se o componente estiver sendo suspenso, a aplicação precisa usar `usePrefetchQuery` e `useSuspenseQuery`. Para mais informações sobre isso:

- [Prefetching](https://tanstack.com/query/latest/docs/framework/react/guides/prefetching)
- [`usePrefetchQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/usePrefetchQuery)
- [`useSuspenseQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseQuery)

Outra forma de fazer prefetch de dados é identificando as intenções do usuário com event handlers e então buscando os dados que possivelmente serão necessários. É possível fazer isso usando o método [`prefetchQuery`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientprefetchquery).

Este método apenas popula o cache da query e não retorna nada.

```tsx
function ShowDetailsButton() {
  const queryClient = useQueryClient()

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['details'],
      queryFn: getDetailsData,
      // Prefetch só é disparado quando os dados são mais antigos que o staleTime,
      // então em um caso como este você definitivamente quer definir um
      staleTime: 60000,
    })
  }

  return (
    <button onMouseEnter={prefetch} onFocus={prefetch} onClick={...}>
      Show Details
    </button>
  )
}

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/prefetching
```

Fazendo isso, toda vez que o usuário passar o mouse ou focar no elemento, os dados serão pré-buscados. É importante definir um stale time nesses casos para que a aplicação não faça muitas requisições desnecessárias ao servidor.

## Garantindo os dados da query

O Tanstack Query fornece um método para popular o cache do query client chamado [`ensureQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientensurequerydata) que pode ser usado para garantir que os dados de uma determinada query estejam disponíveis quando necessário. Garantir que os dados da query estejam lá significa que o método vai verificar se a query que está sendo feita tem dados em cache que não estão obsoletos e retorná-los, e se a query ainda não tiver dados em cache, o método `ensureQueryData` chama a API internamente, salva sua resposta no cache e retorna os dados.

```ts
const data = await queryClient.ensureQueryData({ queryKey, queryFn })

// Fonte: https://tanstack.com/query/latest/docs/reference/QueryClient#queryclientensurequerydata
```

Isso é útil quando a aplicação precisa fazer prefetch de dados que podem já estar em cache. Por exemplo, dentro de loaders de rotas, pois a página poderia ser acessada diretamente via URL e isso exigiria que todos os dados fossem carregados do zero dentro do loader, ou a página poderia ser aberta pelo usuário navegando até ela usando um link que pode ser usado para fazer prefetch dos dados da página de destino quando o mouse estiver sobre ele, o que significa que o loader da rota não precisa pré-buscar os dados novamente.

## Queries infinitas

Também é possível realizar queries baseadas em cursor com carregamento infinito usando o Tanstack Query. Usando os hooks [`useInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useInfiniteQuery), [`useSuspenseInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/useSuspenseInfiniteQuery) e [`usePrefetchInfiniteQuery`](https://tanstack.com/query/latest/docs/framework/react/reference/usePrefetchInfiniteQuery), é possível carregar infinitamente qualquer query baseada em cursor em qualquer situação que a aplicação possa enfrentar.

```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

function Projects() {
  const fetchProjects = async ({ pageParam }) => {
    const res = await fetch('/api/projects?cursor=' + pageParam)
    return res.json()
  }

  const {
    data,
    error,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery({
    queryKey: ['projects'],
    queryFn: fetchProjects,
    initialPageParam: 0,
    getNextPageParam: (lastPage, pages) => lastPage.nextCursor,
  })

  return status === 'pending' ? (
    <p>Loading...</p>
  ) : status === 'error' ? (
    <p>Error: {error.message}</p>
  ) : (
    <>
      {data.pages.map((group, i) => (
        <React.Fragment key={i}>
          {group.data.map((project) => (
            <p key={project.id}>{project.name}</p>
          ))}
        </React.Fragment>
      ))}
      <div>
        <button
          onClick={() => fetchNextPage()}
          disabled={!hasNextPage || isFetchingNextPage}
        >
          {isFetchingNextPage
            ? 'Loading more...'
            : hasNextPage
              ? 'Load More'
              : 'Nothing more to load'}
        </button>
      </div>
      <div>{isFetching && !isFetchingNextPage ? 'Fetching...' : null}</div>
    </>
  )
}

// Fonte: https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries
```

Além disso, é possível fazer prefetch dos dados de queries infinitas dentro de effects, callbacks e fora do React usando:

- [`prefetchInfiniteQuery`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientprefetchinfinitequery)
- [`ensureInfiniteQueryData`](https://tanstack.com/query/latest/docs/reference/QueryClient/#queryclientensureinfinitequerydata)

Para mais detalhes sobre as opções de query infinita:

[Opções de query infinita](https://tanstack.com/query/latest/docs/framework/react/reference/infiniteQueryOptions)

## Devtools

O Tanstack Query vem com um componente integrado que ajuda os desenvolvedores a debugar as queries e mutations sendo feitas pela aplicação. Com este componente, é possível ver o estado das queries e mutations, bem como manipular dados e disparar eventos manualmente. É altamente recomendado ter isso instalado na aplicação.

O estado de uma query pode ser:

- Fresh (Novo)
- Fetching (Buscando)
- Paused (Pausado)
- Stale (Obsoleto)
- Inactive (Inativo)

[Devtools](https://tanstack.com/query/latest/docs/framework/react/devtools)

## Loaders de rotas

A maioria dos roteadores client-side modernos permite que as aplicações tenham uma função loader que executa fora do framework sendo usado, o que significa que o loader será executado antes do framework renderizar a página.

No [React Router Dom](https://reactrouter.com/en/main), desde a versão 6, é possível configurar um loader para cada rota individual da seguinte forma:

```ts
createBrowserRouter([
  {
    path: "/teams/:teamId",
    loader: ({ params }) => {
      return fakeGetTeam(params.teamId)
    }
  }
])

// Fonte: https://reactrouter.com/en/main/route/loader
```

Usando loaders, é possível recuperar dados de qualquer lugar, como serviços externos, e disponibilizá-los para o componente sendo renderizado nesta rota.

```ts
function loader({ request }) {
  const url = new URL(request.url)
  const searchTerm = url.searchParams.get("q")
  return searchProducts(searchTerm)
}

// Fonte: https://reactrouter.com/en/main/route/loader
```

Então é possível acessar os dados usando o hook [`useLoaderData`](https://reactrouter.com/en/main/hooks/use-loader-data):

```ts
function SomeRoute() {
  const data = useLoaderData()
  // { some: "thing" }
}

// Fonte: https://reactrouter.com/en/main/route/loader
```

Outra coisa importante a se notar é que ao fazer lazy loading de componentes, o React Router Dom vai chamar as funções lazy e loader ao mesmo tempo, então quando o navegador terminar de baixar o javascript para renderizar a página, é altamente possível que o loader tenha terminado de executar, o que significa que os dados estarão disponíveis quase instantaneamente.

## Integrando loaders de rotas e Tanstack Query

Embora seja possível usar apenas as capacidades do roteador para disponibilizar dados antecipadamente na aplicação, é ainda melhor combinar ambos e aproveitar seus pontos fortes.

Dito isso, a aplicação pode aproveitar a forma como os loaders funcionam e o momento em que são executados no ciclo de vida para fazer prefetch de recursos usando Tanstack Query.

```ts
import { createBrowserRouter } from 'react-router-dom'
import { QueryClient } from '@tanstack/react-query'
import { getUsers } from '@api/get-users'

const queryClient = new QueryClient()

createBrowserRouter([
  {
    path: "/some-route",
    loader: async () => {
      // Garante que toda query que tem ['users'] como query key tenha dados em cache
      await queryClient.ensureQueryData({
        queryKey: ['users'],
        queryFn: getUsers
      })

      // Precisa retornar algo, mesmo que seja null
      return null
    }
  }
])
```

## Boas práticas e performance

Existem algumas boas práticas que são aconselhadas a serem seguidas ao usar o Tanstack Query. Ao garantir que essas práticas são seguidas, é mais provável que a performance percebida, manutenibilidade e UX da aplicação sejam mantidas em um bom nível.

## Aguardando promises

Em relação à performance de prefetch de dados de queries, é importante prestar atenção ao código bloqueante, pois às vezes aguardar uma promise ser resolvida não é necessário para a aplicação.

```ts
// Código não bloqueante
function prefetch() {
  queryClient.prefetchQuery(queryOptions)
}

// Código bloqueante
async function prefetch() {
  await queryClient.prefetchQuery(queryOptions)
}
```

Não há nada depois da chamada `prefetchQuery`, então não é necessariamente preciso aguardar a promise, a menos que seja intencional.

Outra coisa importante a se ter em mente é usar concorrência de promises quando necessário. Às vezes uma função precisa fazer prefetch de mais de uma query ao mesmo tempo e isso pode levar a uma performance ruim e tempos de carregamento longos se as promises não forem resolvidas concorrentemente.

```ts
// Ruim - aguarda cada promise ser resolvida
function prefetch() {
  await queryClient.prefetchQuery(queryOptions1)
  await queryClient.prefetchQuery(queryOptions2)
  await queryClient.prefetchQuery(queryOptions3)
}

// Bom - resolve promises concorrentemente
async function prefetch() {
  await Promise.all([
    queryClient.prefetchQuery(queryOptions1),
    queryClient.prefetchQuery(queryOptions2),
    queryClient.prefetchQuery(queryOptions3),
  ])
}
```

Note que às vezes uma query pode precisar de uma informação que retorna de outra query, então fazer prefetch da query antecipadamente ou persistir os valores importantes em lugares que podem ser acessados globalmente como o local storage ou a URL podem ser boas estratégias.

## Reusabilidade e isolamento

Uma boa prática adicional é isolar queries em funções para que possam ser reutilizadas facilmente em diferentes lugares. Além disso, é importante lembrar de expor as coisas importantes e ao mesmo tempo manter um bom nível de encapsulamento. O Tanstack Query tem uma função auxiliar chamada [`queryOptions`](https://tanstack.com/query/latest/docs/framework/react/reference/queryOptions#queryoptions) que facilita isso:

```ts
// users-query-options.ts
import { getUsers } from "@/services/users-service/users-service"
import { keepPreviousData, queryOptions } from "@tanstack/react-query"

export const usersQueryOptions = queryOptions({
  queryKey: ["users"],
  queryFn: getUsers,
  placeholderData: keepPreviousData, // Evita transições bruscas de dados
})

// query.types.ts
export type QueryParams<TData, TReturn> = {
  select?: (data: TData) => TReturn // Habilita mapeamento de dados da query
  initialData?: TData | (() => TData)
}

// use-users.ts
import { usersQueryOptions } from "@/lib/react-query/users-query-options/users-query-options"
import { useQuery } from "@tanstack/react-query"

import type { QueryParams } from "@/lib/react-query/query.types"
import type { ApiUser } from "@/services/users-service/users-service"

export function useUsers<TReturn = ApiUser[]>({
  select,
  initialData,
}: QueryParams<ApiUser[], TReturn> = {}) {
  const { data, isLoading, isFetching, isPending, isError, error, refetch } = useQuery({
    ...usersQueryOptions,
    select,
    initialData,
  })

  return {
    data,
    isLoading,
    isFetching,
    isPending,
    isError,
    error,
    refetch,
  }
}
```

O hook React personalizado acima é uma boa estratégia para isolar queries. Ele expõe os métodos `select` e `initialData` e retorna apenas um subconjunto do objeto retornado pelo `useQuery`. Observe como o hook é tipado usando generics, permitindo que o tipo de dados retornado seja inferido a partir do retorno do método `select` que tem como padrão o tipo de retorno original, isso é extremamente importante para a segurança de tipos.

Uso do hook personalizado:

```tsx
// users.tsx
import { useUsers } from "@/hooks/use-users/use-users"
import { UserMapper } from "@/mappers/user-mapper/user-mapper"

const userMapper = UserMapper.create() // conjunto estável de métodos

export function Users() {
  const { data: users = [] } = useUsers() // data é inferido como um array de api users
  const { data: mappedUsers = [] } = useUsers({ select: userMapper.mapMany }) // data é inferido como um array de mapped users;
  const { data: usersCount = 0 } = useUsers({ select: userMapper.count }) // data é inferido como um número;

  return ...
}
```

Adicionalmente, é importante usar funções estáveis nos parâmetros `select` e `initialData` pois isso otimiza o processo de renderização. Funções estáveis podem ser funções criadas ou importadas de fora do contexto do React (hooks, componentes, etc...), ou funções que foram memoizadas pelo `useCallback`. Para mais informações veja:

[Otimizações de renderização](https://tanstack.com/query/latest/docs/framework/react/guides/render-optimizations)

## Query client

Além das coisas mencionadas acima, é importante definir valores padrão para opções importantes na instância do query client.

```ts
import { QueryClient } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false, // Defina isto como true apenas em queries específicas se necessário
      refetchOnWindowFocus: false,
      staleTime: 1000 * 60, // 1 minuto para cada query, a menos que seja sobrescrito pela própria query
    },
  }
});
```

## Linting

O Tanstack Query fornece uma ferramenta de linting para garantir as melhores práticas ao usá-lo. Para mais detalhes sobre como configurá-la, veja:

[Plugin ESLint query](https://tanstack.com/query/latest/docs/eslint/eslint-plugin-query)