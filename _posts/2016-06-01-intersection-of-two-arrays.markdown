---
layout: "post"
title: "从easy刷起-LeetCode"
---
# <font color="#e3796b">Question</font>
[找出两个int数组的相同元素](https://leetcode.com/problems/intersection-of-two-arrays-ii/)

# <font color="#e3796b">My Answer</font>
```JAVA
public class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        sort(nums1);
        sort(nums2);
        List<Integer> result = new ArrayList<Integer>();
        int length1 = nums1.length;
        int length2 = nums2.length;
        int position1 = 0;
        int position2 = 0;
        for(; position1 < length1; position1++){
            for(; position2 < length2; position2++){
                if(nums1[position1] < nums2[position2]){
                    break;
                }
                if(nums1[position1] == nums2[position2]){
                    result.add(nums1[position1]);
                    position2++;
                    break;
                }
            }
        }
        int resultSize = result.size();
        int[] resultArray = new int[resultSize];
        for(int i = 0; i < resultSize; i++){
            resultArray[i] = result.get(i);
        }
        return resultArray;
    }

    //sort the specified array into ascending numerical order
    private void sort(int[] nums){
        int length = nums.length;
        for(int i = 0; i < length; i++){
            int minPos = i;
            for(int j = i; j < length; j++){
                if(nums[j] < nums[minPos]){
                    minPos = j;
                }
            }
            int min = nums[minPos];
            nums[minPos] = nums[i];
            nums[i] = min;
        }
    }
}

```

# <font color="#e3796b">More</font>
网站的60个测试用例跑完需要12ms，感觉没有几个人写的算法可以比我的再慢一点。。。

用Arrays.sort(int[] a);函数代替我自己的这个sort函数，用例跑完竟然只需要6ms。。。

还有，数组转化成Set，List，用他们的函数去操作呢？用Set不可以，因为Set中不可以有重复的元素。用ArrayList去做，测试用例跑完竟然要用1104ms，太吓人了！！！代码如下：

```JAVA
public class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        List<Integer> l1 = new ArrayList<Integer>();
        List<Integer> l2 = new ArrayList<Integer>();
        List<Integer> result = new ArrayList<Integer>();
        for(int i : nums1){
            l1.add(i);
        }
        for(int i : nums2){
            l2.add(i);
        }

        for(int i : l1){
            for(int j : l2){
                if (l2.contains(i)){
                    result.add(i);
                    l2.remove(new Integer(i));
                    break;
                }
            }
        }
        int[] returnResult = new int[result.size()];
        int pos = 0;
        for(int i : result){
            returnResult[pos++] = i;
        }
        return returnResult;
    }
}

```

# <font color="#e3796b">Resource: Solution For LeetCode</font>
[Leetcode question&solutions in Java](http://codeluli.blogspot.com/)
[Github-hoael-leetcode](https://github.com/haoel/leetcode)
