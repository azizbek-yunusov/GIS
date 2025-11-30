# Live Coding Interview - Algorithm Masalalari va Yechimlari

## 1. Two Sum (Ikki sonning yig'indisi)

**Masala:** Array berilgan va target raqam berilgan. Array ichidan yig'indisi target ga teng bo'lgan ikki sonning indekslarini toping.

**Misol:**
```
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1]
Tushuntirish: nums[0] + nums[1] = 2 + 7 = 9
```

**Yechim 1 - Brute Force (O(n²)):**
```javascript
function twoSum(nums, target) {
  // Har bir elementni boshqalari bilan solishtirish
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
  return [];
}

console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

**Yechim 2 - Hash Map (O(n)) - OPTIMAL:**
```javascript
function twoSum(nums, target) {
  const map = new Map(); // qiymat: indeks
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    // Complement map da bormi?
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    
    // Hozirgi elementni map ga qo'shish
    map.set(nums[i], i);
  }
  
  return [];
}

// Time: O(n), Space: O(n)
```

**Izoh:**
- Hash map yordamida bir marta aylanib javobni topamiz
- Har bir element uchun uning "juftini" (complement) qidiramiz
- Space complexity qurbon qilib time complexity ni yaxshilaymiz

---

## 2. Reverse String (String ni teskari aylantirish)

**Masala:** String ni teskari aylantiring.

**Misol:**
```
Input: "hello"
Output: "olleh"
```

**Yechim 1 - Built-in metodlar:**
```javascript
function reverseString(str) {
  return str.split('').reverse().join('');
}
```

**Yechim 2 - Two Pointers (O(n)):**
```javascript
function reverseString(str) {
  const arr = str.split('');
  let left = 0;
  let right = arr.length - 1;
  
  while (left < right) {
    // Elementlarni almashtirish
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }
  
  return arr.join('');
}

console.log(reverseString("hello")); // "olleh"
```

**Yechim 3 - Rekursiv:**
```javascript
function reverseString(str) {
  // Base case
  if (str === '') return '';
  
  // Birinchi belgini oxiriga qo'shish
  return reverseString(str.slice(1)) + str[0];
}
```

**Izoh:**
- Two pointers yechimi eng optimal (in-place)
- Rekursiv yechim elegant lekin stack overflow xavfi bor
- Built-in metodlar interview da ko'pincha ruxsat etiladi

---

## 3. Palindrome Checker (Palindrom tekshirish)

**Masala:** String palindrom ekanligini tekshiring (oldinga va orqaga bir xil).

**Misol:**
```
Input: "racecar"
Output: true

Input: "hello"
Output: false
```

**Yechim 1 - Reverse va solishtirish:**
```javascript
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  const reversed = cleaned.split('').reverse().join('');
  return cleaned === reversed;
}
```

**Yechim 2 - Two Pointers (O(n)) - OPTIMAL:**
```javascript
function isPalindrome(str) {
  // Faqat harflar va raqamlar, kichik harfga
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  
  let left = 0;
  let right = cleaned.length - 1;
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }
  
  return true;
}

console.log(isPalindrome("A man, a plan, a canal: Panama")); // true
console.log(isPalindrome("race a car")); // false
```

**Izoh:**
- Two pointers yechimi O(n/2) = O(n) vaqt oladi
- Space complexity O(1) (cleaned string ni hisobga olmasak)
- Regex bilan faqat kerakli belgilarni qoldirish muhim

---

## 4. FizzBuzz

**Masala:** 1 dan n gacha raqamlarni chiqaring, lekin:
- 3 ga bo'linadigan raqamlar o'rniga "Fizz"
- 5 ga bo'linadigan raqamlar o'rniga "Buzz"
- 3 va 5 ga bo'linadigan raqamlar o'rniga "FizzBuzz"

**Yechim:**
```javascript
function fizzBuzz(n) {
  const result = [];
  
  for (let i = 1; i <= n; i++) {
    // Muhim: avval 15 ga bo'linishni tekshirish
    if (i % 15 === 0) {
      result.push("FizzBuzz");
    } else if (i % 3 === 0) {
      result.push("Fizz");
    } else if (i % 5 === 0) {
      result.push("Buzz");
    } else {
      result.push(i.toString());
    }
  }
  
  return result;
}

console.log(fizzBuzz(15));
// ["1", "2", "Fizz", "4", "Buzz", "Fizz", "7", "8", "Fizz", 
//  "Buzz", "11", "Fizz", "13", "14", "FizzBuzz"]
```

**Yechim 2 - Kengaytirilgan (extensible):**
```javascript
function fizzBuzz(n) {
  const result = [];
  
  for (let i = 1; i <= n; i++) {
    let output = '';
    
    if (i % 3 === 0) output += 'Fizz';
    if (i % 5 === 0) output += 'Buzz';
    
    result.push(output || i.toString());
  }
  
  return result;
}
```

**Izoh:**
- Ikkinchi yechim yangi qoidalar qo'shish uchun osonroq
- Tartib muhim: kattadan kichikka tekshirish
- Time: O(n), Space: O(n)

---

## 5. Maximum Subarray Sum (Kadane's Algorithm)

**Masala:** Array ichidagi eng katta yig'indiga ega bo'lgan subarray topish.

**Misol:**
```
Input: [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: 6
Tushuntirish: [4, -1, 2, 1] = 6
```

**Yechim - Kadane's Algorithm (O(n)):**
```javascript
function maxSubArray(nums) {
  let maxSum = nums[0];      // Global maksimum
  let currentSum = nums[0];  // Hozirgi maksimum
  
  for (let i = 1; i < nums.length; i++) {
    // Yangi boshlamoq yoki davom etmoq?
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    
    // Global maksimumni yangilash
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}

console.log(maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6
```

**Batafsil izoh:**
```javascript
function maxSubArrayDetailed(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  let start = 0;
  let end = 0;
  let tempStart = 0;
  
  for (let i = 1; i < nums.length; i++) {
    // Agar yangi boshlamoq yaxshiroq bo'lsa
    if (nums[i] > currentSum + nums[i]) {
      currentSum = nums[i];
      tempStart = i;  // Yangi boshlanish nuqtasi
    } else {
      currentSum = currentSum + nums[i];
    }
    
    // Global maksimumni yangilash
    if (currentSum > maxSum) {
      maxSum = currentSum;
      start = tempStart;
      end = i;
    }
  }
  
  return {
    sum: maxSum,
    subarray: nums.slice(start, end + 1)
  };
}

console.log(maxSubArrayDetailed([-2, 1, -3, 4, -1, 2, 1, -5, 4]));
// { sum: 6, subarray: [4, -1, 2, 1] }
```

**Izoh:**
- Dynamic Programming klassik misoli
- Time: O(n), Space: O(1)
- Har qadam da qaror qilish: davom etmoq yoki yangi boshlamoq?

---

## 6. Valid Anagram (Anagram tekshirish)

**Masala:** Ikki string anagram ekanligini tekshiring (bir xil harflardan tashkil topgan).

**Misol:**
```
Input: s = "anagram", t = "nagaram"
Output: true

Input: s = "rat", t = "car"
Output: false
```

**Yechim 1 - Sorting (O(n log n)):**
```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  
  const sortedS = s.split('').sort().join('');
  const sortedT = t.split('').sort().join('');
  
  return sortedS === sortedT;
}
```

**Yechim 2 - Hash Map (O(n)) - OPTIMAL:**
```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  
  const charCount = {};
  
  // Birinchi string harflarini sanash
  for (let char of s) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  // Ikkinchi string harflarini ayirish
  for (let char of t) {
    if (!charCount[char]) return false;
    charCount[char]--;
  }
  
  return true;
}

console.log(isAnagram("anagram", "nagaram")); // true
```

**Yechim 3 - Array bilan (Unicode uchun):**
```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  
  const count = new Array(26).fill(0);
  
  for (let i = 0; i < s.length; i++) {
    count[s.charCodeAt(i) - 97]++;  // 'a' = 97
    count[t.charCodeAt(i) - 97]--;
  }
  
  return count.every(c => c === 0);
}
```

**Izoh:**
- Hash map yechimi eng optimal
- Array yechimi faqat lowercase ingliz harflari uchun
- Space: O(1) (cheklangan belgilar soni)

---

## 7. Binary Search (Ikkilik qidiruv)

**Masala:** Tartiblangan array da element topish.

**Misol:**
```
Input: nums = [-1, 0, 3, 5, 9, 12], target = 9
Output: 4 (indeks)
```

**Yechim - Iterative (O(log n)):**
```javascript
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length - 1;
  
  while (left <= right) {
    // Overflow dan qochish uchun
    const mid = left + Math.floor((right - left) / 2);
    
    if (nums[mid] === target) {
      return mid;  // Topildi
    } else if (nums[mid] < target) {
      left = mid + 1;  // O'ng qismda qidirish
    } else {
      right = mid - 1;  // Chap qismda qidirish
    }
  }
  
  return -1;  // Topilmadi
}

console.log(binarySearch([-1, 0, 3, 5, 9, 12], 9)); // 4
console.log(binarySearch([-1, 0, 3, 5, 9, 12], 2)); // -1
```

**Yechim - Recursive:**
```javascript
function binarySearchRecursive(nums, target, left = 0, right = nums.length - 1) {
  if (left > right) return -1;
  
  const mid = left + Math.floor((right - left) / 2);
  
  if (nums[mid] === target) {
    return mid;
  } else if (nums[mid] < target) {
    return binarySearchRecursive(nums, target, mid + 1, right);
  } else {
    return binarySearchRecursive(nums, target, left, mid - 1);
  }
}
```

**Izoh:**
- Time: O(log n) - har safar yarmiga bo'lish
- Space: O(1) iterative, O(log n) recursive (call stack)
- Faqat tartiblangan array uchun ishlaydi
- `left + Math.floor((right - left) / 2)` overflow dan himoya qiladi

---

## 8. Merge Two Sorted Arrays (Ikki tartiblangan array ni birlashtirish)

**Masala:** Ikki tartiblangan array ni bitta tartiblangan array ga aylantirish.

**Misol:**
```
Input: nums1 = [1, 2, 3], nums2 = [2, 5, 6]
Output: [1, 2, 2, 3, 5, 6]
```

**Yechim - Two Pointers (O(n + m)):**
```javascript
function mergeSortedArrays(nums1, nums2) {
  const merged = [];
  let i = 0;  // nums1 pointer
  let j = 0;  // nums2 pointer
  
  // Ikkala array ham tugamagunicha
  while (i < nums1.length && j < nums2.length) {
    if (nums1[i] < nums2[j]) {
      merged.push(nums1[i]);
      i++;
    } else {
      merged.push(nums2[j]);
      j++;
    }
  }
  
  // Qolgan elementlarni qo'shish
  while (i < nums1.length) {
    merged.push(nums1[i]);
    i++;
  }
  
  while (j < nums2.length) {
    merged.push(nums2[j]);
    j++;
  }
  
  return merged;
}

console.log(mergeSortedArrays([1, 2, 3], [2, 5, 6]));
// [1, 2, 2, 3, 5, 6]
```

**Optimized - Spread operator:**
```javascript
function mergeSortedArrays(nums1, nums2) {
  const merged = [];
  let i = 0, j = 0;
  
  while (i < nums1.length && j < nums2.length) {
    merged.push(nums1[i] < nums2[j] ? nums1[i++] : nums2[j++]);
  }
  
  return [...merged, ...nums1.slice(i), ...nums2.slice(j)];
}
```

**Izoh:**
- Time: O(n + m) - har ikkala array ni bir marta ko'rib chiqish
- Space: O(n + m) - yangi array
- Two pointers klassik texnikasi
- Merge Sort algoritmining asosi

---

## 9. First Non-Repeating Character (Takrorlanmaydigan birinchi belgi)

**Masala:** String dagi birinchi takrorlanmaydigan belgini toping.

**Misol:**
```
Input: "leetcode"
Output: "l"

Input: "loveleetcode"
Output: "v"
```

**Yechim - Hash Map (O(n)):**
```javascript
function firstUniqChar(s) {
  const charCount = {};
  
  // 1. Barcha belgilarni sanash
  for (let char of s) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  // 2. Birinchi unique belgini topish
  for (let char of s) {
    if (charCount[char] === 1) {
      return char;
    }
  }
  
  return null;  // Topilmasa
}

console.log(firstUniqChar("leetcode"));      // "l"
console.log(firstUniqChar("loveleetcode"));  // "v"
console.log(firstUniqChar("aabb"));          // null
```

**Yechim 2 - Map bilan (tartib saqlash):**
```javascript
function firstUniqChar(s) {
  const charCount = new Map();
  
  // Sanash
  for (let char of s) {
    charCount.set(char, (charCount.get(char) || 0) + 1);
  }
  
  // Birinchi unique
  for (let [char, count] of charCount) {
    if (count === 1) {
      return char;
    }
  }
  
  return null;
}
```

**Yechim 3 - Index bilan:**
```javascript
function firstUniqChar(s) {
  for (let i = 0; i < s.length; i++) {
    const char = s[i];
    
    // Agar birinchi va oxirgi indeks bir xil bo'lsa - unique
    if (s.indexOf(char) === s.lastIndexOf(char)) {
      return char;
    }
  }
  
  return null;
}
```

**Izoh:**
- Time: O(n) - ikki marta o'tish
- Space: O(k) - k = unique belgilar soni
- Map insertion order ni saqlaydi (ES6+)

---

## 10. Valid Parentheses (To'g'ri qavslar)

**Masala:** Qavslar to'g'ri yopilgan yoki yopilmaganligini tekshiring.

**Misol:**
```
Input: "()"
Output: true

Input: "()[]{}"
Output: true

Input: "(]"
Output: false

Input: "([)]"
Output: false
```

**Yechim - Stack (O(n)):**
```javascript
function isValidParentheses(s) {
  const stack = [];
  const pairs = {
    ')': '(',
    ']': '[',
    '}': '{'
  };
  
  for (let char of s) {
    // Ochuvchi qavs - stack ga qo'shish
    if (char === '(' || char === '[' || char === '{') {
      stack.push(char);
    } 
    // Yopuvchi qavs - tekshirish
    else {
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    }
  }
  
  // Stack bo'sh bo'lishi kerak
  return stack.length === 0;
}

console.log(isValidParentheses("()"));        // true
console.log(isValidParentheses("()[]{}"));    // true
console.log(isValidParentheses("(]"));        // false
console.log(isValidParentheses("([)]"));      // false
console.log(isValidParentheses("{[]}"));      // true
```

**Izoh:**
- Stack - LIFO (Last In First Out) struktura
- Time: O(n), Space: O(n)
- Har bir yopuvchi qavs uchun stack dan tekshirish
- Oxirida stack bo'sh bo'lishi kerak

---

## 11. Reverse Linked List (Bog'langan ro'yxatni teskari aylantirish)

**Masala:** Linked list ni teskari aylantiring.

**Misol:**
```
Input: 1 -> 2 -> 3 -> 4 -> 5
Output: 5 -> 4 -> 3 -> 2 -> 1
```

**Yechim - Iterative (O(n)):**
```javascript
class ListNode {
  constructor(val = 0, next = null) {
    this.val = val;
    this.next = next;
  }
}

function reverseLinkedList(head) {
  let prev = null;
  let current = head;
  
  while (current !== null) {
    // Keyingi node ni saqlash
    const next = current.next;
    
    // Yo'nalishni teskari aylantirish
    current.next = prev;
    
    // Oldinga siljish
    prev = current;
    current = next;
  }
  
  return prev;  // Yangi head
}

// Test
const list = new ListNode(1, 
  new ListNode(2, 
    new ListNode(3, 
      new ListNode(4, 
        new ListNode(5)))));

const reversed = reverseLinkedList(list);
```

**Yechim - Recursive:**
```javascript
function reverseLinkedListRecursive(head) {
  // Base case
  if (head === null || head.next === null) {
    return head;
  }
  
  // Qolgan qismni teskari aylantirish
  const newHead = reverseLinkedListRecursive(head.next);
  
  // Pointer larni o'zgartirish
  head.next.next = head;
  head.next = null;
  
  return newHead;
}
```

**Vizual tushuncha:**
```
Original: 1 -> 2 -> 3 -> null

Step 1:   null <- 1    2 -> 3 -> null
          prev   curr  next

Step 2:   null <- 1 <- 2    3 -> null
                 prev  curr next

Step 3:   null <- 1 <- 2 <- 3
                       prev curr(null)
```

**Izoh:**
- Time: O(n), Space: O(1) iterative, O(n) recursive
- Uchta pointer: prev, current, next
- Recursive yechim elegant lekin stack overflow xavfi

---

## 12. Find Duplicates (Takrorlanuvchilarni topish)

**Masala:** Array da takrorlanuvchi elementlarni toping.

**Misol:**
```
Input: [1, 2, 3, 1, 2, 4]
Output: [1, 2]
```

**Yechim 1 - Set bilan (O(n)):**
```javascript
function findDuplicates(nums) {
  const seen = new Set();
  const duplicates = new Set();
  
  for (let num of nums) {
    if (seen.has(num)) {
      duplicates.add(num);
    } else {
      seen.add(num);
    }
  }
  
  return Array.from(duplicates);
}

console.log(findDuplicates([1, 2, 3, 1, 2, 4])); // [1, 2]
```

**Yechim 2 - Object bilan:**
```javascript
function findDuplicates(nums) {
  const count = {};
  const duplicates = [];
  
  for (let num of nums) {
    count[num] = (count[num] || 0) + 1;
  }
  
  for (let num in count) {
    if (count[num] > 1) {
      duplicates.push(Number(num));
    }
  }
  
  return duplicates;
}
```

**Yechim 3 - In-place (maxsus shart: 1 ≤ nums[i] ≤ n):**
```javascript
function findDuplicates(nums) {
  const duplicates = [];
  
  for (let i = 0; i < nums.length; i++) {
    const index = Math.abs(nums[i]) - 1;
    
    // Agar allaqachon manfiy bo'lsa - duplicate
    if (nums[index] < 0) {
      duplicates.push(Math.abs(nums[i]));
    } else {
      nums[index] = -nums[index];
    }
  }
  
  return duplicates;
}

// Time: O(n), Space: O(1)
```

---

## Interview Maslahatlar

### 1. Problem Solving Yondashuv

```javascript
// AKSE - Algoritm, Kod, Sinov, Explain

// 1. ALGORITM - Qog'ozda yechimni chizish
// 2. KOD - Kodni yozish
// 3. SINOV - Test case lar bilan tekshirish
// 4. EXPLAIN - Vaqt va xotira murakkabligini tushuntirish
```

### 2. Optimization Strategiya

```javascript
// Brute Force -> Optimal

function optimizationSteps(problem) {
  // 1. Avval ishlashini ta'minlash (Brute Force)
  const bruteForce = solveBruteForce(problem);
  
  // 2. Time complexity ni yaxshilash
  // - Hash Map/Set ishlatish
  // - Two Pointers
  // - Binary Search
  // - Dynamic Programming
  
  // 3. Space complexity ni yaxshilash
  // - In-place operations
  // - Constant space
  
  return optimizedSolution;
}
```

### 3. Edge Cases tekshirish

```javascript
function testEdgeCases(solution) {
  // 1. Bo'sh input
  console.log(solution([]));
  console.log(solution(""));
  console.log(solution(null));
  
  // 2. Bitta element
  console.log(solution([1]));
  
  // 3. Ikki element
  console.log(solution([1, 2]));
  
  // 4. Barcha bir xil
  console.log(solution([1, 1, 1]));
  
  // 5. Tartiblangan
  console.log(solution([1, 2, 3, 4]));
  
  // 6. Teskari tartiblangan
  console.log(solution([4, 3, 2, 1]));
  
  // 7. Manfiy sonlar
  console.log(solution([-1, -2, -3]));
  
  // 8. Katta raqamlar
  console.log(solution([Number.MAX_SAFE_INTEGER]));
}
```

### 4. Kommunikatsiya

```javascript
// Interview paytida:

// ✅ TO'G'RI:
"Men bu masalani uchta usul bilan yechishni ko'ryapman:
1. Brute Force - O(n²) time
2. Hash Map - O(n) time, O(n) space
3. Two Pointers - O(n) time, O(1) space

Qaysi birini implement qilishni xohlaysiz?"

// ❌ NOTO'G'RI:
*Jim bo'lib kod yozish*
```

## Time va Space Complexity Cheat Sheet

```javascript
const complexities = {
  'O(1)': 'Constant - eng yaxshi',
  'O(log n)': 'Logarithmic - Binary Search',
  'O(n)': 'Linear - bir marta o\'tish',
  'O(n log n)': 'Sorting - Merge Sort, Quick Sort',
  'O(n²)': 'Quadratic - Nested loops',
  'O(2^n)': 'Exponential - Fibonacci recursive',
  'O(n!)': 'Factorial - Permutations'
};
```

## Xulosa

Bu masalalar FAANG va boshqa tech kompaniyalarda eng ko'p uchraydigan savollarga kiradi. Har birini:
1. Tushunish
2. Brute force yechim
3. Optimal yechim
4. Edge cases
5. Complexity analysis

Bularni o'rganib, amaliyotda qo'llash interview da muvaffaqiyat kafolati!
Live Coding interview uchun eng mashhur algorithm masalalari va ularning batafsil yechimlari tayyorladim!
Qamrab olingan masalalar:

Two Sum - Hash Map texnikasi
Reverse String - Two Pointers
Palindrome - String manipulation
FizzBuzz - Klassik masala
Maximum Subarray - Kadane's Algorithm
Valid Anagram - Hash Map
Binary Search - O(log n) qidiruv
Merge Sorted Arrays - Two Pointers
First Non-Repeating Character - Hash Map
Valid Parentheses - Stack
Reverse Linked List - Pointer manipulation
Find Duplicates - Set/Hash Map

Har bir masala uchun:

✅ Masala bayoni va misollar
✅ Bir nechta yechim usullari (Brute Force va Optimal)
✅ Kod misollari
✅ Batafsil izohlar
✅ Time va Space Complexity tahlili
✅ Vizual tushuntirishlar

Qo'shimcha:

Interview maslahatlar
Edge cases tekshirish
Kommunikatsiya strategiyasi
Complexity Cheat Sheet
