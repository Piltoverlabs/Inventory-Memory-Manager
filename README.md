# Inventory-Memory-Manager

Implemente em **C** um sistema de inventário que armazene múltiplos itens em **um único bloco de memória**, sem utilizar `struct`, `realloc()` ou alocações individuais para os itens.

## 1. Layout

O bloco deve conter:

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

O header deve seguir exatamente essa ordem. Cada item ocupa **32 bytes**, incluindo `'\0'`.

`createInventory()` deve retornar um ponteiro para o início da região de dados. Os metadados devem ser acessados por aritmética de ponteiros, retrocedendo `HEADER_SIZE` bytes.

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

Alocar header + todos os slots em uma única operação. Inicializar `capacity`, `count = 0`, `elementSize = 32` e `name`. Validar parâmetros e overflow. Retornar `NULL` em caso de erro.

### `addToInventory()`

Adicionar no próximo slot disponível. Não ultrapassar a capacidade e não aceitar nomes que não caibam em 32 bytes, incluindo `'\0'`. Atualizar `count` somente após sucesso. Retornar `1` ou `0`.

### `getFromInventory()`

Percorrer somente os slots ocupados e buscar pelo nome. Retornar um ponteiro para o item armazenado ou `NULL` se não encontrado. Não retornar cópias.

### `removeFromInventory()`

Localizar o item usando `getFromInventory()`. Para manter a região ocupada contígua, mover o último item para a posição removida quando necessário. Decrementar `count`, sem alterar `capacity` ou liberar memória individualmente. Se o item não existir, não modificar o inventário.

### `printInventory()`

Imprimir:

```text
Inventory Name: ...
capacity: ...
count: ...
element size: 32

items:
...
```

Deve funcionar com inventário vazio.

### `deleteInventory()`

Liberar o bloco original completo. Lembre-se de que `inventory` aponta para a região de dados, não para o início da alocação.

## 3. Regras

* Linguagem: **C**.
* Proibido `struct` para representar o inventário/header.
* Uma única alocação dinâmica.
* Proibido `realloc()`.
* Nenhuma alocação por item.
* Usar aritmética de ponteiros para os metadados.
* Manter os itens em posições contíguas.
* Não acessar memória fora do bloco.
* Validar parâmetros e falhas de alocação.
* Tratar overflow no cálculo do tamanho total.
* Remoções não podem alterar a capacidade.

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

Após a remoção:

```text
Inventory Name: potato-inventory
capacity: 8
count: 2
element size: 32

items:
potato-1
potato-3
```

## 5. Testes

Teste:

* Capacidade `0` e inventário vazio.
* Inserção até a capacidade e tentativa além dela.
* Nome maior que 31 caracteres.
* Busca existente e inexistente.
* Remoção do primeiro, intermediário, último e inexistente.
* Exclusão do inventário.

**Objetivo:** demonstrar domínio de alocação dinâmica, layout de memória, aritmética de ponteiros, strings e gerenciamento manual de memória em C.
