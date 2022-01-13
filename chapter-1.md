# main.c
```C
#include "common.h"
#include "chunk.h"
#include "debug.h"

int main(int argc, char* argv[]) {
	Chunk chunk;
	initChunk(&chunk);
	int constant = addConstant(&chunk, 1.2);
	writeChunk(&chunk, OP_CONSTANT);
	writeChunk(&chunk, constant);

	writeChunk(&chunk, OP_RETURN);
	disassembleChunk(&chunk, "test chunk");
	freeChunk(&chunk);

	return 0;
}
```

# chunk.h

```C
#ifndef clox_chunk_h
  #define clox_chunk_h

  #include "common.h"
  #include "value.h"

  typedef enum {
    OP_CONSTANT,
    OP_RETURN,
  } OpCode;

  typedef struct {
    int count;
    int capacity;
    uint8_t* code;
    ValueArray constants;
  } Chunk;       

  void initChunk(Chunk* chunk);
  void freeChunk(Chunk* chunk);
  void writeChunk(Chunk* chunk, uint8_t byte);
  int addConstant(Chunk* chunk, Value value);
#endif
```

# common.h

```C
#ifndef clox_common_h
  #define clox_common_h
  #include <stdbool.h> // bool
  #include <stddef.h> // size_t
  #include <stdint.h> // int8_t, int16_t, int32_t, int64_t, uint8_t, uint16_t, uint32_t
#endif
```


# debug.h

```C
#ifndef clox_debug_h
  #define clox_debug_h

  #include "chunk.h"

  void disassembleChunk(Chunk* chunk, const char* name);
  int disassembleInstruction(Chunk* chunk, int offset);

#endif
```

# memory.h
```C
#ifndef clox_memory_h
  #define clox_memory_h

  #include "common.h"

  #define GROW_CAPACITY(capacity) ((capacity < 8) ? 8 : (capacity) * 2)

  #define GROW_ARRAY(type, pointer, oldCount, newCount) \
	(type*)reallocate(pointer, sizeof(type) * (oldCount), sizeof(type) * (newCount))

  #define FREE_ARRAY(type, pointer, oldCount) \
	reallocate(pointer, sizeof(type) * oldCount, 0);

  void* reallocate(void* pointer, size_t oldSize, size_t newSize);

#endif
```

# chunk.c
```C
#include <stdlib.h>
#include "chunk.h"
#include "memory.h"

void initChunk(Chunk* chunk) {
	chunk->count = 0;
	chunk->capacity = 0;
	chunk->code = NULL;
	initValueArray(&chunk->constants);
}

void freeChunk(Chunk* chunk) {
	FREE_ARRAY(uint8_t, chunk->code, chunk->capacity);
	freeValueArray(&chunk->constants);
	initChunk(chunk);
}

void writeChunk(Chunk* chunk, uint8_t byte) {
	if (chunk->capacity < chunk->count + 1) {
		int oldCapacity = chunk->capacity;
		chunk->capacity = GROW_CAPACITY(oldCapacity);
		chunk->code = GROW_ARRAY(uint8_t, chunk->code, oldCapacity, chunk->capacity);
	}

	chunk->code[chunk->count] = byte;
	chunk->count += 1;
}

int addConstant(Chunk* chunk, Value value) {
	writeValueArray(&chunk->constants, value);
	// despues de agregar una constante siempre devolvemos su índice
	// en la tabla de constantes.
	return chunk->constants.count - 1;
}
```

```C
#include <stdio.h>
#include "debug.h"
#include "value.h"

void disassembleChunk(Chunk* chunk, const char* name) {
	printf("== %s ==\n", name);

	for (int offset = 0; offset < chunk->count;) {
		offset = disassembleInstruction(chunk, offset);
	}
}

static int constantInstruction(const char* name, Chunk* chunk, int offset) {
	uint8_t constant = chunk->code[offset + 1];
	printf("%-16s %4d '", name, constant);
	printValue(chunk->constants.values[constant]);
	printf("'\n");
	return offset + 2; // se desplaza 2 espacios (1 por OP_CONSTANT y 2 por la constante)
}

int disassembleInstruction(Chunk* chunk, int offset) {
	printf("%04d ", offset);

	uint8_t instruction = chunk->code[offset];
	switch (instruction) {
	case OP_CONSTANT:
		return constantInstruction("OP_CONSTANT", chunk, offset);
	case OP_RETURN:
		return simpleInstruction("OP_RETURN", offset);
	default:
		printf("Unknown opcode %d\n", instruction);
		return offset + 1;
	}
}

static int simpleInstruction(const char* name, int offset) {
	printf("%s\n", name);
	return offset + 1;
}
```

```C
#include <stdlib.h>
#include "memory.h"

void* reallocate(void* pointer, size_t oldSize, size_t newSize) {
	if (newSize == 0) {
		free(pointer);
		return NULL;
	}

	void* result = realloc(pointer, newSize);
	if (result == NULL) {
		exit(EXIT_FAILURE);
	}
	return result;
}
```

# value.h
```C
#ifndef clox_value_h
 #define clox_value_h

 #include "common.h"

 typedef double Value;

 typedef struct {
	int capacity;
	int count;
	Value* values;
 } ValueArray;

 void initValueArray(ValueArray* array);
 void writeValueArray(ValueArray* array, Value value);
 void freeValueArray(ValueArray* array);
 void printValue(Value value);
#endif
```
