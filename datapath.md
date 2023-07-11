
## R-type

##### ``add $t0, $t1, $t2``

##### 皆在暫存器內完成

#### 目標： *register file*：``rs = rt + rd``，經由*ALU*運算。


#### 流程：
1. **RegFile**讀**rs**, **rt**位址進**readReg1**, **readReg2**，並由**readData1**, **readData2**，送資料進**ALU**
2. **Control**讀取OP code及funct，控制ALUcont，完成計算
3. ALU計算結果送進**RegFile**中的**writeData**
4. **RegFile**讀取**rd**暫存器
5. **Control**控制regWrite完成寫入至**rd**

## Load word(I-type)

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

## Store word(I-type)

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

## Branch(I-type)

##### ``beq $s1, $s2, 20``

#### 目標： 比較暫存器內容是否相等，並計算跳躍位址(ALU的zero及``PC+4``)

#### 流程： 
1. 把**rs**, **rt**分別送進**RegFile**內的**readReg1**, **readReg2**
2. **PC+4**送進加法器
3. **RegFile**讀取資料後送至**ALU**，判斷是否為zero
4. immediate做sign extension後，left shift 2位(x4)後送至加法器和``PC+4``做運算，得到**Branch target address**

## DataPath

* 兩個input時需靠**mux**決定
* 可以output至多個元件
* mux由**control unit**控制

![](https://hackmd.io/_uploads/SJS1uY9Fh.png)

## Datapath for Control

