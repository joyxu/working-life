# Template Method

Template method是design pattern中的基礎，運用介面的方式將一個任務的執行流程固定，但將實作細節交由使用者。

這邊展示一段傳統的C++來描述template method的原理
```
#include <iostream>
#include <cstdio>
using namespace std;
 
class DriveAutomobile{
    public:
        virtual void step1(){ printf("上車\n"); }
        virtual void step2(){ printf("校正後照鏡\n"); }
        virtual void step3(){ printf("繫安全帶\n"); }
        virtual void step4(){ printf("發動開走\n"); }
 
        void go(){
            step1(); step2(); step3(); step4();
        }
};
 
class DrivePlane : public DriveAutomobile{
    public:
        void step1(){ printf("架艦橋 & 登機\n"); }
        void step2(){ printf("地勤檢查\n"); }
        void step4(){ printf("加速衝刺 & 起飛\n"); }
};
 
int main()
{
    printf("這在開車!!\n");
    DriveAutomobile *car = new DriveAutomobile();
    car->go();
 
    printf("\n\n這是開飛機!!\n");
    DriveAutomobile *plane = new DrivePlane();
    plane->go();
    return 0;
}
```

題外話，不用challenge為什麼讓開飛機去繼承開自動車這邏輯，自動車在日文漢字中描述的是有輪子的交通工具。飛機有輪子吧？

有吧！ 那不就結了嗎？

唯一比較有問題的是，物件名稱應該是名詞我卻用了動詞。
 
以上程式碼非常標準，是典型的template method，在C語言當中也可以實現流程固定的邏輯，但稍微難看了點，請見以下
```
#include <stdio.h>
 
typedef void (* Step1) (void);
typedef void (* Step2) (void);
typedef void (* Step3) (void);
typedef void (* Step4) (void);
 
struct Drive {
    Step1 one;
    Step2 two;
    Step3 three;
    Step4 four;
};
 
void CarStep1(void){ printf("上車\n"); }
void CarStep2(void){ printf("校正後照鏡\n"); }
void CarStep3(void){ printf("繫安全帶\n"); }
void CarStep4(void){ printf("發動開走\n"); }
 
 
void PlaneStep1(void){ printf("架艦橋 & 登機\n"); }
void PlaneStep2(void){ printf("地勤檢查\n"); }
void PlaneStep3(void){ printf("繫安全帶\n"); }
void PlaneStep4(void){ printf("加速衝刺 & 起飛\n"); }
 
void DriveAutomobile(struct Drive drive)
{
    drive.one(); drive.two(); drive.three(); drive.four();
}
 
int main()
{
    printf("這在開車!!\n");
    struct Drive car, plane;
    car.one = CarStep1; car.two = CarStep2;
    car.three = CarStep3; car.four = CarStep4;
    DriveAutomobile(car);
 
    printf("\n\n這是開飛機!!\n");
    plane.one = PlaneStep1; plane.two = PlaneStep2;
    plane.three = PlaneStep3; plane.four = PlaneStep4;
    DriveAutomobile(plane);
    return 0;
}
```
運用callback function的方式也可以使C語言做到近似於物件導向的概念，而這template method也很標準的實現了。其實，透過callback function，可以讓C語言做出很多design pattern。

而上面的C範例其實可以寫的更封裝一點以便更貼近物件導向語言，例如：寫一個constructor去將填callback function的部分給包裝，使填寫step1234的邏輯可以不被user知道，但這不影響template method in C的精神，這邊就沒有要大費周章去改了。

`