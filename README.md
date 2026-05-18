#include <iostream>
#include <cmath>
#include <vector>
#include <iomanip>

using namespace std;

float modelTemp(float t, float Tk, float Tsr, float r) {
    return Tsr + (Tk - Tsr) * exp(-r * t);
}

void cofe(const vector<float>& time, const vector<float>& T_exp,
          vector<float>& x, vector<float>& y) {
    int n = time.size();
    x.resize(n - 1);
    y.resize(n - 1);
    
    for (int i = 0; i < n - 1; i++) {
        float dt = time[i + 1] - time[i];
        y[i] = (T_exp[i + 1] - T_exp[i]) / dt;
        x[i] = (T_exp[i] + T_exp[i + 1]) / 2;
    }
}

void coeff(const vector<float>& x, const vector<float>& y, float& a, float& b) {
    int n = x.size();
    float sum_x = 0, sum_y = 0;
    for (int i = 0; i < n; i++) {
        sum_x += x[i];
        sum_y += y[i];
    }
    float mean_x = sum_x / n;
    float mean_y = sum_y / n;
    
    float xy = 0, x2 = 0;
    for (int i = 0; i < n; i++) {
        xy += (x[i] - mean_x) * (y[i] - mean_y);
        x2 += (x[i] - mean_x) * (x[i] - mean_x);
    }
    
    a = xy / x2;
    b = mean_y - a * mean_x;
}

float approx(float x, float a, float b) {
    return a * x + b;
}

float korrel(const vector<float>& x, const vector<float>& y, float a, float b) {
    int n = x.size();
    float sum_y = 0;
    for (int i = 0; i < n; i++) {
        sum_y += y[i];
    }
    float mean_y = sum_y / n;
    
    float SS_res = 0, SS_tot = 0;
    for (int i = 0; i < n; i++) {
        float y_pred = approx(x[i], a, b);
        SS_res += (y[i] - y_pred) * (y[i] - y_pred);
        SS_tot += (y[i] - mean_y) * (y[i] - mean_y);
    }
    return 1 - SS_res / SS_tot;
}

int main() {
    setlocale(LC_ALL, "Russian");
    
    float Tk, Tsr, r;
    int n;
    
    cout << "Введите Tk, Tsr, r: ";
    cin >> Tk >> Tsr >> r;
    cout << "Введите количество точек n: ";
    cin >> n;
    
    vector<float> time(n), T_exp(n);
    float step = 60.0 / (n - 1);
    
    cout << "\nГенерация модели:" << endl;
    for (int i = 0; i < n; i++) {
        time[i] = i * step;
        T_exp[i] = modelTemp(time[i], Tk, Tsr, r);
        cout << "t=" << fixed << setprecision(2) << time[i] 
             << " T=" << T_exp[i] << endl;
    }
    
    vector<float> x, y;
    cofe(time, T_exp, x, y);
    
    float a, b;
    coeff(x, y, a, b);
    float R2 = korrel(x, y, a, b);
    
    cout << "\nАппроксимирующая прямая: dT/dt = " << a << " * T + " << b << endl;
    cout << "Коэффициент детерминации R² = " << R2 << endl;
    
    return 0;
}
