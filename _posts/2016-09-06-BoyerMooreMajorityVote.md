---
title: "Boyer-Moore Majority Vote"
tags: [算法]
---

Leetcode 229

### 问题描述

给定一个长度为 n的数组:
int[] nums
其中有一个数，它出现的次数大于n/2，称为主要元素，找到它。

### 基本思想

比较直观的解释：在数组中找到两个不相同的元素并删除它们，不断重复此过程，直到数组中元素都相同，那么剩下的元素就是主要元素。
思想并不复杂，但是要凭空想出这个算法来也不是件容易的事。另外，给我们的是数组，直接在里面删除元素是很费时的。取而代之，可以利用一个计数变量来实现。

代码如下：

{% highlight c++ %}
int majorityElement(vector<int>& nums){
 	int count=0,major=0;
	for(int num: nums){
		if(count == 0) major = num;
		if(major == num) count++;
		else count--;
	}
	return major;
}
{% endhighlight %}

对于上面的代码：
先随意确定一个候选元素，count是候选元素的计数，当遇到一个跟候选元素不同的元素时，两者数量上抵消一个，count减1。一旦count变成0，就重新找一个候选元素。
当遇到一个与候选元素不同的元素时，就要抵消。对于候选元素和当前元素，可能存在两种情况：1）两者中有一个正好是主要元素；2）两者都不是主要元素。
对于情况1)，抵消过后，主要元素还是主要元素；对于情况2），可以说主要的元素的地位得到了巩固。所以算法最终能找到主要元素。
另外，若这样的元素不一定存在，那么当我们找到这样一个元素时，还要进一步验证一下它是否满足条件。很简单，再遍历一遍，统计它的出现次数。

### 扩展



> Leetcode 229 - Majority Element II  

> Given an integer array of size n, find all elements that appear more than ⌊ n/3 ⌋ times. The algorithm should run in linear time and in O(1) space.

我们的Boyer-Moore算法思路，在这里依然可用，但需要些改动：
1)满足条件的元素最多有两个，那么需要两组变量。上面的count, major变成了count1, major1; count2, major2。
2)选出的两个元素，需要验证它们的出现次数是否真的满足条件。

代码如下：

{% highlight c++ %}
vector<int> majorityElement(vector<int>& nums) {
	int count1 = 0, count2 = 0, major1 = 0, major2 = 0;
	for(int num : nums){
		if(major1 == num) count1++;
		else if(major2 == num) count2++;
		else if(!count1) {
			major1 = num;
			count1++;
		}
		else if(!count2) {
			major2 = num;
			count2++;
		}
		else {
			count1--;
			count2--;
		}	
	}    
	int size = nums.size();
	count1 = 0;
	count2 = 0;
	for(int num: nums){
		count1+=(num==major1?1:0);
		count2+=(num==major2?1:0);
	}
	vector<int> res;
	if(count1 > size/3) res.push_back(major1);
	if(major1 != major2 && count2 > size/3) res.push_back(major2);
	return res;
}
{% endhighlight %}