# Container With Most Water

[Problem Description](https://leetcode.cn/problems/container-with-most-water/description/)

[Explain](https://www.bilibili.com/video/BV1rv4y1H7o6/?p=180)

```java
public int maxArea(int[] height) {
    int i = 0;
    int j = height.length - 1;
    int max = 0;
    while (i < j) {
        if (height[i] < height[j]) {
            max = Math.max(max, height[i] * (j - i));
            i++;
        } else {
            max = Math.max(max, height[j] * (j - i));
            j--;
        }
    }
    return max;
}
```

# Climbing Stairs

[Problem Description](https://leetcode.cn/problems/climbing-stairs/description/)

```java
public static int climbStairs(int n) {
    rec(n);
    return count;
}

public static int count = 0;

public static void rec(int n) {
    if (n < 0) {
        return;
    }

    if (n == 0) {
        count++;
        return;
    }

    rec(n - 1);
    rec(n - 2);
}
```

# Climbing Stairs

[Explain](https://www.bilibili.com/video/BV1eg411w7gn?p=5&spm_id_from=pageDriver&vd_source=2b0f5d4521fd544614edfc30d4ab38e1)

```java
private static HashMap<Integer, Integer> hashMap = new HashMap<>();

static {
    hashMap.put(1, 1);
    hashMap.put(2, 2);
}

public static int climbStairs(int n) {
    if (hashMap.get(n) != null) {
        return hashMap.get(n);
    } else {
        int r = climbStairs(n - 1) + climbStairs(n - 2);
        hashMap.put(n, r);
        return r;
    }
}
```

# Climbing Stairs

```java
public static int climbStairs(int n) {
    if (n <= 2) {
        return n;
    }
    
    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    
    return dp[n];
}
```

# Climbing Stairs

```java
public static int climbStairs(int n) {
    if (n <= 2) {
        return n;
    }
    
    int prePre = 1;
    int pre = 2;
    for (int i = 3; i <= n; i++) {
        int cur = prePre + pre;
        prePre = pre;
        pre = cur;
    }
    
    return pre;
}
```

# Move Zeroes with Non-Sort

[Problem Description](https://leetcode.cn/problems/move-zeroes/description/)

[Explain](https://www.bilibili.com/video/BV1eg411w7gn?p=9&spm_id_from=pageDriver&vd_source=2b0f5d4521fd544614edfc30d4ab38e1)

```java
public static void moveZeroes(int[] nums) {
    int k = 0;
    for (int num : nums) {
        if (num != 0) {
            nums[k++] = num;
        }
    }
    
    while (k < nums.length) {
        nums[k++] = 0;
    }
}
```

# Move Zeroes (Non-Sort)

```java
public static void moveZeroes(int[] nums) {
    int l = 0;
    int r = 0;
    
    while (r < nums.length) {
        if (nums[r] != 0) {
            int t = nums[l];
            nums[l] = nums[r];
            nums[r] = t;
            l++;
        }
        r++;
    }
}
```

# Move Zeroes (Sort)

```java
public static void moveZeroes(int[] nums) {
    for (int low = 1; low < nums.length; low++) {
        int idx = low;
        int val = nums[low];
        
        while (idx > 0 && compare(nums[idx - 1], val) > 0) {
            nums[idx] = nums[idx - 1];
            idx--;
        }
        
        if (idx != low) {
            nums[idx] = val;
        }
    }
}

public static int compare(int num1, int num2) {
    if (num1 == 0) {
        return 1;
    }
    
    if (num2 == 0) {
        return -1;
    }
    
    return num1 - num2;
}
```

# Find All Numbers Disappeared in an Array

[Problem Description](https://leetcode.cn/problems/find-all-numbers-disappeared-in-an-array/description/)

[Explain](https://www.bilibili.com/video/BV1eg411w7gn/?p=10&spm_id_from=pageDriver&vd_source=2b0f5d4521fd544614edfc30d4ab38e1)

```java
public static List<Integer> findDisappearedNumbers(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        int idx = Math.abs(nums[i]) - 1;
        int val = -Math.abs(nums[idx]);
        nums[idx] = val;
    }
    
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < nums.length; i++) {
        if (nums[i] > 0) {
            result.add(i + 1);
        }
    }
    
    return result;
}
```

# Find All Numbers Disappeared in an Array

```java
public static List<Integer> findDisappearedNumbers(int[] nums) {
    int len = nums.length;
    for (int i = 0; i < len; i++) {
        int idx = (nums[i] - 1) % len;
        nums[idx] += len;
    }
    
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < len; i++) {
        if (nums[i] <= len) {
            result.add(i + 1);
        }
    }
    
    return result;
}
```

# Find All Numbers Disappeared in an Array (HashMap)

```java
public static List<Integer> findDisappearedNumbers(int[] nums) {
    HashMap<Integer, Integer> hashMap = new HashMap<>();
    int min = nums[0];
    int max = nums[0];
    for (int num : nums) {
        min = Math.min(min, num);
        max = Math.max(max, num);
        hashMap.put(num, num);
    }
    
    List<Integer> result = new ArrayList<>();
    for (int num = min; num <= max; num++) {
        if (!hashMap.containsKey(num)) {
            result.add(num);
        }
    }
    
    return result;
}
```

