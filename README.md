## Objetivo

Implemente um sistema de inventário em C que armazene múltiplos itens em um único bloco de memória alocado dinamicamente.

O inventário deve armazenar seus metadados e seus dados no mesmo bloco de memória, sem utilizar `struct` para representar o inventário.

A implementação deve permitir criar um inventário com capacidade fixa, adicionar itens, buscar itens pelo nome, remover itens e liberar toda a memória alocada.

## 1. Representação da memória

O inventário deve ser representado por um único bloco de memória, dividido em duas regiões:

* **Header:** armazena os metadados do inventário.
* **Data:** armazena os itens.

O header deve conter, nesta ordem:

1. Capacidade máxima de itens (`size_t`).
2. Quantidade atual de itens (`size_t`).
3. Tamanho máximo de cada item em bytes (`size_t`).
4. Nome do inventário (`char[32]`).

A região de dados deve começar imediatamente após o header.

Cada item deve ocupar exatamente 32 bytes, incluindo o terminador nulo (`'\0'`).

O layout esperado é:

```text
+----------------------------------+
| Capacity          (size_t)       |
+----------------------------------+
| Count             (size_t)       |
+----------------------------------+
| Element Size      (size_t)       |
+----------------------------------+
| Inventory Name    (char[32])     |
+----------------------------------+
| Item 0            (char[32])     |
+----------------------------------+
| Item 1            (char[32])     |
+----------------------------------+
| Item 2            (char[32])     |
+----------------------------------+
| ...                              |
+----------------------------------+
```

A função de criação deve retornar um ponteiro para o início da região de dados, e não para o início do bloco alocado.

Consequentemente, deve ser possível acessar os metadados retrocedendo o tamanho do header a partir desse ponteiro.

## 2. Funções obrigatórias

Implemente as seguintes funções:

```c
void *createInventory(
    const char *inventoryName,
    size_t inventorySize
);

int addToInventory(
    void *inventory,
    const char *itemName
);

void *getFromInventory(
    void *inventory,
    const char *itemName
);

void removeFromInventory(
    void *inventory,
    const char *itemName
);

void printInventory(void *inventory);

void deleteInventory(void *inventory);
```

### `createInventory()`

Cria um inventário com o nome e a capacidade informados.

Requisitos:

* Alocar memória para o header e todos os slots de dados em uma única operação.
* Inicializar os metadados.
* Inicializar a quantidade de itens com zero.
* Copiar o nome do inventário para o header.
* Retornar um ponteiro para o início da região de dados.
* Retornar `NULL` em caso de falha ou parâmetros inválidos.

### `addToInventory()`

Adiciona um item ao inventário.

Requisitos:

* Não permitir ultrapassar a capacidade máxima.
* Não permitir nomes que não caibam em um slot de 32 bytes, incluindo `'\0'`.
* Armazenar o item no próximo slot disponível.
* Atualizar a quantidade de itens somente quando a inserção for concluída.
* Retornar `1` em caso de sucesso e `0` em caso de falha.

### `getFromInventory()`

Busca um item pelo nome.

Requisitos:

* Percorrer apenas os slots ocupados.
* Comparar os nomes dos itens.
* Retornar um ponteiro para o item encontrado.
* Retornar `NULL` caso o item não exista.

A função não deve retornar uma cópia do item. Deve retornar um ponteiro para os dados armazenados no próprio inventário.

### `removeFromInventory()`

Remove um item identificado pelo nome.

Requisitos:

* Localizar o item utilizando `getFromInventory()`.
* Remover o item sem deixar lacunas entre os itens ocupados.
* Atualizar a quantidade de itens.
* Não alterar a capacidade máxima.
* Não liberar individualmente a memória do item.

**Regra:** para remover um item que não seja o último, mova o último item ocupado para a posição do item removido. Assim, a região ocupada permanece contígua.

Se o item não existir, o inventário deve permanecer inalterado.

### `printInventory()`

Imprime as informações do inventário:

* Nome.
* Capacidade.
* Quantidade atual de itens.
* Tamanho de cada slot.
* Lista dos itens ocupados.

A função deve funcionar corretamente quando o inventário estiver vazio.

### `deleteInventory()`

Libera o bloco de memória original utilizado pelo inventário.

Lembre-se: o ponteiro retornado por `createInventory()` aponta para a região de dados, não para o início da alocação.

## 3. Regras obrigatórias

1. A linguagem utilizada deve ser **C**.
2. É proibido utilizar `struct` para representar o inventário ou seu header.
3. O inventário deve utilizar uma única alocação dinâmica para armazenar header e dados.
4. Não utilize `realloc()` para redimensionar o inventário.
5. Não utilize uma alocação dinâmica individual para cada item.
6. Os metadados devem ser acessados por meio de aritmética de ponteiros.
7. Os itens devem ser armazenados em posições contíguas de memória.
8. A busca deve ser feita pelo nome do item.
9. Não ultrapasse os limites de memória alocados.
10. Valide os parâmetros e trate falhas de alocação.
11. Evite casts desnecessários e preserve a correção dos tipos de ponteiro.
12. Trate possíveis overflows ao calcular o tamanho total da alocação.
13. A remoção de um item não pode invalidar os metadados nem alterar a capacidade do inventário.

## 4. Exemplo de utilização

```c
int main(void) {
    void *inventory =
        createInventory("potato-inventory", 8);

    if (!inventory)
        return 1;

    addToInventory(inventory, "potato-1");
    addToInventory(inventory, "potato-2");
    addToInventory(inventory, "potato-3");

    printInventory(inventory);

    char *item = getFromInventory(
        inventory,
        "potato-2"
    );

    if (item)
        printf("Found: %s\n", item);

    removeFromInventory(inventory, "potato-2");

    printInventory(inventory);

    deleteInventory(inventory);

    return 0;
}
```

## 5. Resultado esperado

Antes da remoção:

```text
Inventory Name: potato-inventory
capacity: 8
count: 3
element size: 32

items:
potato-1
potato-2
potato-3

Found: potato-2
```

Após remover `potato-2`:

```text
Inventory Name: potato-inventory
capacity: 8
count: 2
element size: 32

items:
potato-1
potato-3
```

A ordem dos itens pode mudar durante a remoção, desde que não existam lacunas entre os slots ocupados.

## 6. Casos que devem ser testados

* Criar um inventário com capacidade zero.
* Adicionar itens até atingir a capacidade máxima.
* Tentar inserir um item quando o inventário estiver cheio.
* Tentar inserir um nome que exceda o tamanho permitido.
* Buscar um item existente e outro inexistente.
* Remover o primeiro, um item intermediário e o último.
* Remover um item inexistente.
* Imprimir um inventário vazio.
* Excluir o inventário após utilizá-lo.

**Objetivo final:** demonstrar domínio sobre alocação dinâmica, layout de memória, aritmética de ponteiros, representação de metadados, strings em C e gerenciamento manual de memória.
