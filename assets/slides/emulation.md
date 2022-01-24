# Emulation

What are we doing? Implementing functionality of one interface on a subsystem with a different one

Definition: "Architected" - programmer visible state or interface for a machine

The emulation software (emulator) maintains:
1. Memory (program code, data, stack, etc.)
2. Emulated machine state (registers, PC, condition codes, etc.)


Example state:

```
struct machine_state {
    uint32_t pc;
    uint8_t * ram;
    uint32_t regs[NUM_REGS];
    uint32_t ctrlA;
    uint32_t ctrlB;
    ...
};
```

For a real ISA, this can get quite large!

## Decode Dispatch

Typical emulation model is "Decode-and-dispatch"

More or less:

```
while (1) {
    // fetch 
    instr = read_mem(pc)

    // decode
    op = decode(instr)

    // dispatch (execute)
    switch (op) {
      case INSTR_A:
         // ...
      case INSTR_B:
         // ...
      default:
        bad instr
    }
}
```

Decoding might actually occur in two stages. Suppose we have an instruction encoding in our ISA like so

```
|---------------------------|
| INC  | ...
| 001  | ...
|---------------------------|
```

```
|------------------------------|
| INCA | IMM_FLAG |  SIGNED_IMM
| 001  | 1        |
|------------------------------|
```

Here the opcode is the same, but depending on the flag in bit 3, we might be doing an increment by one, or
an increment by an amount specified by a signed immediate operand. For this case, we might have something like this

```
while (1) {
    // fetch 
    instr = read_mem(pc)

    // decode
    op = decode(instr)

    // dispatch (execute)
    switch (op) {
      case INC
        if (instr >> 3 & 0x1) {
           // emulate INCA
        } else {
           // emulate INC
        } 

      default:
        bad instr
    }
}
```


## Threaded Interpretation

Decode-and-dispatch can be slow. Why? 
* conditional control flow from the switch
* branching from the loop

These days this is somewhat less of an issue: branch prediction is pretty good!

We can "thread" together instructions instead, though:

```
op_table = {
    &handle_add, 
    &handle_sub,
    ...
}

fun handle_add() {
    regs[a] = regs[b] + regs[c]
    op_table[read_mem(pc)]() // thread the needle!
    
} 

fun handle_sub() {
    regs[a] = regs[b] - regs[c]
    op_table[read_mem(pc)]() // thread the needle!
}

emulate() {
    // fetch
    first_instr = read_mem(pc)

    // decode
    op = decode(instr)

    op_table[op]()
}
```

Downside? More icache usage (replicated code). Still have a lookup table!


## Predecoding

We can use an intermediate representation to speed things up a bit if we have a 
complex instruction set. Why? think loops: we decode same instruction more than once!

We can do something like this:
```
struct instruction {
    uint8_t opcode
    uint8_t oper_a;
    uint32_t immediate;
    uint8_t flagA : 1;
    ...
} 
```

Then we can have our actual code in RAM, and we can _pre-decode_ it into an array of these instructions:

```
struct instruction predecoded_instrs[MAX];

fun pre_decode() {
    for (i = 0; i < PREDECODE_MAX; i++) {
        instr = read_mem(get_pc()); 
        predecoded_instrs[i] = decode(instr);
    } 
}
```

We can then walk through these instructions one by one and save on decoding costs. 
For control flow we have to be careful about PC! In fact, we need two PCs. 

We can be more clever and stash a pointer to the interpretation routine for a particular
instruction in the instruction itself:

```
struct instruction {
    void (*handler)();
    uint8_t opcode
    uint8_t oper_a;
    uint32_t immediate;
    uint8_t flagA : 1;
    ...
} 
```

This is **direct threaded interpretation**. 


