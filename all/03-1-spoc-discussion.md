# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

>   * 【最优匹配】优点：大部分分配的尺寸较小时，效果更好，可避免大的空闲分区被拆分 缺点：释放分区慢，容易产生很多无用的小碎片
>   * 【最差匹配】优点：中等大小的分配较多时，效果更好 缺点：释放分区慢，容易破坏较大的空闲分区
>   * 【最先匹配】优点：实现简单，在高地址空间有大块的空闲分区 缺点：分配大块时较慢
>   * 【buddy system】优点：使用2的幂次来分配空间，能够避免把大的内存块拆的太碎，且使分配和释放过程迅速 缺点：拆分和合并需要大量的链表和位图操作，开销可能会较大
>   * 【随机分配算法】每次随机寻找一个比需求大的块进行分配

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)
> * 使用了树状结构来组织，如chapter5-5中第五页的幻灯片所示。每次分配空间，从根节点出发进行DFS查找，并根据所需空间与空闲空间一半的大小关系选择占用空闲空间或者分割空闲空间。回收空间时，需要判断相邻空间大小是否相等，并且地址小的空间起始位置是否为空间大小的倍数。

```
#include<iostream>
using namespace std;

//采用树结构组织空间
class Page{
public:
	int start;
	int page_size;
	int rem;
	Page *parent;
	Page *left;
	Page *right;
	Page(int s, int r, Page *par){
		start = s;
		rem = r;
		page_size = r;
		parent = par;
		left = NULL;
		right = NULL;
	}
};
Page root(0, 1<<20, NULL); //初始分配1M的空间

void update_rem(Page* node) 
{
	if(node->parent)
	{	
		//向上更新树
		int rem_new = max(node->parent->left->rem, node->parent->right->rem);
		if( node->parent->rem != rem_new)
		{
			node->parent-> rem = rem_new;
			update_rem(node->parent);
		}
	}
}

Page* find_page(Page* cur, int size_p)
{
	if(cur->rem < size_p)
		return NULL;
	if(!(cur->left)&&!(cur->right)) //没有子节点
	{
		if((cur->rem >> 1) < size_p) //空闲空间的一半小于所需大小，则整块空间被占用
		{	
			cur->rem = 0;
			update_rem(cur);
			return cur;
		}
		//将空间进行拆分
		cur->left = new Page(cur->start, cur->rem>>1, cur);
		cur->right = new Page(cur->start + (cur->rem>>1), cur->rem>>1, cur);
		return find_page(cur->left, size_p);
	}
	else //有子节点
	{
		if(cur->left->rem >= size_p) 
			return find_page(cur->left,size_p);
		else 
			return find_page(cur->right,size_p); 
	}
}

Page* alloc_page(int size_p)
{
	return find_page(&root, size_p);
}

void free_page(Page* p)
{
	p->rem = p->page_size; 
	Page* par = p->parent;
	if(par){
		if(par->left->page_size != par->right->page_size) return;
		int size_p = par->left->page_size;
		if(par->left->rem == size_p && par->right->rem == size_p && (par->left->start % size_p == 0))
		{
			//符合三个条件才进行合并节点并删除子节点
			delete par-> left;
			delete par-> right;
			par->left = NULL;
			par->right = NULL;
			free_page(par);
		}
	}
} 

void print_allpage(Page* cur){
	if(cur->left){
		print_allpage(cur->left);
	}
	if(cur->right){
		print_allpage(cur->right);
	}
	if(!cur->right && !cur->left){
		if(cur -> rem != 0) cout << "free_space: " << ((cur->start) >> 10) << "k ->" << ((cur->start+cur->page_size) >> 10) <<"k "<< endl;
		else
			cout << "occupied_space: " << ((cur->start) >> 10) << "k ->" << ((cur->start+cur->page_size) >> 10) <<"k "<< endl;
	}

}


int main()
{
	Page* a = alloc_page(128*(1<<10));
	Page* b = alloc_page(64*(1<<10));
	Page* c = alloc_page(64*(1<<10));
	Page* d = alloc_page(256*(1<<10));
	Page* e = alloc_page(256*(1<<10));
	Page* f = alloc_page(256*(1<<10));
	cout << "insert ABCDEF" << endl;
	print_allpage(&root);
	cout << endl;
	
	cout << "Delete EF" << endl;
	free_page(f);
	free_page(e);
	print_allpage(&root);
	cout << endl;
	
	cout << "DELETE ABC" << endl;
	free_page(a);
	free_page(b);
	free_page(c);
	print_allpage(&root);
	cout << endl;
	
	cout << "DELETE D" << endl;
	free_page(d);
	print_allpage(&root);
		
}

```
>   *测试结果（occupied_space表示此段空间已经被占用 free_space表示此段空间没有被占用）

```
insert ABCDEF
occupied_space: 0k ->128k 
occupied_space: 128k ->192k 
occupied_space: 192k ->256k 
occupied_space: 256k ->512k 
occupied_space: 512k ->768k 
occupied_space: 768k ->1024k 

Delete EF
occupied_space: 0k ->128k 
occupied_space: 128k ->192k 
occupied_space: 192k ->256k 
occupied_space: 256k ->512k 
free_space: 512k ->1024k 

DELETE ABC
free_space: 0k ->256k 
occupied_space: 256k ->512k 
free_space: 512k ->1024k 

DELETE D
free_space: 0k ->1024k 
```
--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



