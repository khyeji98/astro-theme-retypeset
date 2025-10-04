---
title: Leetcode 39. Search Insert Position
published: 2025-10-04
tags:
  - Algorithm
  - Leetcode
toc: false
lang: ko
abbrlink : leetcode-39
---

# 이진 탐색(Binary Search)

- **이진(Binary)** : "둘로 나누는", "두가지 상태로 구분하는"  
  - 컴퓨터에서 바이너리는 0과 1처럼 두개의 값으로 상태를 나누는 것을 의미함  
  - 즉, 문제 풀이를 위해 탐색 공간을 둘로 나눠 탐색하며 범위를 줄여나가는 것을 의미함

---

이진 탐색 첫 문제([LeetCode 39. Search Insert Position](https://leetcode.com))를 풀기 위해 키워드를 뜯어보고,  
이때 들었던 생각은 "둘로 나눌 기준(divider)이 필요하겠구나!"  
그렇게 이진 탐색 첫 문제를 무모하게 풀기 시작했습니다  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbolKB1%2FbtsQ1ZpP9qF%2FAAAAAAAAAAAAAAAAAAAAAMTiUiXqjQfJdWwIf1hhwtJacXP4QdpEGRnEyGE43UJ6%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1761922799%26allow_ip%3D%26allow_referer%3D%26signature%3DhSyFL10nHb5ofqY%252FJbrq1oiaA6E%253D)  
  
이런 경우, 저런 경우 따져가며 코드를 짜다보니 if의 if… elseif… 가 생겨나고 결국 TLE가 발생해버렸습니다..  
결국 약간의 힌트를 얻고, 깨달았습니다 “범위를 줄여나가는 것보다 나누는 기준선에 초점을 두었구나”
  
## 그렇게 약간의 힌트를 곁들인 최종 코드(+ 개선후)

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        left, right = 0, len(nums) - 1

        while left <= right:
            mid = (left + right) // 2

            if nums[mid] == target:
                return mid
            elif nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1

        return left
```

left와 right를 통해 범위를 설정해주고, 반복문을 순회할 때마다 범위의 중간인 mid를 통해 범위를 새로 설정해주고 mid 값을 새롭게 설정합니다.
  
사실 제 기준에서 가장 헷갈렸던 부분은 “왜 마지막에 left만 반환해도 될까?”였습니다.

```  
# TestCase
nums = [1, 3, 5, 6]  
target = 2
```

위 케이스를 기준으로 설명해보자면, left와 right 값이 동일해지는 순간 `right = mid - 1`가 실행되고 right는 left보다 값이 작아져 더이상 반복문을 순회하지 않습니다.  
그리고 이를 통해 알 수 있는 것은 “left == right == mid 인 경우, nums[mid] 값은 target보다 컸다”
  
> **즉 nums에는 target과 같은 값이 없었고,**  
> **target은 nums[left]에 insert 되어야 한다.**