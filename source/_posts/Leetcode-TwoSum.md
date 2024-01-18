---
title: leetcode-TwoSum算法
date: 2021-05-14 01:47:47
tags: Leetcode

---



Two Sum
Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.
You may assume that each input would have exactly one solution, and you may not use the same element twice.
You can return the answer in any order.

Example 1:
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Output: Because nums[0] + nums[1] == 9, we return [0, 1].

Example 2:
Input: nums = [3,2,4], target = 6
Output: [1,2]

Example 3:
Input: nums = [3,3], target = 6
Output: [0,1]

Constraints:
2 <= nums.length <= 103
-109 <= nums[i] <= 109
-109 <= target <= 109
Only one valid answer exists.

```java
package leetcode;

import java.util.Arrays;

public class AddCount {
    public static void main(String[] args) {
        int numsArray[] = {2,7,11,15};
        int target = 9;
        System.out.println(Arrays.toString(twoSum(numsArray, target)));
    }
  
    public static int[] twoSum(int[] nums, int target) {
        int result[] = new int[2];
        for (int i = 0; i < nums.length-1; i++) {
            for (int j = 1+i; j < nums.length; j++) {
                if (nums[i]+nums[j]==target){
                    result[0]=i;
                    result[1]=j;
                    return result;
                }
            }
        }
        return null;
    }
}

public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[] { map.get(complement), i };
        }
        map.put(nums[i], i);
    }
    throw new IllegalArgumentException("No two sum solution");
}
```