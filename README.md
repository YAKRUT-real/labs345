# labs345
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
)
func main() {
	scanner := bufio.NewScanner(os.Stdin)
	scanner.Split(bufio.ScanWords)
	readInt := func() int {
		scanner.Scan()
		n, _ := strconv.Atoi(scanner.Text())
		return n
	}
	readInt64 := func() int64 {
		scanner.Scan()
		n, _ := strconv.ParseInt(scanner.Text(), 10, 64)
		return n
	}
	n := readInt()
	m := readInt()
	arr := make([]int64, n)
	for i := 0; i < n; i++ {
		arr[i] = readInt64()
	}
	prefix := make([]int64, n+1)
	for i := 0; i < n; i++ {
		prefix[i+1] = prefix[i] + arr[i]
	}
	var scorePavel, scoreVika int64
	var lastMovePavel, lastMoveVika int
	pos := 0
	isPavelTurn := true
	for pos < n {
		remaining := n - pos
		maxTake := m
		if remaining < maxTake {
			maxTake = remaining
		}
		bestK := -1
		bestSum := int64(-9e18)
		for k := 1; k <= maxTake; k++ {
			if (isPavelTurn && k == lastMovePavel) || (!isPavelTurn && k == lastMoveVika) {
				continue
			}

			sum := prefix[pos+k] - prefix[pos]

			if sum > bestSum || (sum == bestSum && k < bestK) {
				bestSum = sum
				bestK = k
			}
		}
		if isPavelTurn {
			scorePavel += bestSum
			lastMovePavel = bestK
		} else {
			scoreVika += bestSum
			lastMoveVika = bestK
		}

		pos += bestK
		isPavelTurn = !isPavelTurn
	}
	if scorePavel > scoreVika {
		fmt.Println(1)
	} else {
		fmt.Println(0)
	}
}
