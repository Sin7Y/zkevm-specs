# Tables

For the zkevm we use the following dynamic and fixed tables for lookups to the EVM circuit.  The validity of the dynamic tables contents is proved by their own associated circuit.

Code spec at [table.py](../src/zkevm_specs/evm/table.py)

## `tx_table`

Proved by the tx circuit.

| 0 TxID | 1 Tag               | 2          | 3 value |
| ---    | ---                 | ---        | ---     |
|        | *TxContextFieldTag* |            |         |
| $TxID  | Nonce               | 0          | $value  |
| $TxID  | Gas                 | 0          | $value  |
| $TxID  | GasPrice            | 0          | $value  |
| $TxID  | GasTipCap           | 0          | $value  |
| $TxID  | GasFeeCap           | 0          | $value  |
| $TxID  | CallerAddress       | 0          | $value  |
| $TxID  | CalleeAddress       | 0          | $value  |
| $TxID  | IsCreate            | 0          | $value  |
| $TxID  | Value               | 0          | $value  |
| $TxID  | CallDataLength      | 0          | $value  |
| $TxID  | CallData            | $ByteIndex | $value  |

## `rw_table`

Proved by the state circuit.

> - **CallContext constant**: read-only data in a call, usually checked with the
>   caller before the beginning of a call.
> - **CallContext state**: used by caller to save its own CallState when it's going
>   to dive into another call, and will be read out to restore caller's
>   CallState in the end by callee.
> - **CallContext last callee**: read-only data inside a call like previous section
>   for opcode `RETURNDATASIZE` and `RETURNDATACOPY`, except they will be
>   updated when end of callee execution.

Type sizes:

> - **txID**: 24 bytes
> - **address**: 160 bytes
> - **callID**: 25 bytes
> - **stackPointer**: 10 bytes
> - **memoryAddress**: 40 bytes
> - **Memory -> value, valuePrev**: 1 byte
> - **storageKey**: field size, RLC encoded (Random Linear Combination)
> - **value, valuePrev**: variable size, depending on *Key 1 (Tag)* and *Key 3*
>   where appropiate.

| 0 *rwc*  | 1 *isWrite* | 2 *Key 1 (Tag)*            | 3 *Key 2* | 4 *Key 3*                  | 5 *Key 4*   | 6 *Value 1* | 7 *Value 2* | 8 *Aux 1* | 9 *Aux 2*       |
| ---      | ---         | ---                        | ---       | ---                        | ---         | ---         | ---         | ---       | ---             |
|          |             | *RwTableTag*               |           |                            |             |             |             |           |                 |
| $counter | true        | TxAccessListAccount        | $txID     | $address                   | 0           | $value      | $valuePrev  | 0         | 0               |
| $counter | true        | TxAccessListAccountStorage | $txID     | $address                   | $storageKey | $value      | $valuePrev  | 0         | 0               |
| $counter | $isWrite    | TxRefund                   | $txID     | 0                          | 0           | $value      | $valuePrev  | 0         | 0               |
|          |             |                            |           |                            |             |             |             |           |                 |
|          |             |                            |           | *AccountFieldTag*          |             |             |             |           |                 |
| $counter | $isWrite    | Account                    | $address  | Nonce                      | 0           | $value      | $valuePrev  | 0         | 0               |
| $counter | $isWrite    | Account                    | $address  | Balance                    | 0           | $value      | $valuePrev  | 0         | 0               |
| $counter | $isWrite    | Account                    | $address  | CodeHash                   | 0           | $value      | $valuePrev  | 0         | 0               |
| $counter | true        | AccountDestructed          | $address  | 0                          | 0           | $value      | $valuePrev  | 0         | 0               |
|          |             |                            |           |                            |             |             |             |           |                 |
|          |             | *CallContext constant*     |           | *CallContextFieldTag* (ro) |             |             |             |           |                 |
| $counter | false       | CallContext                | $callID   | RwCounterEndOfReversion    | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | CallerId                   | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | TxId                       | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | Depth                      | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | CallerAddress              | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | CalleeAddress              | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | CallDataOffset             | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | CallDataLength             | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | ReturnDataOffset           | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | ReturnDataLength           | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | Value                      | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | Result                     | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | IsPersistent               | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | IsStatic                   | 0           | $value      | 0           | 0         | 0               |
|          |             |                            |           |                            |             |             |             |           |                 |
|          |             | *CallContext last callee*  |           | *CallContextFieldTag* (rw) |             |             |             |           |                 |
| $counter | false       | CallContext                | $callID   | LastCalleeId               | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | LastCalleeReturnDataOffset | 0           | $value      | 0           | 0         | 0               |
| $counter | false       | CallContext                | $callID   | LastCalleeReturnDataLength | 0           | $value      | 0           | 0         | 0               |
|          |             |                            |           |                            |             |             |             |           |                 |
|          |             | *CallContext state*        |           | *CallContextFieldTag* (rw) |             |             |             |           |                 |
| $counter | $isWrite    | CallContext                | $callID   | IsRoot                     | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | IsCreate                   | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | CodeSource                 | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | ProgramCounter             | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | StackPointer               | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | GasLeft                    | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | MemorySize                 | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | CallContext                | $callID   | StateWriteCounter          | 0           | $value      | 0           | 0         | 0               |
|          |             |                            |           |                            |             |             |             |           |                 |
| $counter | $isWrite    | Stack                      | $callID   | $stackPointer              | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | Memory                     | $callID   | $memoryAddress             | 0           | $value      | 0           | 0         | 0               |
| $counter | $isWrite    | AccountStorage             | $address  | 0                          | $storageKey | $value      | $valuePrev  | $txID     | $CommittedValue |

## `bytecode_table`

Proved by the bytecode circuit.

> - **isCode**: A boolean value to specify if the value is executable opcode or
>   the data portion of PUSH\* operations.

| 0 codeHash | 1 byteIndex | 2 value | 3 isCode |
| ---        | ---         | ---     | ---      |
| $codeHash  | $byteIndex  | $value  | $isCode  |

## `block_table`

Proved by the block circuit.

| 0 Tag                  | 1      | 2 value |
| ---                    | ---    | ---     |
| *BlockContextFieldTag* |        |         |
| Coinbase               | 0      | $value  |
| GasLimit               | 0      | $value  |
| BlockNumber            | 0      | $value  |
| Time                   | 0      | $value  |
| Difficulty             | 0      | $value  |
| BaseFee                | 0      | $value  |
| BlockHash              | 0..256 | $value  |

## `fixed`

> - **execution_state.responsible_opcode()**: map execution_state (opcode's
>   successful cases, where multiple similar opcodes may be merged into a
>   single execution state, like `LT`, `GT`, `EQ` in `CMP` state) to opcode
>   that can generate that execution state.
> - **invalid_opcodes()**: set of invalid opcodes
> - **state_write_opcodes()**: set of opcodes that write the state.
> - **stack_underflow_pairs**: set of opcodes and stack pointer value that
>   causes underflow during execution.
> - **stack_overflow_pairs**: set of opcodes and stack pointer value that
>   causes overflow during execution.

| 0 Tag             | 1                     | 2                                         | 3             |
| ---               | ---                   | ---                                       | ---           |
| *FixedTableTag*   |                       |                                           |               |
| Range16           | 0..16                 | 0                                         | 0             |
| Range32           | 0..32                 | 0                                         | 0             |
| Range64           | 0..64                 | 0                                         | 0             |
| Range256          | 0..256                | 0                                         | 0             |
| Range512          | 0..512                | 0                                         | 0             |
| Range1024         | 0..1024               | 0                                         | 0             |
| SignByte          | value=0..256          | if (value as i8 \< 0) 0xff else 0         | 0             |
| BitwiseAnd        | lhs=0..256            | rhs=0..256                                | $lhs AND $rhs |
| BitwiseOr         | lhs=0..256            | rhs=0..256                                | $lhs OR $rhs  |
| BitwiseXor        | lhs=0..256            | rhs=0..256                                | $lhs XOR $rhs |
| ResponsibleOpcode | $execution_state      | execution_state.responsible_opcode()      | 0             |
| InvalidOpcode     | invalid_opcodes()     | 0                                         | 0             |
| StateWriteOpcode  | state_write_opcodes() | 0                                         | 0             |
| StackOverflow     | $overflow_opcode      | stack_overflow_pairs\[overflow_opcode\]   | 0             |
| StackUnderflow    | $underflow_opcode     | stack_underflow_pairs\[underflow_opcode\] | 0             |
