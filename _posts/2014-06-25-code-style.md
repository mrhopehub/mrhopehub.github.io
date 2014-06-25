---
layout: posts
title: "eclipse 常用"
---

<xmp class="prettyprint linenums">
#include <stdlib.h>
#include <time.h>
#include <math.h>
#include <assert.h>
#include <iostream>
using namespace std;

/**
 * 产生半开区间[left,right)的随机数，注意半开区间,left < right
 * 注意：需要先srand();
 */
int get_rand(int left, int right);

int main() {
    /* 不要忘了播种 */
	srand((unsigned) time(NULL));
	/*
	 * 先按你的1-1000
	 * 10个数，3个区间
	 * all_sum_cnt代表数的个数,section区间的个数
	 */
	int all_sum_cnt = 10, section = 3;
	/* 还有多少个数字，开始rest_cnt = all_sum_cnt - section */
	int rest_cnt = all_sum_cnt - section;
	for (int i = 1; i <= section; i++) {
		/* 每个区间产生数字的个数都是随机的，但必须有，也就是至少为1 */
		int tmp;
		/* 最后的区间个数就是剩下的数字个数，不用随机数 */
		if (i == section)
			tmp = rest_cnt;
		else
			/* 该区间要产生数字的个数 */
			tmp = get_rand(0, rest_cnt);
		/* 更新rest_cnt */
		rest_cnt -= tmp;
		cout << "区间[" << pow(10, i - 1) << "-" << pow(10, i) << ")生成" << tmp + 1
				<< "个数字:" << endl;
		/* 产生[10^(i-1)-10^i)区间(tmp + 1)个数，可能重复 */
		for (int j = 0; j < (tmp + 1); j++) {
			cout << get_rand(pow(10, i - 1), pow(10, i)) << endl;
		}
	}
	return 0;
}

int get_rand(int left, int right) {
	assert(left<right);
	int result;
	int offset = right - left;
	/* 产生[0,offset)区间随机数 */
	result = rand() % offset;
	/* 再加上left */
	result += left;
	return result;
}

</xmp>