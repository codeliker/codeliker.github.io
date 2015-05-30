---
layout: docs
category : code
tagline: "cpp"
tags : [PE, export, table,RVA,offset]
---

1.PE文件结构
PE前面的内容：
DOS头、NT头、节表
节表紧跟在NT头后面，NT头有个字段表名节表的个数。他们的关系如下：
{% highlight bash %}
IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)BaseAddr;
IMAGE_NT_HEADERS* pNTHeader = (IMAGE_NT_HEADERS*)((ULONG)BaseAddr + pDosHeader->e_lfanew);
PIMAGE_SECTION_HEADER pSectionHeader =(PIMAGE_SECTION_HEADER) ((ULONG)pNTHeader + sizeof(IMAGE_NT_HEADERS));
{% endhighlight %}

2.相对虚拟地址（RVA）和文件偏移(Offset)
RVA：当PE文件加载到内存后（如通过PE加载器加载到内存中），相对内存基址BaseAddr的偏移
Offset:相对PE文件头部的偏移
在PE文件结构中，很多地址记录的都是RVA，如果要对应到文件偏移，需要进行转换。转换的方法如下：
首先定位到RVA处于哪个节中，然后计算出文件偏移。
每个节包含RVA、文件偏移、大小等信息，通过遍历，可以知道RVA处于哪个节，接着使用如下方法即可得到Offset:
{% highlight bash %}
Offset = (LPVOID)((ULONG)RVA - pTempSecHeader->VirtualAddress + pTempSecHeader->PointerToRawData);
{% endhighlight %}
完整代码如下：
{% highlight bash %}
//根据RVA,得到文件偏移
LPVOID RVAToOffset(LPVOID BaseAddr,LPVOID RVA)
{
	LPVOID Offset = NULL;

	IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)BaseAddr;
	IMAGE_NT_HEADERS* pNTHeader = (IMAGE_NT_HEADERS*)((ULONG)BaseAddr + pDosHeader->e_lfanew);
	PIMAGE_SECTION_HEADER pSectionHeader =(PIMAGE_SECTION_HEADER) ((ULONG)pNTHeader + sizeof(IMAGE_NT_HEADERS));

	DWORD i =0;
	PIMAGE_SECTION_HEADER pTempSecHeader;
	for (i=0;i< pNTHeader->FileHeader.NumberOfSections;i++)
	{
		pTempSecHeader = (PIMAGE_SECTION_HEADER)((ULONG)pSectionHeader + sizeof(IMAGE_SECTION_HEADER) * i);

		if ((ULONG)RVA >= pTempSecHeader->VirtualAddress && (ULONG)RVA < (ULONG)(pTempSecHeader->VirtualAddress + pTempSecHeader->Misc.VirtualSize))
		{//导出表处于该节内
			Offset = (LPVOID)((ULONG)RVA - pTempSecHeader->VirtualAddress + pTempSecHeader->PointerToRawData);
			break;
		}
	}

	return Offset;
}
{% endhighlight %}


3.导出表（Export Table)
NT头部的OptionHead部分的DataDirectory数组，存放各个导出、导入等表的RVA和大小，导出表RVA获取方法如下：
{% highlight cpp %}
LPVOID pExport_RVA = (LPVOID)(pOptionHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
{% endhighlight %}
导出表中包含着AddressOfFunctions、AddressOfNames、AddressOfNameOrdinals（均是RVA），均要注意的是：
得到AddressOfFunctions、AddressOfNames均是地址数组，他们中存放的地址也是RVA。
因此，通过导出表，得到函数名、RVA、文件偏移地址代码如下：
{% highlight cpp %}
	PIMAGE_EXPORT_DIRECTORY pExport = (PIMAGE_EXPORT_DIRECTORY)((ULONG)pExport_Offset + (ULONG)BaseAddr);
	DWORD nCount = pExport->NumberOfFunctions;//模块中导出的函数数目
	ULONG* pFuncAddr = (ULONG*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfFunctions));//函数地址基址
	ULONG* pNameArray = (ULONG*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfNames));//名字基址地址存放地
	USHORT* pIndex = (USHORT*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfNameOrdinals));//序号

	////得到函数名及其对应的文件偏移
	ULONG taddr = 0;
	DWORD i;
	for(i=0;i<nCount;i++)
	{
		
		char* psFuncName = (char*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pNameArray[i]));//函数名
		DWORD nIndex = pIndex[i] + pExport->Base - 1;//索引号
		ULONG addr = pFuncAddr[nIndex];//地址(RVA)
		ULONG addr_offset = (ULONG)RVAToOffset(BaseAddr,(LPVOID)addr);//转为文件偏移
		//打印：序号 函数名  RVA 文件偏移
		printf(("Index:%d Name:%s RVA:0x%08x Offset:0x%08x\n"),nIndex,psFuncName,addr,addr_offset);
	}
{% endhighlight %}


4.通过PE文件，获得导出表函数RVA、Offset、函数名完整代码如下：
{% highlight cpp %}
void GetExportFuncAddr(TCHAR* filename)
{
	LPVOID BaseAddr = ReadFileToBuffer(filename);
	if(BaseAddr == NULL)
	{
		printf("ReadFileToBuffer fail.\n");
		return;
	}

	/****************************核心实现区********************************/
	//PVOID BaseAddr = hModule;
	IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)BaseAddr;
	IMAGE_NT_HEADERS* pNTHeader = (IMAGE_NT_HEADERS*)((ULONG)BaseAddr + pDosHeader->e_lfanew);
	IMAGE_OPTIONAL_HEADER* pOptionHead = (IMAGE_OPTIONAL_HEADER*)&(pNTHeader->OptionalHeader);
	//导出表相对虚拟地址（RVA)
	LPVOID pExport_RVA = (LPVOID)(pOptionHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);

	PIMAGE_SECTION_HEADER pSectionHeader =(PIMAGE_SECTION_HEADER) ((ULONG)pNTHeader + sizeof(IMAGE_NT_HEADERS));
	
	LPVOID pExport_Offset= RVAToOffset(BaseAddr,pExport_RVA);

	PIMAGE_EXPORT_DIRECTORY pExport = (PIMAGE_EXPORT_DIRECTORY)((ULONG)pExport_Offset + (ULONG)BaseAddr);
	DWORD nCount = pExport->NumberOfFunctions;//模块中导出的函数数目
	ULONG* pFuncAddr = (ULONG*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfFunctions));//函数地址基址
	ULONG* pNameArray = (ULONG*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfNames));//名字基址地址存放地
	USHORT* pIndex = (USHORT*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pExport->AddressOfNameOrdinals));//序号

	//得到函数名及其对应的文件偏移
	ULONG taddr = 0;
	DWORD i;
	for(i=0;i<nCount;i++)
	{
		
		char* psFuncName = (char*)((ULONG)BaseAddr + (ULONG)RVAToOffset(BaseAddr,(LPVOID)pNameArray[i]));//(char*)((ULONG)BaseAddr + pNameArray[i]);//函数名
		DWORD nIndex = pIndex[i] + pExport->Base - 1;//索引号
		ULONG addr = pFuncAddr[nIndex];//地址(RVA)
		ULONG addr_offset = (ULONG)RVAToOffset(BaseAddr,(LPVOID)addr);//转为文件偏移
		//打印：序号 函数名  文件偏移
		printf(("Index:%d Name:%s RVA:0x%08x Offset:0x%08x\n"),nIndex,psFuncName,addr,addr_offset);
	}

	/*********************************************************************/

	//释放工作
	//FreeLibrary(hModule);
	VirtualFree(BaseAddr,0,MEM_RELEASE);
}

{% endhighlight %}