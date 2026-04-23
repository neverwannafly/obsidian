[[leetcode medium]]  [[math]] [[factorization]]
## Intuition
- the approach seems pretty intuitive, get all the factors of a number in an optimized way and then just sum the factors of the number that have 4 factors
- To find factors we only need to enumerate till sqrt(N) hence we can solve this problem in O(N sqrt(N))


## Solution
```cpp
// @leet start

int divisorSum(int num) {
  std::set<int> divs;
  for (int i=1; i*i <= num; i++) {
    if (num % i == 0) {divs.insert(i); divs.insert(num/i);}
  }

  if (divs.size() != 4) return 0;

  int sum = 0;
  for (auto it = divs.begin(); it != divs.end(); it ++) {
    sum += *it;
  }
  return sum;
}

class Solution {
public:
    int sumFourDivisors(vector<int>& nums) {
      long ans = 0;
      for (int i=0; i<nums.size(); i++) {
        ans += divisorSum(nums[i]);
      }

      return ans;

    }
};
// @leet end

```