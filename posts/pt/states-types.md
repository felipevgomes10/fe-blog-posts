<!-- O Guia Completo de State Management no React -->

<!-- 
    Este guia examina as principais abordagens de state management no React, desde state local até soluções complexas de gerenciamento de estado. Abrange padrões essenciais incluindo useState, useReducer, Context API e bibliotecas externas, fornecendo exemplos práticos e boas práticas para escolher soluções apropriadas com base nas necessidades da aplicação. O guia enfatiza começar com abordagens simples e aumentar a complexidade do gerenciamento de estado apenas quando necessário para manter a clareza e performance do código.
 -->

 <!-- https://utfs.io/a/oqi3glmmqm/WfWc1HX19bacd9juZ59iczvg8VTYL47NZP90BEMyf2X3Sarp -->

## O Guia Completo de State Management no React

O state management é um dos conceitos fundamentais no React que todo desenvolvedor precisa dominar. À medida que as aplicações se tornam mais complexas, entender os diferentes tipos de state e seus casos de uso apropriados torna-se crucial para construir aplicações sustentáveis e performáticas. Este guia explora as várias abordagens de state management no React, desde o state local simples até soluções externas complexas.

## Entendendo o State Local

O state local representa a forma mais básica de state management no React, tipicamente gerenciado através do hook `useState`. Ao construir componentes que precisam lidar com seus próprios dados de forma independente, o state local oferece uma solução direta. Inputs de formulário servem como um exemplo perfeito - quando um usuário digita em um campo de texto, esse valor de input só precisa existir dentro do próprio componente do formulário. Da mesma forma, o status aberto/fechado de um modal ou a visibilidade de um indicador de carregamento frequentemente só importam para o componente que os exibe.

```tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount((count) => count + 1)}>
      Contador: {count}
    </button>
  );
}
```

O state local se destaca em cenários onde os dados não precisam ser compartilhados entre componentes. Ele mantém a lógica do seu componente encapsulada e torna o código mais fácil de entender e manter. Considere usar state local quando seu componente precisa rastrear valores simples que não afetam outras partes da sua aplicação.

## Gerenciando States Complexos com Reducers

À medida que a lógica de states se torna mais complexa, gerenciar múltiplas atualizações de states relacionadas com useState pode se tornar difícil de manejar. O hook `useReducer` fornece uma abordagem mais estruturada, particularmente quando as atualizações de state dependem de múltiplos fatores ou quando uma ação deve disparar várias mudanças de state. Este padrão parecerá familiar para desenvolvedores que já trabalharam com Redux, pois segue princípios similares.

```tsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    default:
      return state;
  }
}

function TodoList() {
  const [todos, dispatch] = useReducer(todoReducer, []);
  
  const addTodo = (text) => {
    dispatch({
      type: 'ADD_TODO',
      payload: { id: Date.now(), text, completed: false }
    });
  };
}
```

O padrão reducer se destaca quando você precisa manter transições de state previsíveis e quando sua lógica de state precisa ser compartilhada entre múltiplos componentes. Ele fornece um local centralizado para a lógica de mutação de state, tornando mais fácil depurar e testar sua aplicação.

## Compartilhando States Entre Componentes

O compartilhamento de state entre componentes pode ser abordado de duas maneiras principais: elevando o state e usando context. Quando componentes precisam compartilhar state, a primeira abordagem envolve mover esse state para o ancestral comum mais próximo. Este padrão, conhecido como "lifting state up", funciona bem para componentes intimamente relacionados que precisam permanecer sincronizados.

```tsx
function Parent() {
  const [shared, setShared] = useState('');
  
  return (
    <>
      <ChildA shared={shared} setShared={setShared} />
      <ChildB shared={shared} />
    </>
  );
}
```

Para states que precisam ser acessados por muitos componentes em diferentes partes da sua aplicação, a Context API do React fornece uma solução mais elegante. O Context permite que você evite o "prop drilling" - o processo de passar props através de componentes intermediários que não precisam deles.

```tsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

## Gerenciando State Assíncrono

Lidar com operações assíncronas introduz desafios únicos no state management. Ao buscar dados de uma API, fazer upload de arquivos ou gerenciar conexões WebSocket, você precisa rastrear múltiplos valores de state simultaneamente: os próprios dados, status de carregamento e potenciais erros. Esta complexidade requer uma consideração cuidadosa da estrutura do state e tratamento de erros.

```tsx
function UserProfile() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch('/api/user');
        const userData = await response.json();
        setData(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, []);

  if (loading) return <div>Carregando...</div>;
  if (error) return <div>Erro: {error}</div>;
  if (!data) return null;

  return <div>{data.name}</div>;
}
```

## Gerenciamento de State na URL

O state na URL desempenha um papel crucial em aplicações web ao manter o state da aplicação dentro da própria URL. Esta abordagem permite que os usuários marquem estados específicos da aplicação e compartilhem links que reproduzam exatamente as mesmas visualizações de páginas. O state na URL prova ser particularmente valioso para recursos como paginação, parâmetros de busca e configurações de filtro.

```tsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const page = searchParams.get('page') || 1;
  const category = searchParams.get('category') || 'all';

  const updateFilters = (newCategory) => {
    setSearchParams({
      page: 1,
      category: newCategory
    });
  };

  return (
    <div>
      <select 
        value={category} 
        onChange={(e) => updateFilters(e.target.value)}
      >
        <option value="all">Todos</option>
        <option value="electronics">Eletrônicos</option>
        <option value="books">Livros</option>
      </select>
    </div>
  );
}
```

## State de Referência com useRef

Embora tecnicamente não seja "state" no React, refs fornecem uma maneira de manter valores mutáveis entre renderizações sem disparar re-renderizações. Isso os torna ideais para armazenar referências do DOM, gerenciar intervalos e timeouts, e rastrear valores que não devem causar atualizações de componentes.

```tsx
function StopWatch() {
  const intervalRef = useRef(null);
  const [time, setTime] = useState(0);

  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };

  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };
}
```

## Melhores Práticas de State Management

O state management efetivo começa com o escopo adequado. Mantenha o state o mais próximo possível de onde ele é necessário, movendo-o para cima na árvore de componentes apenas quando necessário. Mantenha uma única fonte de verdade para seus dados, evitando duplicação de state que pode levar a problemas de sincronização.

Considere as implicações de performance das suas escolhas de state management. Nem todo dado precisa estar no state - valores derivados podem frequentemente ser calculados em tempo real. Ao implementar operações assíncronas, trate os estados de carregamento e erro de forma elegante para fornecer uma experiência de usuário suave.

O state na URL deve ser aproveitado para estados da aplicação compartilháveis, enquanto o state local é suficiente para elementos temporários da UI. Implemente error boundaries para lidar graciosamente com erros relacionados ao state e prevenir crashes da aplicação.

## Soluções Externas de State Management

À medida que as aplicações crescem, bibliotecas externas de state management podem fornecer estrutura e capacidades adicionais. Redux oferece uma solução robusta para aplicações em larga escala com interações complexas de state, enquanto Zustand fornece uma alternativa mais leve com uma API mais simples. Jotai e Recoil se destacam no state management atômico, quebrando o state em peças menores e gerenciáveis.

Para aplicações fortemente focadas em state assíncrono, o TanStack Query (anteriormente React Query) oferece ferramentas especializadas para lidar com dados assíncronos, caching e sincronização com serviços backend. Cada solução tem seus pontos fortes, e escolher a certa depende das necessidades específicas da sua aplicação.

## Conclusão

Aplicações React modernas frequentemente requerem uma combinação de abordagens de state management. Entender os pontos fortes e casos de uso apropriados para cada tipo de state permite que você construa aplicações mais sustentáveis e performáticas. Comece com a solução mais simples que atenda às suas necessidades e escale sua abordagem de state management conforme sua aplicação cresce.

Diferentes partes da sua aplicação podem se beneficiar de diferentes abordagens de state management - não existe uma solução única que sirva para todos os casos. Foque em escolher a ferramenta certa para cada requisito específico enquanto mantém a clareza do código e a performance. À medida que o ecossistema React continua evoluindo, mantenha-se informado sobre novos padrões e melhores práticas para garantir que sua estratégia de state management permaneça efetiva.

_Siga a documentação oficial do React e mantenha-se atualizado com as melhores práticas conforme o ecossistema evolui._

## Leituras Adicionais

- [Padrões de Data Fetching no React](https://fe-blog.techgg.app/pt/posts/data-fetching-patterns)
- [Entendendo Derived State no React](https://fe-blog.techgg.app/pt/posts/derived-states)

## Recursos Adicionais

1. React state management:
	- [State: A Memória de um Componente](https://react.dev/learn/state-a-components-memory)
	- [useState](https://react.dev/reference/react/useState)
	- [useReducer](https://react.dev/reference/react/useReducer)
	- [useRef](https://react.dev/reference/react/useRef)
	
2. State na URL: 
	- [useSearchParams](https://reactrouter.com/en/main/hooks/use-search-params)

3. Bibliotecas de state management: 
	- [Redux](https://redux.js.org/)
	- [Zustand](https://zustand.docs.pmnd.rs/getting-started/introduction)
	- [Jotai](https://jotai.org/)
	- [Recoil](https://recoiljs.org/)

3. Bibliotecas de state management assíncrono:
	- [Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview)
	- [SWR](https://swr.vercel.app/pt-BR)