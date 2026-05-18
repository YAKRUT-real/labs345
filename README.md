package main

import (
	"fmt"
)
func f(x float64) float64{
	if x<=-3.0{
		return -x - 6.0
	} else if x <= -1.0{
		return -1.5*x*x-4.5*x-3.0
	} else if x <= 0.0{
		return -3.0*x-3.0
	} else if x<=3.0{
		return (2.0/9.0)*x*x*x-3.0
	} else{
		return -1.5*x+7.5
	}
	}
func main(){
	var x_start, x_end, dx float64
	fmt.Print("Введине Хнач, Хкон, dx: ")
	fmt.Scan(&x_start, &x_end, &dx)
	fmt.Println("\n   x       y\n")
	fmt.Println("----------------\n")
	oshibka := 1e-9
	for x := x_start; x<= x_end+oshibka; x+=dx{
		fmt.Printf("%8.4f%10.4f\n",x,f(x))
	}
}
