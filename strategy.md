# Strategy

這篇要講的是個比callback function屌上一百萬倍的技巧，不會這招千萬別說你會寫C語言，剛好可以配上今天要講的主題：strategy。

不免俗要講一下，strategy是design pattern中的大範本！很多pattern都是從strategy衍伸出去，當一件事情有很多種做法的時候，可以在run-time選擇做法。

當然，這不一定要用strategy，可以用一堆if-else，但是用了strategy可以讓程式碼更容易維護，要新增功能也很方便。

先來一段C++的標準code
```
#include <iostream>
#include <cstdio>
#include <cstdlib>
using namespace std;
 
class Sorter {
    public:
        virtual void sort() = 0;
};
class QuickSorter : public Sorter{
    public:
        void sort(){ printf("Quick sort is selected.\n"); }
};
class BubbleSorter : public Sorter{
    public:
        void sort(){ printf("Bubble sort is selected.\n"); }
};
 
int main()
{
    Sorter *sorter = NULL;
    string input = "default";
 
    while(input != "q"){
        system("cls");
        printf("Choose a kind of sorting:\n(a) Quick sort\n(b) Bubble sort\n(q) Quit\n");
        cin >> input;
 
        if(sorter != NULL) delete sorter;
        if(input == "a"){
            sorter = new QuickSorter;
        }else if(input == "b"){
            sorter = new BubbleSorter;
        }else continue;
        sorter->sort();
        system("pause");
    }
    if(sorter != NULL) delete sorter;
    return 0;
}
```
上述是C++玩Strategy的做法，要做排序有很多種排序的演算法，那就將排序拉出去變成一個class，要新增排序的方式就去繼承，例如`class MergeSorter : public Sorter`，這樣就多了一種Merge sort。
 
在C語言當中沒有class的概念，再三強調這點，所以要怎麼做到run-time抽換呢？當然用function pointer也行，寫一堆排序的function pointer來挑著用，也滿方便的，也很容易維護和新增。
 
但是！！！！！！！！！！！！

如果每次都搞一樣的招數，別說看的人，就連我都煩了。所以，這邊有一個新招，`用MACRO來完成`，MACRO相信寫C語言的人一定不陌生，但接下來的用法請睜大眼睛看，你們一定非常陌生！

```
#include <stdio.h>
 
#define SORT(x) sort_##x##_()
 
void sort_quick_(){ printf("Quick sort is selected.\n"); }
void sort_bubble_(){ printf("Bubble sort is selected.\n"); }
 
int main()
{
    char input = '!';
 
    while(input != 'q'){
        system("cls");
        printf("Choose a kind of sorting:\n(a) Quick sort\n(b) Bubble sort\n(q) Quit\n");
        input = getchar();
 
        if(input == 'a'){
            SORT(quick);
        }else if(input == 'b'){
            SORT(bubble);
        }else continue;
 
        system("pause");
    }
    return 0;
}
```
和剛說的一樣，把排序的演算法寫好。但是我不是用function pointer的形式，而是真真正正的function，透過MACRO做function的轉換。

比起class換來換去，這招強多了。另外，這招的performence非常的好，因為MACRO是前置處理器，在compile time就決定了。
