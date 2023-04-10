## SBreak系统调用
- 功能：调整进程数据段,返回数据段的新结尾地址
```cpp
void Process::SBreak()
{
	// 获取user结构
    User& u = Kernel::Instance().GetUser();
	// newEnd是sbrk或brk系统调用的唯一参数，传入数据段的新结尾
    unsigned int newEnd = u.u_arg[0];
	// 获取内存描述符，并更新数据段的大小
    MemoryDescriptor& md = u.u_MemoryDescriptor;
    unsigned int newSize = newEnd - md.m_DataStartAddress;
	// newEnd为0,表示数据段大小不变
    if (newEnd == 0)
    {
        u.u_ar0[User::EAX] = md.m_DataStartAddress + md.m_DataSize;
        return;
    }
    
	/*
     * @comment 原unix v6中estabur()函数，用于建立用户态地址空间的相对地址映射表，然后调用
     * MapToPageTable()函数将相对地址映射表加载到用户态页表中。
     */
     // 此处如果超出了8M的用户程序地址空间限制,会返回false,无法为数据段建立地址映射
    if ( false == u.u_MemoryDescriptor.EstablishUserPageTable(md.m_TextStartAddress,md.m_TextSize, md.m_DataStartAddress,newSize, md.m_StackSize))
    {
        //系统调用出错时，不可以用这种方式返回。执行这条路径会导致 u.u_intflg == 1，u.u_error被错误修改为EINTR（4）；无论何故导致系统调用失败。
        //aRetU(u.u_qsav);
        return;
    }
```
- EstablishUserPageTable函数的实现
```cpp
bool MemoryDescriptor::EstablishUserPageTable( unsigned long textVirtualAddress, unsigned long textSize, unsigned long dataVirtualAddress, unsigned long dataSize, unsigned long stackSize )
{
    User& u = Kernel::Instance().GetUser();
    /* 如果超出允许的用户程序最大8M的地址空间限制 */
    if ( textSize + dataSize + stackSize  + PageManager::PAGE_SIZE > USER_SPACE_SIZE - textVirtualAddress)
    {
        u.u_error = User::ENOMEM;
        Diagnose::Write("u.u_error = %d\n",u.u_error);
        return false;
    }
	
    this->ClearUserPageTable();
    /* 以相对起始地址phyPageIndex为0，为正文段建立相对地址映照表 */
    unsigned int phyPageIndex = 0;
    phyPageIndex = this->MapEntry(textVirtualAddress, textSize, phyPageIndex, false);
    /* 以相对起始地址phyPageIndex为1，ppda区占用1页4K大小物理内存，为数据段建立相对地址映照表 */
    phyPageIndex = 1;
    phyPageIndex = this->MapEntry(dataVirtualAddress, dataSize, phyPageIndex, true);
    /* 紧跟着数据段之后，为堆栈段建立相对地址映照表 */
    unsigned long stackStartAddress = (USER_SPACE_START_ADDRESS + USER_SPACE_SIZE - stackSize) & 0xFFFFF000;
    this->MapEntry(stackStartAddress, stackSize, phyPageIndex, true);
    /* 将相对地址映照表根据正文段和数据段在内存中的起始地址pText->x_caddr、p_addr，建立用户态内存区的页表映射 */
    this->MapToPageTable();
    return true;
}
```
- 继续SBreak函数
```cpp
	// 数据段大小的改变量
    int change = newSize - md.m_DataSize;
    md.m_DataSize = newSize;
    // USIZE为一个页表的大小0x1000,估计与Expand的具体实现有关,不用管
    // Expand:重新为数据段分配内存空间
    newSize += ProcessManager::USIZE + md.m_StackSize;
    /* 数据段缩小 */
    // 先转移内存,再调用Expand释放多余内存
    if ( change < 0 )
    {
        int dst = u.u_procp->p_addr + newSize - md.m_StackSize;
        int count = md.m_StackSize;
        while(count--)
        {
            Utility::CopySeg(dst - change, dst);
            dst++;
        }
        this->Expand(newSize);
    }

    /* 数据段增大 */
    // 先调用Expand分配新的内存,再进行转移
    else if ( change > 0 )
    {
        this->Expand(newSize);
        int dst = u.u_procp->p_addr + newSize;
        int count = md.m_StackSize;
        while(count--)
        {
            dst--;
            Utility::CopySeg(dst - change, dst);
        }
    }
	// 返回新的数据段末尾地址
    u.u_ar0[User::EAX] = md.m_DataStartAddress + md.m_DataSize;

}
```
## Brk和SBrk系统调用
- brk系统调用,调用17号入口函数,传入Data段末尾作为系统调用参数
```cpp
/* 使用errno需要include "stdlib.h" */
extern errno;
int brk(void * newEndDataAddr)
{
    int res;
    __asm__ volatile ("int $0x80":"=a"(res):"a"(17),"b"(newEndDataAddr));
    /* 系统调用的返回值赋值APP的全局变量errno */
    if ( res >= 0 )
        return res;
	// 如果返回res为负数,输出错误
    errno = -1*res;
    printf("%d\n",errno);
    return -1;
}
// 17号系统调用入口函数就是sBreak
```
- sbrk系统调用,调用brk系统调用
```cpp
// fakeedata,可能使fake的data段end?
unsigned int fakeedata = 0;
int sbrk(int increment)
{
	// 若fakeedata为0,数据段大小不变
    if (fakeedata == 0)
    {
        fakeedata = brk(0);
    }
    // 字节-1,可能是由于从0开始
    unsigned int newedata = fakeedata + increment - 1;
    // brk输入以页偏移为单位,此处是页数加1
    brk(((newedata >> 12) + 1) << 12);
    fakeedata = newedata + 1;
    return fakeedata;
}
```
## malloc系统调用
```cpp
char *malloc_begin = NULL;
char *malloc_end = NULL;

typedef struct flist {
   unsigned int size;
   struct flist *nlink;
};

struct flist *malloc_head = NULL;
void* malloc(unsigned int size)
{
    if (malloc_begin == NULL)
    {
        /* code */
        malloc_begin = sbrk(0);
        malloc_end = sbrk(PAGE_SIZE);
        malloc_head = malloc_begin;
        malloc_head->size = sizeof(struct flist);
        malloc_head->nlink = NULL;
    }

    if (size == 0)
    {
        return NULL;
    }

    size += sizeof(struct flist);
    size = ((size + 7) >> 3) << 3;
    struct flist* iter = malloc_head;
    // find a place to insert
    while(iter->nlink)
    {
        if ((int)(iter->nlink) - iter->size - (int)iter >= size)
        {
            struct flist *temp = (char *)iter + (iter->size);
            temp->nlink = iter->nlink;
            iter->nlink = temp;
            temp->size = size;
            return (char *)temp + sizeof(struct flist);
        }
        iter = iter->nlink;
    }
    // not found

    int expand = size - (malloc_end - (char *)iter - (iter->size));

    expand = ((expand + PAGE_SIZE - 1) / PAGE_SIZE) * PAGE_SIZE;

    malloc_end = sbrk(expand);

    iter->nlink = (char *)iter + (iter->size);

    iter = iter->nlink;

    iter->size = size;

    iter->nlink = NULL;

    printf("%u\n", iter);

    return (char*)iter + sizeof(struct flist);

}
```