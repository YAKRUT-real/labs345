#include <iostream>
#include <cmath>
#include <iomanip>
using namespace std;

float cofe(float Tk,float Tsr,float r,float t)
{
    return (Tsr+(Tk-Tsr)*exp(-r*t));
}

int main()
{
    setlocale(LC_ALL, "Russian"); 
    float a[300][2];
    int i = 0;
    float Tk,Tsr,r;
    cout<<"Введите Tk Tsr r:"<<endl;
    cin>>Tk;
    cin>>Tsr;
    cin>>r;
    for(float t = 0;t<=60;t+=0.2)
    {
        a[i][0] = t;
        a[i][1] = cofe(Tk,Tsr,r,t);
        i++;
    }
    cout<<"x     y"<<endl<<"-----------"<<endl;
    for(i = 0;i<300;i++)
    {
        cout<<fixed<<setprecision(2)<<a[i][0]<<" "<<setprecision(6)<<a[i][1]<<endl;
    }
    return 0;
}
