---
layout: post
title: 햄버거 다이어트 문제풀이
date: 2024-02-11 12:12:39
description: 실습 과제
tags: 
categories: 알고리즘
---

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

/**
 * 이것도 재귀를 돌면서 햄버거 선택, 미선택 두개의 가지치기로 나눠가면서 완전탐색 느낌으로 간다.
 * 그런데 재료를 선택하는 방법 말고 재료를 빼는 방법을 선택한다.
 * 그래서 빼는 흐름이면 그냥 재귀 호출하고 거기서 칼로리 조건보다 이하가 됐는지 확인하기만 하면 된다.
 * 이 경우 일단 탈출조건(칼로리 조건이하)가 됐다면 그 이후는 더 살펴볼 필요가 없어진다. 이후 탐색에서는 점수가 줄어들거나 유지될 뿐이기 때문이다.
 */

public class 실습023_SWEA_5215_햄버거다이어트 {
    static int T, N, L, tempAns;
    static int[] tastes = new int[20];
    static int[] calories = new int[20];
    static StringBuilder ans = new StringBuilder("");
    public static void search(int start, int taste, int calrory){
        if(calrory <= L){ // 칼로리가 특정 조건 이하로 됐을 때
            if(tempAns < taste) // 업데이트
                tempAns = taste;
            return;
        }

        for(int i = start; i < N; i++){ // 다음 재료 탐색
            search(i+1, taste - tastes[i], calrory - calories[i]); // 현재 재료를 뺀다.
        }
    }
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in)); // 빠른입력
        T = Integer.parseInt(br.readLine());
        for(int t = 1 ; t <= T; t++){
            StringTokenizer st = new StringTokenizer(br.readLine());
            N = Integer.parseInt(st.nextToken());
            L = Integer.parseInt(st.nextToken());
            int totalTaste = 0, totalCalories = 0;
            for(int i = 0; i < N ; i++){
                st = new StringTokenizer(br.readLine());
                tastes[i] = Integer.parseInt(st.nextToken());
                calories[i] = Integer.parseInt(st.nextToken());
                totalTaste += tastes[i];
                totalCalories += calories[i];
            }
            tempAns = 0;
            search(0, totalTaste, totalCalories);
            ans.append("#").append(t).append(" ").append(tempAns).append("\n");
        }
        System.out.println(ans);
    }
}
```
