
## Contents

1. Design abstraction
2. Analyze a instruction
3. Construct a datapath
4. Construct a control unit
5. Performance
6. Multi-cycle processor
7. Reference

## Analyze a instruction

建構三種指令：``R-type, I-type, J-type``

![](https://hackmd.io/_uploads/HknDmNnKh.jpg)

1. 名詞解釋
    - op: operation，決定指令的操作種類
    - rs, rt, rd：source, target, destination register 位址
    - shamt：shift amount
    - funct：補足R-type之op無法描述完的操作
2. logical reg transfer
    - R-type: ``R[rd] = R[rs] + R[rt] ; PC=PC+4``
    - lw: ``R[rt] = mem[rs+sign_extend(imm)] ; PC=PC+4``
    - sw: ``mem[rs+sign_extend(imm)] = R[rt]``
    - beq: ``if(R[rs]==R[rt]) PC=(PC+4)+sign_extend(imm)``
3. Datapath中需要描述的功能
    - memory
        - 儲存指令及資料
    - register(32*32)
        - read rs
        - read rt
        - write rt, rd
    - PC
    - sign extension
    - ALU
    - ADD 4, immidiate


## Construct a datapath

### R-type

##### ``add $t0, $t1, $t2``

##### 皆在暫存器內完成

#### 目標： *register file*：``rs = rt + rd``，經由*ALU*運算。


#### 流程：
1. **RegFile**讀**rs**, **rt**位址進**readReg1**, **readReg2**，並由**readData1**, **readData2**，送資料進**ALU**
2. **Control**讀取OP code及funct，控制ALUcont，完成計算
3. ALU計算結果送進**RegFile**中的**writeData**
4. **RegFile**讀取**rd**暫存器
5. **Control**控制regWrite完成寫入至**rd**

### Load word(I-type)

##### ``lw $s1, 12($s2)``
##### load進暫存器，因此是在register file進行寫入

#### 目標： immediate經過sign-extension後和``rs``位址運算後，讀取結果位址資料並放在``rt``上。

* load word是計算``imm+rs``位址，並讀取該位址資料，寫入``rt``中

#### 流程：
1. **RegFile**讀**rs**的資料並送ALU
2. **immediate**經sign extension後，送入ALU
3. **Control**讀OP code後，結合instruction的**funct**部分控制ALU完成運算
4. 運算結果送至**DataMemory**的Address欄位
5. **Control**控制**DataMemory**的MemRead，讀取該位址的資料並送進**RegFile**的WriteData
6. 讀**rt**進RegFile的**WriteReg**
7. 完成寫入 

### Store word(I-type)

##### ``sw $s1, 12($s2)``
##### store進記憶體，因此是在memory進行寫入

#### 目標： immediate經過sign-extension後和``rs``位址運算後，得到目標位址，並讀取``rt``資料放在其上。

#### 流程：

1. **RegFile**讀**rs**的資料並送ALU
2. **immediate**經sign extension後，送入ALU
3. **Control**讀OP code後，控制ALU完成運算
4. 運算結果送至**DataMemory**的Address欄位
5. **RegFile**用ReadReg2, ReadData2讀**rt**資料，送至**DataMemory**的WriteData
6. Control控制**MemWrite**，將WriteData內容寫入至Address欄位

### Branch(I-type)

##### ``beq $s1, $s2, 20``

#### 目標： 比較暫存器內容是否相等，並計算跳躍位址(ALU的zero及``PC+4``)

#### 流程： 
1. 把**rs**, **rt**分別送進**RegFile**內的**readReg1**, **readReg2**
2. **PC+4**送進加法器
3. **RegFile**讀取資料後送至**ALU**，判斷是否為zero
4. immediate做sign extension後，left shift 2位(x4)後送至加法器和``PC+4``做運算，得到**Branch target address**

### Jump

##### ``j LOOP``

#### 流程：
1. 限制跳躍路徑在同一block內，因此跳躍目的位址的前四碼與目前位址相同，不需紀錄；byte address，因此位址後兩位為``00``，pseudo address的

### DataPath

* 兩個input時需靠**mux**決定
* 可以output至多個元件
* mux由**control unit**控制

![](https://hackmd.io/_uploads/r1mxBM6Fn.png)


### Critical path

- def: 所花時間最長者
- 簡化版Datapath：

![](https://hackmd.io/_uploads/HJ7SRQ6F2.png)

- 簡化版的簡化版：

|        | IM  | Reg | Mux | ALU | DM  | Mux | Reg |
|:------:|:---:|:---:|:---:|:---:| --- |:---:|:---:|
| R-type |  V  |  V  |  V  |  V  |     |  V  |  V  |
|   lw   |  V  |  V  |     |  V  | V   |  V  |  V  |
|   sw   |  V  |  V  |     |  V  | V   |     |     |
|  beq   |  V  |  V  |  V  |  V  |     |  V  |     |
|  jump  |     |     |     |     |     |     |     |



- 最長者為load word
- load word決定minimum clock cycle time

*指令種類與各指令所用之功能單元*



|    type    |   unit   |    unit    | unit |    unit    |    unit    |
|:----------:|:--------:|:----------:|:----:|:----------:|:----------:|
|   R-type   | 指令擷取 | 暫存器存取 | ALU  | 暫存器存取 |            |
| load word  | 指令擷取 | 暫存器存取 | ALU  | 記憶體存取 | 暫存器存取 |
| store word | 指令擷取 | 暫存器存取 | ALU  | 記憶體存取 |            |
|    beq     | 指令擷取 | 暫存器存取 | ALU  |            |            |
|    jump    | 指令擷取 |            |      |            |            |



## Construct a control unit

- MUX的來源及功用：


| whose input |       0       |       1       |             問              |
|:-----------:|:-------------:|:-------------:|:---------------------------:|
|  mux of PC  |     PC+4      |  PC+4+位移量  |          是不是beq          |
|     PC      |   mux of PC   |     jump      |         是不是jump          |
|  write reg  |      rt       |      rd       |        是不是R-type         |
|     ALU     | readData(reg) |   immediate   |        要不要跟imm做運算         |
|  writeData  |  ALU result   | readData(mem) | 要不要把memory的東西寫進reg |

- ALU control


| ALU control |     function     |
|:-----------:|:----------------:|
|    0000     |       AND        |
|    0001     |        OR        |
|    0010     |       add        |
|    0110     |    substract     |
|    0111     | set on less than |
|    1100     |       NOR        |

- True table of control signal



|          | add | sub | lw  |  sw   |  beq  |
|:--------:|:---:|:---:| --- |:-----:|:-----:|
| OP code  |  0  |  0  | 35  |  43   |   4   |
|  regDst  |  1  |  1  | 0   | ==x== | ==x== |
|  ALUsrc  |  0  |  0  | 1   |   1   |   0   |
| MemtoReg |  0  |  0  | 1   | ==x== | ==x== |
| regWrite |  1  |  1  | 1   |   0   |   0   |
| memRead  |  0  |  0  | 1   |   0   |   0   |
| memwrite |  0  |  0  | 0   |   1   |   0   |
|  branch  |  0  |  0  | 0   |   0   |   1   |
|  ALUop0  |  1  |  1  | 0   |   0   |   0   |
|  ALUop1  |  0  |  0  | 0   |   0   |   1   |

- 4條mux control: ``regDst, ALUsrc, MemtoReg, branch``
- 3條資料讀寫: ``regWrite, memRead, memWrite``
- 2條ALU control: ``ALUop0, ALUop1``

> Q：什麼時候可以``x``?<br/>
> A：input不進行寫入的時候，當operation不會寫入memory或register，control unit不需浪費力氣控制輸入的mux為何


## Performance

- single cycle: 執行時間最長的指令所花時間即為clock cycle time
- multiple cycle: 各指令的執行時間\*出現頻率為clock cycle time

## Multiple cycle processor

- single cycle的缺點在於其所有指令的執行時間皆為一Clock cycle time，取決於critical path指令；最長及最短指令的執行時間相同，效率不佳
- multi-cycle將各指令再拆分成一連串步驟，每個步驟花費一個Clock cycle time
- 執行時間長的指令步驟較多，所需CPI也較大
- 一個步驟只能使用一個主要功能單元

| instruction | step | step  |        step         | step | step | CPI |
|:-----------:|:----:|:-----:|:-------------------:|:----:|:----:|:---:|
|   R-type    |  IF  | ID/RF |       execute       |      |  WB  |  4  |
|     lw      |  IF  | ID/RF |      addr cal       |  MA  |  WB  |  5  |
|     sw      |  IF  | ID/RF |      addr cal       |  MA  |      |  4  |
|     beq     |  IF  | ID/RF |     compare reg     |      |      |  3  |
|    jump     |  IF  |  ID   | compose target addr |      |      |  3  |
