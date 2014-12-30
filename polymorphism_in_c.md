# Polymorphism in C

Polymorphism翻譯成中文是多型(態)

這是一個物件導向語言中的基本觀念，就是變數(class、struct)在run-time去改變行為而不是compile-time決定。

C語言很難做到這點，用function pointer抽換可以勉強做到，但這樣的實作effort容易使人卻步，換來的好處又有限，最大的缺點是超難用。client必須知道今天要掛甚麼function pointer，光這點就已經脫離物件導向的封裝概念了。

今天要介紹一種能夠完全封裝並且實作多型的機制，不多說，先來看看client的code

```
int main(int argc, char **argv)
{
	struct dl_module *module = NULL;
	int value = 0;
	
	printf("get %s's price: ", argv[1]);
	
	module = module_open(argv[1]);
	
	value = (*module->get_price)();
	printf("%d\n", value);
	
	module_close(module);
	return 0;
}
```
`struct dl_module`是我多型的interface，與多數的物件導向一樣，這是多型的必要條件，當使用者從shell餵進來不同的參數，程式必須動態的吐出價格。

這邊只有定義一個類別，卻可以做到run-time改變value的能力，這就是多型的典型特徵。

因為C語言沒有建構式和解構式，所以client只好傻傻的call `module_open`和`module_close`，自動化的建構式和解構式容我有機會再介紹吧。

接著來看神奇的module相關API
```
struct dl_module
{
	void *handle;
	int (*get_price)(void);
};

struct dl_module *module_open(char *name)
{
	char path[100];
	void *handle = NULL;
	struct dl_module *module = malloc(sizeof(struct dl_module));
	
	sprintf(path, "lib%s.so", name);
	handle = dlopen(path, RTLD_NOW);
	
	module->get_price = (int (*)(void))dlsym(handle, "get_price");
	module->handle = handle;
	return module;
}

void module_close(struct dl_module *module)
{
	dlclose(module->handle);
	free(module);
}
```
透過dlopen和dlsym，這就允許程式在run-time去動態載入shared library或dynamic library，而外部的shell只需要把.so準備好即可。

> gcc poly_test.c -ldl    
> gcc -fPIC cola.c -shared -o libcola.so    
> LD_LIBRARY_PATH=./ ./a.out cola    
> > get cola's price: 29    

 
example中的cola可以換成各種東西，例如sunflower或banana，只是記得要實作libsunflower.so。

至於cola.c裡面其實就只有一個function: get_price，而且很單純的return 29 (反正盡量簡單比較好懂)

這種做法就將不同的實作封裝在shared library中，而client可以透過string去抽換功能，甚至更殺一點的，連function name都用string去抽換也行。

這些sample code都沒做error handle，要抄的人記得自己加 (不然很容易SIGSEGV)

只是

只是

只是

只是

只是

只是

這種作法是把compile-time可以抓到的bug推遲到run-time去，若是筆誤`get_price`寫成get_prince，這支程式就掰了。

配套就是寫一個parser去建立symbol的總表並且去檢查所有的module_open是不是符合總表的內容，然後在Makefile內去call這個parser做preprocess，如此即可兩全其美，又有好的程式架構又能兼顧compile-time bug free。
