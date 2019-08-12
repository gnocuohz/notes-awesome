## linux命令
### 内存
**pmap**  
pmap -x 7 | sort -n -k3  

**cat /proc/pid/maps**  
/proc/$PID/maps中的每一行描述进程或线程中连续虚拟内存的区域。每行都有以下字段：  

address  - 这是进程地址空间中区域的起始和结束地址  
permissions  - 描述如何访问区域中的页面。有四种不同的权限：读取，写入，执行和共享。如果禁用读/写/执行，则会出现 - 而不是r / w / x。如果某个区域未共享，则为私有区域，因此将显示p而不是s。如果进程尝试以不允许的方式访问内存，则会生成分段错误。可以使用mprotect系统调用更改权限。  
offset  - 如果区域是从文件映射的（使用mmap），则这是映射开始的文件中的偏移量。如果内存未从文件映射，则它只是0。  
device  - 如果区域是从文件映射的，则这是文件所在的主要和次要设备编号（十六进制）。  
inode  - 如果区域是从文件映射的，则这是文件编号。  
pathname  - 如果区域是从文件映射的，则这是文件的名称。匿名映射区域的此字段为空。还有一些特殊区域，其名称如[heap]，[stack]或[vdso]。 [vdso]代表虚拟动态共享对象。它被系统调用用于切换到内核模式。  
你可能会注意到很多匿名区域。这些通常由mmap创建，但不附加到任何文件。它们用于许多杂项内容，例如共享内存或未在堆上分配的缓冲区。例如，我认为pthread库使用匿名映射区域作为新线程的堆栈。  
**cat /proc/pid/smaps**  
slab top  
/proc/zoneinfo  
/proc/meminfo  
/proc/buddyinfo  
/proc/swaps  

cat /proc/15098/maps | sed -e "s/\([0-9a-f]\{8\}\)-\([0-9a-f]\{8\}\)/0x\1 0x\2/" | awk '{printf("\033[0;33m[%8d Page]\033[0m \033[0;35m[%8d KB]\033[0m %s\n", (strtonum($2) - strtonum($1))/4096, (strtonum($2) - strtonum($1))/1024, $0)}'  
