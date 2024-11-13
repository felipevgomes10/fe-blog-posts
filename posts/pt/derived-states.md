<!-- Entendendo estados derivados no React -->

<!-- 
    Mergulhe no poder do derived state no React - uma abordagem revolucionária para gerenciamento de estado. Este guia derruba conceitos errôneos comuns, apresentando exemplos práticos que transformarão sua forma de manipular dados em componentes. Perfeito para desenvolvedores cansados de re-renders desnecessários e complexidade no gerenciamento de estado.
 -->

 <!-- https://utfs.io/a/oqi3glmmqm/WfWc1HX19bacmhVS119geSuB3rftZEV8aJWIonQ0Clc2qhvs -->

## Entendendo estados derivados no React

Estado derivado no React representa valores calculados a partir de state ou props existentes. Embora seja simples em conceito, entender quando e como usar um estado derivado de forma eficaz pode melhorar significativamente a performance da aplicação e a manutenibilidade do código.

## O que é um estado derivado?

Estado derivado refere-se a valores que podem ser calculados inteiramente a partir de outras partes do state ou props. Em vez de armazenar esses valores calculáveis no state, eles são computados quando necessário.

## Quando usar um estado derivado:

1. Filtragem ou ordenação de listas:

```tsx
function ProductList({ products }) {
  // Estado derivado - produtos filtrados por disponibilidade
  const availableProducts = products.filter(product => product.inStock);
  
  // Estado derivado - ordenados por preço
  const sortedProducts = [...availableProducts].sort((a, b) => a.price - b.price);

  return (
    <ul>
      {sortedProducts.map(product => (
        <li key={product.id}>{product.name} - ${product.price}</li>
      ))}
    </ul>
  );
}
```

2. Calculando totais ou médias:

```tsx
function ShoppingCart({ items }) {
  // Estado derivado - preço total
  const totalPrice = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  
  // Estado derivado - preço médio por item
  const averagePrice = items.length > 0 ? totalPrice / items.length : 0;

  return (
    <div>
      <p>Total: ${totalPrice}</p>
      <p>Preço médio por item: ${averagePrice.toFixed(2)}</p>
    </div>
  );
}
```

3. Formatando dados para exibição:

```tsx
function UserProfile({ user }) {
  // Estado derivado - nome formatado
  const displayName = `${user.title} ${user.firstName} ${user.lastName}`;
  
  // Estado derivado - endereço formatado
  const fullAddress = `${user.street}, ${user.city}, ${user.country} ${user.zipCode}`;

  return (
    <div>
      <h2>{displayName}</h2>
      <p>{fullAddress}</p>
    </div>
  );
}
```

4. Combinando múltiplos valores de state:

```tsx
function SearchResults({ users, searchTerm, filterRole }) {
  // Estado derivado - combina critérios de busca e filtro
  const filteredUsers = users.filter(user => {
    const matchesSearch = user.name.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesRole = filterRole === 'all' || user.role === filterRole;
    return matchesSearch && matchesRole;
  });

  return (
    <ul>
      {filteredUsers.map(user => (
        <li key={user.id}>{user.name} - {user.role}</li>
      ))}
    </ul>
  );
}
```

## Anti-patterns a evitar:

1. Armazenando valores calculáveis no state:

```tsx
function ProductList({ products }) {
  // ❌ Má prática
  const [totalPrice, setTotalPrice] = useState(0);

  useEffect(() => {
    const sum = products.reduce((acc, product) => acc + product.price, 0);
    setTotalPrice(sum);
  }, [products]);

  // ✅ Melhor abordagem
  const totalPrice = products.reduce((acc, product) => acc + product.price, 0);

  return <div>Total: ${totalPrice}</div>;
}
```

2. Duplicando props no state:

```tsx
function UserCard({ user }) {
  // ❌ Má prática
  const [userName, setUserName] = useState(user.name);
  
  useEffect(() => {
    setUserName(user.name);
  }, [user.name]);

  // ✅ Melhor abordagem - usar a prop diretamente
  return <div>{user.name}</div>;
}
```

3. Atualizando valores derivados em useEffect:

```tsx
function FilteredList({ items, searchQuery }) {
  // ❌ Má prática
  const [filteredItems, setFilteredItems] = useState([]);

  useEffect(() => {
    const filtered = items.filter(item => 
      item.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
    setFilteredItems(filtered);
  }, [items, searchQuery]);

  // ✅ Melhor abordagem - computar durante o render
  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

## Padrões de implementação

A forma mais simples de estado derivado é um cálculo direto dentro do método render:

```tsx
function ProductList({ products }) {
  const totalPrice = products.reduce((sum, product) => sum + product.price, 0);
  
  return (
    <div>
      <p>Total: ${totalPrice}</p>
      <ul>{/* Renderização da lista de produtos */}</ul>
    </div>
  );
}
```

Quando os cálculos são custosos, use useMemo para memorizar resultados:

```tsx
function FilteredList({ items, filterCriteria }) {
  const filteredItems = useMemo(() => {
    return items.filter(item => {
      return item.category === filterCriteria;
    });
  }, [items, filterCriteria]);

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

Para gerenciamento complexo de state, considere usar padrões de selector:

```tsx
function UserDashboard({ user }) {
  // Função selector
  const getFullName = useCallback(() => {
    return `${user.firstName} ${user.lastName}`;
  }, [user.firstName, user.lastName]);

  return <h1>Bem-vindo, {getFullName()}</h1>;
}
```

## Considerações de performance

1. **Trade-offs de memoização:**
   - Memoize apenas cálculos custosos
   - Considere o custo de memória da memoização
   - Avalie cuidadosamente o conteúdo do array de dependências

2. **Timing de computação:**
   - Prefira computar valores durante o render
   - Evite atualizações desnecessárias do state
   - Considere usar web workers para computações pesadas

## Melhores práticas

1. **Mantenha simples:**
   - Comece sem memoização
   - Adicione otimização apenas quando necessário
   - Use ferramentas de profiling para identificar gargalos

2. **Mantenha funções puras:**
   - Garanta que os cálculos sejam determinísticos
   - Evite side effects em computações de estados derivados
   - Mantenha os cálculos autocontidos

3. **Documente dependências:**
   - Documentação clara dos requisitos de entrada
   - Arrays de dependência explícitos em hooks
   - Comentários explicando cálculos complexos

## Erros comuns

1. **Otimização excessiva:**

   ```tsx
   // Memoização desnecessária
   const capitalizedName = useMemo(() => {
     return name.toUpperCase();
   }, [name]);

   // Melhor abordagem
   const capitalizedName = name.toUpperCase();
   ```

2. **Dependências incorretas:**

   ```tsx
   // Dependência faltando
   const fullName = useMemo(() => {
     return `${firstName} ${lastName}`;
   }, [firstName]); // lastName faltando

   // Implementação correta
   const fullName = useMemo(() => {
     return `${firstName} ${lastName}`;
   }, [firstName, lastName]);
   ```

## Testando um estado derivado

```tsx
describe('ProductList', () => {
  it('calcula corretamente o preço total', () => {
    const products = [
      { id: 1, price: 10 },
      { id: 2, price: 20 }
    ];
    
    const { getByText } = render(<ProductList products={products} />);
    expect(getByText('Total: $30')).toBeInTheDocument();
  });
});
```

## Conclusão

Estado derivado é um conceito poderoso no React quando usado apropriadamente. Seguindo esses padrões e melhores práticas, desenvolvedores podem criar aplicações mais sustentáveis e performáticas. Lembre-se de medir os impactos na performance antes de adicionar complexidade através de memoização ou outras técnicas de otimização.

## Recursos adicionais

- Documentação React: [Hooks API Reference](https://react.dev/reference/react/hooks)
- React DevTools: [Profiler](https://react.dev/reference/react/Profiler)
- React DevTools: [Extension](https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)