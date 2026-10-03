# Inventory-Memory-Manager

Implemente em **C** um sistema de inventário que armazene múltiplos itens em **um único bloco de memória**, sem `struct`, `realloc()` ou alocações individuais por item.

## Layout

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

O header deve seguir essa ordem. Cada item ocupa exatamente **32 bytes**, incluindo `'\0'`.

`createInventory()` deve retornar o início da região de dados. Os metadados devem ser acessados por aritmética de ponteiros, retrocedendo `HEADER_SIZE` bytes.

## Funções

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

Adicionar no próximo slot disponível. Não ultrapassar a capacidade nem aceitar nomes que não caibam em 32 bytes, incluindo `'\0'`. Atualizar `count` somente após sucesso. Retornar `1` ou `0`.

### `getFromInventory()`

Buscar pelo nome percorrendo somente os slots ocupados. Retornar um ponteiro para o item armazenado ou `NULL`. Não criar cópia.

### `removeFromInventory()`

Localizar usando `getFromInventory()`. Para evitar lacunas, mover o último item ocupado para a posição removida. Decrementar `count`. Não alterar `capacity` nem liberar memória individualmente. Se não encontrado, não modificar o inventário.

### `printInventory()`

Imprimir nome, capacidade, quantidade, tamanho do slot e os itens ocupados. Deve funcionar com inventário vazio.

### `deleteInventory()`

Liberar o bloco original completo. O ponteiro recebido aponta para os dados, não para o início da alocação.

## Regras

* Linguagem: **C**.
* Sem `struct` para inventário/header.
* Uma única alocação dinâmica.
* Sem `realloc()`.
* Sem alocações individuais por item.
* Usar aritmética de ponteiros para acessar os metadados.
* Itens devem permanecer contíguos.
* Não acessar memória fora do bloco.
* Validar parâmetros e falhas de alocação.
* Tratar overflow no tamanho da alocação.
* Remoções não podem alterar a capacidade.

## Testes

Teste:

* Capacidade `0` e inventário vazio.
* Inserção até a capacidade e tentativa além dela.
* Nome maior que 31 caracteres.
* Busca existente e inexistente.
* Remoção do primeiro, intermediário, último e inexistente.
* Exclusão do inventário.

**Objetivo:** demonstrar domínio de alocação dinâmica, layout de memória, aritmética de ponteiros, strings e gerenciamento manual de memória em C.
