> <math.h> 中存放了 C 中的数学相关的很多函数

* double sqrt(double arg)：arg 开平方

* double pow(double a,double b)：求 a 的 b 次幂

* double cos(double arg)：arg 的 cos 值

  > ```c
  > #define PI 3.1415926
  > int main(){
  >   // 求出度的单元值
  >  	int degreeUnit = PI/180.0;
  >   double _90 = 90 * degreeUnit;
  >   // 输出 90 度角的 cos 值
  >   printf("",cos(_90));
  >   return 0;
  > }
  > ```
  >
  > 

