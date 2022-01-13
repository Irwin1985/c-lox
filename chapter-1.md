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
