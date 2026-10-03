# Inventory-Memory-Manager

Implemente em **C** um sistema de inventário que armazene múltiplos itens em **um único bloco de memória alocado dinamicamente**.

Não utilize `struct` para representar o inventário. Metadados e dados devem permanecer no mesmo bloco.

## 1. Layout de memória

O bloco deve possuir:

```text
+-----------------------------+
| Capacity      (size_t)      |
| Count         (size_t)      |
| Element Size  (size_t)      |
| Name          (char[32])    |
+-----------------------------+
| Item 0        (char[32])    |
| Item 1        (char[32])    |
| Item 2        (char[32])    |
| ...                         |
+-----------------------------+
```

O header contém, nesta ordem:

1. `size_t capacity`
2. `size_t count`
3. `size_t elementSize`
4. `char name[32]`

Cada item ocupa exatamente **32 bytes**, incluindo `'\0'`.

`createInventory()` deve retornar um ponteiro para o **início da região de dados**, e não para o início da alocação. Os metadados devem ser acessados retrocedendo `HEADER_SIZE` bytes.

## 2. Funções

```c
void *createInventory(const char *inventoryName, size_t inventorySize);

int addToInventory(void *inventory, const char *itemName);

void *getFromInventory(void *inventory, const char *itemName);

void removeFromInventory(void *inventory, const char *itemName);

void printInventory(void *inventory);

void deleteInventory(void *inventory);
```

### `createInventory()`

* Alocar header + todos os slots em **uma única operação**.
* Validar parâmetros e possíveis overflows.
* Inicializar `capacity`, `count`, `elementSize` e `name`.
* `count` deve iniciar em `0`.
* Retornar o início da região de dados.
* Retornar `NULL` em caso de erro.

### `addToInventory()`

* Inserir no próximo slot disponível.
* Não ultrapassar `capacity`.
* O nome deve caber nos 32 bytes, incluindo `'\0'`.
* Atualizar `count` somente após a inserção.
* Retornar `1` em sucesso e `0` em falha.

### `getFromInventory()`

* Percorrer somente os slots ocupados.
* Comparar os nomes.
* Retornar um ponteiro para o item armazenado.
* Retornar `NULL` se não encontrado.
* Não criar uma cópia do item.

### `removeFromInventory()`

* Localizar o item usando `getFromInventory()`.
* Manter os itens ocupados contíguos.
* Se o item removido não for o último, mover o último item para sua posição.
* Decrementar `count`.
* Não alterar `capacity`.
* Não liberar memória individualmente.
* Se o item não existir, não alterar o inventário.

### `printInventory()`

Imprimir:

```text
Inventory Name: ...
capacity: ...
count: ...
element size: ...

items:
...
```

Deve funcionar também com inventário vazio.

### `deleteInventory()`

Liberar o **bloco original completo**, considerando que o ponteiro recebido aponta para a região de dados.

## 3. Regras

* Linguagem: **C**
* Proibido `struct` para inventário/header.
* Uma única alocação dinâmica.
* Proibido `realloc()`.
* Nenhuma alocação individual por item.
* Usar aritmética de ponteiros para acessar os metadados.
* Itens devem permanecer em posições contíguas.
* Não ultrapassar os limites alocados.
* Validar parâmetros e falhas de `malloc()`.
* Tratar overflow no cálculo do tamanho da alocação.
* Preservar a capacidade durante remoções.

## 4. Exemplo

```c
int main(void) {
    void *inventory = createInventory("potato-inventory", 8);

    if (!inventory)
        return 1;

    addToInventory(inventory, "potato-1");
    addToInventory(inventory, "potato-2");
    addToInventory(inventory, "potato-3");

    printInventory(inventory);

    char *item = getFromInventory(inventory, "potato-2");

    if (item)
        printf("Found: %s\n", item);

    removeFromInventory(inventory, "potato-2");

    printInventory(inventory);

    deleteInventory(inventory);
}
```

Resultado após a remoção:

```text
Inventory Name: potato-inventory
capacity: 8
count: 2
element size: 32

items:
potato-1
potato-3
```

## 5. Testes obrigatórios

Teste pelo menos:

* Capacidade `0`.
* Inventário vazio.
* Inserção até a capacidade máxima.
* Inserção com inventário cheio.
* Nome maior que 31 caracteres.
* Busca existente e inexistente.
* Remoção do primeiro item.
* Remoção de item intermediário.
* Remoção do último item.
* Remoção de item inexistente.
* Exclusão do inventário.

**Objetivo:** demonstrar domínio de alocação dinâmica, layout de memória, aritmética de ponteiros, strings e gerenciamento manual de memória em C.
