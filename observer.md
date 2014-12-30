# Observer

繼上回介紹了template method之後，今天要講解的是observer，這是design pattern中用途極為廣泛的一種，也是我的最愛。在C#中，甚至直接把Observer給內建進去，從這就可以得知他的重要性
 
Observer是一種被動呼叫的典範，當有份資料會不定期的更新，而許多人都想要最新版的資料時，有兩種做法，最簡單的是寫個thread不斷去看，但這會造成CPU的busy wait；另一種做法就是當資料更新時，由資料擁有者去通知大家『嘿！寶貝！我變心了喔』，這時候吃醋的人就會跑過來看了，這樣的原理就是observer。

但是要小心，現實生活中要讓跑過來的眾人不要撞在同一個時間點，如果是網路模組也是，可能會造成bottleneck
 
按照慣例，先給一份C++的典型code
```
#include <iostream>
#include <cstdio>
#include <vector>
using namespace std;
 
class Observer
{
    public:
        virtual void update() = 0;
};
 
class Subject
{
    public:
        string data;
        Subject(){ data = "This is a subject data ver.1\n"; }
        ~Subject(){ for(int i=0; i<obs.size i++) delete(obs[i]); }        
		void attach(Observer *ob){ obs.push_back(ob); }        
		void setData(string s){ data = s; notify(); }    
	private:        
		vector<Observer*> obs;        
		void notify(){ for(int i=0; i<obs.size(); i++) obs[i]->update(); }
}; 
		
Subject *subject; 
		
class View : public Observer 
{    
	public:        
		View(){ str = "I am an observer\n"; }        
		void update(){ printf("%s\n", (str+subject->data).c_str()); }    
	private:        
		string str;
}; 

int main()
{    
	subject = new Subject();    
	View *view = new View();    
	subject->attach(view);    
	subject->setData("This is a subject data ver.2\n");    
	return 0;
}
```
從以上程式可以得知，當Subject(資料擁有者)的資料被不知道誰射了之後，就會通知(notify)對他有興趣(attached)的人：『陰道是直通內心的康莊大道，所以我變心了喔』。接著，View這個被戴綠帽的可憐蟲就會趕緊跑過來看看。

原理是，View在一開始就對Subject進行註冊，當Subject之後有變動就會通知。這在C語言當中一樣行的通，只是C語言沒有class和繼承的概念，全部都得用callback function來完成，也就是說『不用叫我，我會去叫你』

說再多都怪怪的，不如直接看段code 
```
#include <stdio.h>
#define MAX 5
typedef void (* Update) (void);
typedef void (* Notify) (void);
typedef void (* Attach) (Update);
typedef void (* SetData) (char *); 

struct Subject {    
	Update obs[MAX];    
	char *data;    
	int size;    
	Attach attach;    
	Notify notify;    
	SetData setData;
}; 

struct Subject subject;
void viewUpdate(void){ printf("I am an observer\n%s", subject.data); }
int i;
void subjectNotify(void){ for(i=0; i<subject.size i++) subject.obs[i](); }
void subjectAttach(Update upd){ subject.obs[subject.size++] = upd; }
void subjectSetData(char *str){ subject.data = str; subject.notify(); } 

int main()
{    
	subject.data = "This is a subject data ver.1\n";    
	subject.attach = subjectAttach;    
	subject.notify = subjectNotify;    
	subject.setData = subjectSetData;    
	subject.size = 0;     
	subject.attach(viewUpdate);    
	subject.setData("This is a subject data ver.2\n");    
	return 0;
} 
```
上述是C語言實現observer的方法，寫的是簡陋了點，但重點有點出來，就是資料觀察者將自己的function pointer丟給資料擁有者，當資料有變更時，擁有者就會透過pointer叫你過來看。

原理簡單，但在實際應用上可以千變萬化也可以透過domain socket或TCP/IP socket來完成這概念。
```