# labs345
#include <iostream>
#include <cmath>
#include <vector>
#include <iomanip>

using namespace std;

double modelTemp(double t, double T0, double Tenv, double r) {
    return Tenv + (T0 - Tenv) * exp(-r * t);
}

void cofe(const vector<double>& time, const vector<double>& T_exp, 
          vector<double>& x, vector<double>& y) {
    int n = time.size();
    x.resize(n - 1);
    y.resize(n - 1);
    
    for (int i = 0; i < n - 1; i++) {
        double dt = time[i + 1] - time[i];
        y[i] = (T_exp[i + 1] - T_exp[i]) / dt;
        x[i] = (T_exp[i] + T_exp[i + 1]) / 2;
    }
}

void coeff(const vector<double>& x, const vector<double>& y, double& a, double& b) {
    int n = x.size();
    double sum_x = 0, sum_y = 0;
    for (int i = 0; i < n; i++) {
        sum_x += x[i];
        sum_y += y[i];
    }
    double mean_x = sum_x / n;
    double mean_y = sum_y / n;

    double xy = 0, x2 = 0;
    for (int i = 0; i < n; i++) {
        xy += (x[i] - mean_x) * (y[i] - mean_y);
        x2 += (x[i] - mean_x) * (x[i] - mean_x);
    }

    a = xy / x2;
    b = mean_y - a * mean_x;
}

double approx(double x, double a, double b) {
    return a * x + b;
}

double korrel(const vector<double>& x, const vector<double>& y, double a, double b) {
    int n = x.size();
    double sum_y = 0;
    for (int i = 0; i < n; i++) {
        sum_y += y[i];
    }
    double mean_y = sum_y / n;

    double SS_res = 0, SS_tot = 0;
    for (int i = 0; i < n; i++) {
        double y_pred = approx(x[i], a, b);
        SS_res += (y[i] - y_pred) * (y[i] - y_pred);
        SS_tot += (y[i] - mean_y) * (y[i] - mean_y);
    }
    return 1 - SS_res / SS_tot;
}

int main() {
    setlocale(LC_ALL, "Russian");
    double T0, Tenv_given, r_given;
    cout << "Введите начальную температуру кофе T0 (°C): ";
    cin >> T0;
    cout << "Введите температуру окружающей среды Tsr (°C): ";
    cin >> Tenv_given;
    cout << "Введите коэффициент остывания r (1/мин): ";
    cin >> r_given;

    int n;
    cout << "Введите количество измерений: ";
    cin >> n;

    vector<double> time(n), T_exp(n);
    cout << "Введите время (мин) и температуру (°C):" << endl;
    for (int i = 0; i < n; i++) {
        cout << i + 1 << ": ";
        cin >> time[i] >> T_exp[i];
    }

    vector<double> T_mid, dTdt;
    cofe(time, T_exp, T_mid, dTdt);

    double a, b;
    coeff(T_mid, dTdt, a, b);
    double R2 = korrel(T_mid, dTdt, a, b);

    double r_found = -a;
    double Tenv_found = b / r_found;

    vector<double> T_model(n);
    for (int i = 0; i < n; i++) {
        T_model[i] = modelTemp(time[i], T0, Tenv_given, r_given);
    }

    double T_mean = 0;
    for (int i = 0; i < n; i++) T_mean += T_exp[i];
    T_mean /= n;

    double SS_res = 0, SS_tot = 0, kor = 0, maxDev = 0;
    for (int i = 0; i < n; i++) {
        double dev = T_exp[i] - T_model[i];
        SS_res += dev * dev;
        SS_tot += (T_exp[i] - T_mean) * (T_exp[i] - T_mean);
        kor += dev * dev;
        if (fabs(dev) > maxDev) maxDev = fabs(dev);
    }
    kor = sqrt(kor / n);
    double R2_model = 1 - SS_res / SS_tot;

    cout << "\nСравнение данных из эксперимента и данных из модели" << endl;
    cout << "|  № | t, мин | T_эксп, °C | T_модель, °C | Отклонение |" << endl;

    for (int i = 0; i < n; i++) {
        double dev = fabs(T_exp[i] - T_model[i]);
        cout << "| " << setw(2) << i + 1 << " | "
             << setw(6) << fixed << setprecision(2) << time[i] << " | "
             << setw(10) << T_exp[i] << " | "
             << setw(12) << T_model[i] << " | "
             << setw(10) << dev << " |" << endl;
    }

    cout << "\nСкорость остывания - регрессия" << endl;
    cout << "| T_ср, °C | dT/dt_эксп | dT/dt_модель |" << endl;

    for (int i = 0; i < n - 1; i++) {
        double model_dt = approx(T_mid[i], a, b);
        cout << "| " << setw(8) << T_mid[i] << " | "
             << setw(10) << dTdt[i] << " | "
             << setw(12) << model_dt << " |" << endl;
    }

    cout << "\nСтатистические данные:" << endl;
    cout << "Коэффициент детерминации (R²) модели: " << setprecision(4) << R2_model << endl;
    cout << "Коэффициент детерминации регрессии (R²): " << setprecision(4) << R2 << endl;
    cout << "Среднеквадратичная ошибка (kor): " << kor << " °C" << endl;
    cout << "Максимальное отклонение: " << maxDev << " °C" << endl;

    cout << "\nПараметры из регрессии:" << endl;
    cout << "Заданный r: " << r_given << " 1/мин" << endl;
    cout << "Найденный r: " << r_found << " 1/мин" << endl;
    cout << "Заданная Tsr: " << Tenv_given << " °C" << endl;
    cout << "Найденная Tsr: " << Tenv_found << " °C" << endl;

    cout << "\nИтоговая оценка модели:" << endl;
    if (fabs(r_found - r_given) < 0.01 && R2_model > 0.95) {
        cout << "Модель адекватна: найденные параметры близки к заданным." << endl;
    } else if (R2_model > 0.9) {
        cout << "Модель хорошо описывает данные, но параметры отличаются." << endl;
    } else {
        cout << "Модель плохо описывает данные. Возможны иные виды воздействия." << endl;
    }
    cout << "\n";
    return 0;
}
