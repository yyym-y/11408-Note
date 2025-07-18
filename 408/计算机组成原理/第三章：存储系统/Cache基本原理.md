# 一、Cache 基本原理


**物理实现：**
为了最大限度地减少延迟，现代计算机设计通常将 Cache **集成在 CPU 内部**。这消除了CPU与外部存储器之间的数据传输延迟。

**硬件材料：**
Cache 使用的是 **SRAM（静态随机存取存储器）**芯片来实现。SRAM 比DRAM 更快，因为它不需要刷新，且电路结构更简单。


## 3. 局部性原理

**程序局部性原理**是 Cache 能够有效工作的根本原因。它指出程序在执行时，对信息（指令和数据）的访问往往集中在存储空间的一个较小区域内，并且在一段时间内会重复访问这些信息。



**1. 空间局部性 (Spatial Locality)：**

* **表现：** 当程序访问了某个存储单元后，其**相邻**的存储单元（如数组中的下一个元素）在很短的时间内也极有可能被访问。
* **原因：**
    * **数据在内存中顺序存储：** 数组元素通常在内存中是按行优先（或列优先）连续存放的。当访问一个元素时，其旁边的元素也已经在靠近的位置。
    * **程序多顺序访问：** 许多程序倾向于按顺序遍历数据结构（如数组、链表）。
    * **指令表现：** 机器指令在内存中也是顺序存放的。CPU在执行完一条指令后，很可能接着执行其后的指令，这也体现了指令的“空间局部性”。

**2. 时间局部性 (Temporal Locality)：**

* **表现：** 在程序执行过程中，一个已经访问过的信息（指令或数据）在不久的将来很可能**再次被访问**。
* **典型场景：**
    * **循环结构：** 这是体现时间局部性最典型的例子。在循环中，循环体内的指令（如加法、比较）会被重复执行多次，循环变量（如 `i`, `j`）和累加变量（`sum`）也会被周期性地访问和修改。
    * **函数调用：** 函数内部的指令和局部变量在函数执行期间会被频繁访问。


**Cache 策略：**
基于局部性原理，Cache 的管理策略通常是：当 CPU 访问某个地址时，不仅会把该地址的数据调入 Cache，还会将其**周围的数据**（即包含该地址的整个数据块）一并**预取**到 Cache 中。例如，访问数组元素 `array[i][j]` 时，会把 `array[i][j]` 所在的整个内存块都调入 Cache，以备后续对 `array[i][j+1]` 等相邻元素的访问。

---

## 4. 性能分析

有两种访存的策略: 1. 缓存未命中之后再访问主存, 2. 同时访问缓存和主存

假设命中概率为 $$H$$

**1. 同时访问策略下的平均访问时间：**
当 Cache 和主存同时被访问时：
* **命中时：** 访问 Cache，时间为 $$t_c$$。主存的访问虽然开始，但因为 Cache 命中而被忽略，所以有效访问时间就是 $$t_c$$。
* **未命中时：** Cache 未命中，CPU 必须等待主存完成访问。时间为 $$t_m$$。
* **平均访问时间 ($$t_{avg}$$):**
    $$t_{avg} = H \times t_c + (1-H) \times t_m$$


**对比分析：若采用先访问 Cache 再访问主存策略：**
在这种策略下，CPU会先检查 Cache。如果未命中，才会再去访问主存。
* **命中时：** 访问 Cache，时间为 $$t_c$$。
* **未命中时：** 先访问 Cache（$$t_c$$），再访问主存（$$t_m$$）。总时间为 $$t_c + t_m$$。
* **平均访问时间 ($$t'_{avg}$$):**
    $$t'_{avg} = H \times t_c + (1-H) \times (t_c + t_m)$$


**结论：**
通过对比可见，**同时访问策略通常效率更高**，因为它减少了未命中时等待 Cache 检查的时间，从而实现了更明显的性能提升。在这个例子中，同时访问策略提升了约 4.17 倍，而先访问策略提升了 4 倍。

---


基于局部性原理，系统将主存划分为固定大小的**块（Block）**。每个块是 Cache 与主存之间数据交换的最小单位，通常大小为几KB（如1KB、2KB、4KB等）。
* 当CPU访问主存中的某个地址时，系统不会只调入该地址的数据，而是将该地址所在的**整个主存块**一并调入 Cache。
* **示例：** 如果主存块大小为1KB，当CPU访问数组的某个元素时，其所在的1KB内存块（包含后续相邻的元素）都会被调入 Cache。

* **交换单位：** Cache 与主存之间的数据交换总是以**“块”**为单位进行。这意味着主存和 Cache 都被划分为大小相同的块。
* **示例：** 如果主存容量为4MB，每块大小为1KB，则主存被划分为 $$4MB / 1KB = 4096$$ 个块。如果 Cache 容量为8KB，每块大小为1KB，则 Cache 被划分为 $$8KB / 1KB = 8$$ 个块。
* **术语说明：**
    * 主存中的块通常也称作**“页 (Page)”**或**“页框 (Page Frame)”**（在虚拟内存管理中）。
    * Cache 中的块通常称作**“行 (Line)”**。


### 5.4 主存地址的块号与块内地址

CPU 发出的主存地址不再仅仅指向一个字节，而是包含了块的信息。

* **地址结构：** 一个主存地址可以被逻辑上划分为两部分：**块号 (Block Number)** 和 **块内地址 (Offset within Block)**。
    * 例如，对于一个 22 位的物理主存地址（对应4MB主存）和 1KB 的块大小：
        * **块内地址：** 1KB = $$2^{10}$$ 字节，所以需要 10 位来表示块内的偏移量（0 到 1023）。
        * **块号：** 剩余的 $$22 - 10 = 12$$ 位用于表示块号（0 到 4095）。
* **寻址过程：** 当 CPU 给出 22 位地址时，前 12 位告诉我们数据在哪一个主存块，后 10 位告诉我们数据在这个块的哪个位置。

### 5.5 CPU 访问主存后的数据调入

* **调入机制：** 当 CPU 访问主存中的某个单元（地址）且该数据不在 Cache 中时（Cache 未命中），系统会立即将其所属的**整个主存块**从主存调入到 Cache 中。
* **注意：** 这是一个**数据复制**过程，而不是移动。主存中的数据母本会保留不变，Cache 中存放的是其副本。
* **后续问题：** 数据调入 Cache 后，如何记录这个 Cache 块是从主存的哪个块复制过来的？CPU 如何知道 Cache 中某一块数据对应主存的哪个地址？这需要解决**映射关系**问题。

---

## 二、知识小结

| 知识点                 | 核心内容                                                                                                                       | 考试重点/易混淆点                                                                                                                                                                             | 难度系数 |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- |
| **Cache 工作原理** | 通过高速缓冲存储器（Cache）缓解CPU与主存速度矛盾，基于程序局部性原理（空间局部性、时间局部性）预加载相邻数据。                 | **命中率计算**（$h$）、**平均访问时间公式**（$t_{avg} = h \cdot t_c + (1-h) \cdot t_m$）。理解 Cache 存在的必要性。                                                                         | ⭐⭐⭐     |
| **局部性原理** | **空间局部性：** 当访问一个数据后，其相邻数据很可能被连续访问；**时间局部性：** 循环结构导致同一指令/数据重复访问。           | 区分程序 A（行访问） vs 程序 B（列访问）在**空间局部性**上的差异，以及它们对 Cache 性能的影响。这是理解 Cache 有效性的基础。                                                                  | ⭐⭐      |
| **主存-Cache 映射** | 主存与 Cache 以**块**（通常为1KB等）为单位交换数据。CPU 发出的主存地址被逻辑上划分为**块号**和**块内地址**。                   | **主存块的别名**（页/页框）与 **Cache 块的别名**（行）。理解地址划分的目的。                                                                                                                  | ⭐⭐⭐⭐    |
| **性能提升计算** | 掌握两种常见访问策略下的平均访问时间计算：1. **先查 Cache 再查主存** ($$t_{avg} = h \cdot t_c + (1-h) \cdot (t_c + t_m)$$); 2. **同时访问** ($$t_{avg} = h \cdot t_c + (1-h) \cdot t_m$$)。 | 典型例题：Cache 速度为主存 5 倍，命中率 95% → 性能提升约 4.17 倍。这是考试中常见的计算题型，需要熟练掌握。                                                                                    | ⭐⭐⭐⭐    |
| **待解决问题** | 1. **映射方式**（如何记录主存块与 Cache 块的关系）；2. **替换算法**（Cache 满时淘汰策略）；3. **写策略**（保证 Cache 与主存数据一致性）。 | 这些是 Cache 设计中的核心挑战，也是后续章节的重点内容。理解这些问题的存在是为了更好地过渡到后续的详细讲解。                                                                                    | ⭐⭐⭐⭐    |