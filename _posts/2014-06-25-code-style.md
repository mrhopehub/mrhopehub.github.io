---
layout: posts
title: "编码风格示例"
---
# 编码风格示例
### 随机整数
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
	srand((unsigned) time(NULL)); /* 不要忘了播种 */
	/*
	 * 先按你的1-1000
	 * 10个数，3个区间
	 * all_sum_cnt代表数的个数,section区间的个数
	 */
	int all_sum_cnt = 10, section = 3;
	/* 还有多少个数字，开始rest_cnt = all_sum_cnt - section */
	int rest_cnt = all_sum_cnt - section;
	for (int i = 1; i <= section; i++) {
		int tmp; /* 每个区间产生数字的个数都是随机的，但必须有，也就是至少为1 */
		if (i == section) /* 最后的区间个数就是剩下的数字个数，不用随机数 */
			tmp = rest_cnt;
		else
			tmp = get_rand(0, rest_cnt); /* 该区间要产生数字的个数 */
		rest_cnt -= tmp; /* 更新rest_cnt */
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
	result = rand() % offset; /* 产生[0,offset)区间随机数 */
	result += left; /* 再加上left */
	return result;
}

</xmp>
### 随机浮点数
<xmp class="prettyprint linenums">
#include <stdlib.h>
#include <time.h>
#include <math.h>
#include <assert.h>
#include <iostream>
using namespace std;
 
/**
 * 产生闭区间[left,right]的随机数，注意是闭区间,left < right
 * 注意：需要先srand();
 */
float get_rand(float left, float right);
 
int main() {
	srand((unsigned) time(NULL)); /* 不要忘了播种 */

	/* 产生[left,right]区间的浮点数
	 * 首先在[0,500]区间产生随机数left
	 * 在[501,1000]区间产生随机数right
	 */
	float left = get_rand(0, 500);
	float right = get_rand(501,1000);

	for (;;)
	{
		/* 如果产生的随机数大于right则结束循环 */
		if (get_rand(left, right) > right)
		{
			printf("over!\n");
			break;
		}
	}

    return 0;
}
 
float get_rand(float left, float right) {
	/* return ((right - left) * ((float)rand() / RAND_MAX)) + left; */
    assert(left < right);
	float result;
	float len = right - left;
	float rand_rate = (float)rand() / RAND_MAX;
	result = (len * rand_rate) + left;
	printf("%11f   %11f   %11f   %11f\n", len, rand_rate, result, right);
	return result;
}

</xmp>