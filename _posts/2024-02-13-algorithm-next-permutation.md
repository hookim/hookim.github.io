---
layout: post
title: Next Permutation 
date: 2024-02-13 16:05:10
description: 순열을 효율적으로 푸는 방법!
tags: 
categories: 알고리즘
---
## 개요

순열을 풀 때 직관적으로 나이브하게 생각한다면 재귀를 사용해서 풀 수 있다. 그런데 해당 방식은, 예를들어, N개의 순열을 만드는 경우라면 O(N! * N)의 시간복잡도가… 엄청나다. `N!`개의 모든 가능한 경우의 수에 대해서 N개를 선택한 경우 길이가 N인 선택 배열을 순회하면서 선택한 순열을 생각하는것! 그러나 O(N)으로 순열을 풀 수 있는 매우 혁신적인 방법이 있다고 한다. 

## 설명

1. 기존의 방식 
    
    ```cpp
    perm(int pick){
    	if(pick == N){
    		// 보통 선택의 여부는 boolean 배열을 사용하니까. 이를 전부 순회하면서 체크하기 여기서 O(N)
    	}
    	
    	// 이 모든게 재귀를 돌면서 N!개의 가지치기가 일어난다.
    	for(int i = 0; i < all_possible_numbers; i++){
    		if(pick[i]) continue;
    
    		pick[i] = true;
    		perm(pick+1);
    		pick[i] = false; 
    	}
    }
    ```
    
    이런식으로 재귀를 활용해서! 쉽지만 느리다. 
    
2. Next Permutation 방식
    1. NextPermutation을 사용하면 순서대로 다음 순열을 탐색하게 된다. ex) 123→ 132 → 213 → 231 → … 
    2. 순열을 거꾸로 봤을 때 뒷부분에서부터 차례로 증가하거나 같은 부분을 찾는다. ex) 5438762 여기서 거꾸로 봤을 때 2 < 6 < 7 < 8이 된다. 그리고 3에서 그 패턴이 끊긴다. 이를 pivot이라고 부른다. 
    3. 찾은 패턴에서 pivot보다 값이 큰 가장 오른쪽 부분을 찾아 swap한다. ex) 543 8762 → 546 8732
    4. 그리고 해당 패턴의 구간을 뒤집어 버린다. ex) 546 2378
    5. 만약에 모든 순열이 전부 역순이라면 순열의 맨 마지막에 왔다는 뜻이다. 그렇기 때문에 따로 플래그를 둬서 종료시킨다. 
    
    ```cpp
    bool nextPermutation(int[] arr){
    	int i = arr.size()-1; // 뒷부분에서부터 시작 
    	int j = i;
    	
    	while(i > 0 && arr[i-1] >= arr[i]){ // 패턴찾기
    		i--;
    	} 
    	if(i == 0) // 만약 주어진 모든 순열이 전부 역순이라면? 마지막이기 때문에 플래그 표시
    		return false;
    	while(arr[i-1] >= arr[j]){ // pivot보다 큰 수를 패턴안에서 찾기 
    		j--;
    	} 
    	swap(arr, i-1, j); // 두 수를 교체하기 
    	j = arr.size()-1; 
    
    	while(i < j){ // 패턴부분을 전부 뒤집기 
    		swap(arr, i++, j--);
    	}
    	return true;
    }
    ```
    

## 참고

[Next Permutation - GeeksforGeeks](https://www.geeksforgeeks.org/next-permutation/)
