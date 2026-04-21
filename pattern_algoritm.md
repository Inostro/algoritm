# Шаблоны 17 алгоритмических паттернов на Go

Каждый паттерн: когда применять → шаблон → классическая задача с полным решением → типичные грабли.

Код предполагает Go 1.21+. Пакеты-помощники импортируются локально в каждом примере.


---

1. Arrays & Hashing
2. Two Pointers
3. Stack
4. Binary Search
5. Sliding Window
6. Linked List
7. Trees
8. Tries
9. Backtracking
10. Heap / Priority Queue
11. Graphs
12. 1-D DP
13. Intervals
14. Greedy
15. Advanced Graphs
16. 2-D DP
17. Bit Manipulation

---

## 1. Arrays & Hashing

**Когда:** частота элементов, дубликаты, anagram, поиск за один проход.

**Ключевые структуры в Go:**

```go
// Множество: map с пустой структурой (0 байт на значение)
seen := make(map[int]struct{})
seen[x] = struct{}{}
_, ok := seen[x]

// Счётчик частот
count := make(map[rune]int)
count[ch]++

// Группировка
groups := make(map[string][]string)
groups[key] = append(groups[key], value)
```

**Шаблон — Two Sum:**

```go
func twoSum(nums []int, target int) []int {
    seen := make(map[int]int) // value → index
    for i, x := range nums {
        if j, ok := seen[target-x]; ok {
            return []int{j, i}
        }
        seen[x] = i
    }
    return nil
}
```

**Шаблон — группировка анаграмм:**

```go
func groupAnagrams(strs []string) [][]string {
    groups := make(map[[26]int][]string)
    for _, s := range strs {
        var key [26]int // массив можно использовать как ключ map
        for _, ch := range s {
            key[ch-'a']++
        }
        groups[key] = append(groups[key], s)
    }
    result := make([][]string, 0, len(groups))
    for _, g := range groups {
        result = append(result, g)
    }
    return result
}
```

**Грабли:**
- Срез `[]int` **нельзя** использовать как ключ map — используй массив `[26]int` или строку.
- Не забывай `, ok` при чтении из map, чтобы отличить отсутствие ключа от нулевого значения.

---

## 2. Two Pointers

**Когда:** отсортированный массив, поиск пары/тройки, палиндромы, удаление дубликатов in-place.

**Шаблон — два индекса навстречу:**

```go
func template(arr []int) {
    l, r := 0, len(arr)-1
    for l < r {
        // логика на основе arr[l] и arr[r]
        if condition {
            l++
        } else {
            r--
        }
    }
}
```

**Классика — Three Sum:**

```go
import "sort"

func threeSum(nums []int) [][]int {
    sort.Ints(nums)
    var result [][]int
    for i := 0; i < len(nums)-2; i++ {
        if i > 0 && nums[i] == nums[i-1] {
            continue // пропускаем дубликаты по i
        }
        l, r := i+1, len(nums)-1
        for l < r {
            sum := nums[i] + nums[l] + nums[r]
            switch {
            case sum < 0:
                l++
            case sum > 0:
                r--
            default:
                result = append(result, []int{nums[i], nums[l], nums[r]})
                // пропускаем дубликаты
                for l < r && nums[l] == nums[l+1] {
                    l++
                }
                for l < r && nums[r] == nums[r-1] {
                    r--
                }
                l++
                r--
            }
        }
    }
    return result
}
```

**Грабли:**
- Пропуск дубликатов — самая частая ошибка. Пропускай **после** добавления в ответ, не до.
- Для палиндромов: индексы `l, r := 0, len(s)-1`, условие `l < r`.

---

## 3. Stack

**Когда:** валидные скобки, monotonic stack («следующий больший элемент»), вычисление выражений, DFS итеративно.

**В Go нет встроенного стека — используем срез:**

```go
stack := []int{}
stack = append(stack, x)          // push
top := stack[len(stack)-1]        // peek
stack = stack[:len(stack)-1]      // pop
isEmpty := len(stack) == 0
```

**Классика — валидные скобки:**

```go
func isValid(s string) bool {
    pairs := map[rune]rune{')': '(', ']': '[', '}': '{'}
    stack := []rune{}
    for _, ch := range s {
        if open, isClose := pairs[ch]; isClose {
            if len(stack) == 0 || stack[len(stack)-1] != open {
                return false
            }
            stack = stack[:len(stack)-1]
        } else {
            stack = append(stack, ch)
        }
    }
    return len(stack) == 0
}
```

**Шаблон — monotonic stack (следующий больший справа):**

```go
func nextGreater(nums []int) []int {
    result := make([]int, len(nums))
    for i := range result {
        result[i] = -1
    }
    stack := []int{} // индексы, значения nums[stack[i]] по убыванию
    for i, x := range nums {
        for len(stack) > 0 && nums[stack[len(stack)-1]] < x {
            idx := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            result[idx] = x
        }
        stack = append(stack, i)
    }
    return result
}
```

**Грабли:**
- Операция `stack[:len(stack)-1]` на пустом срезе не паникует, но `stack[len(stack)-1]` паникует — проверяй `len > 0`.
- В monotonic stack часто хранят **индексы**, а не значения — чтобы знать позицию.

---

## 4. Binary Search

**Когда:** отсортированный массив, «минимум/максимум, удовлетворяющий условию», поиск в отсортированном 2D.

**Канонический шаблон:**

```go
func binarySearch(arr []int, target int) int {
    l, r := 0, len(arr)-1
    for l <= r {
        mid := l + (r-l)/2 // избегаем переполнения
        switch {
        case arr[mid] == target:
            return mid
        case arr[mid] < target:
            l = mid + 1
        default:
            r = mid - 1
        }
    }
    return -1
}
```

**Шаблон — «левая граница» (первое вхождение):**

```go
func leftBound(arr []int, target int) int {
    l, r := 0, len(arr) // правая граница невключительная
    for l < r {
        mid := l + (r-l)/2
        if arr[mid] < target {
            l = mid + 1
        } else {
            r = mid
        }
    }
    return l // первая позиция, где arr[l] >= target
}
```

**В стандартной библиотеке есть `sort.Search`:**

```go
import "sort"

// Найти первый индекс, где arr[i] >= target
idx := sort.Search(len(arr), func(i int) bool {
    return arr[i] >= target
})
```

**Шаблон — бинарный поиск по ответу:**

```go
// "Минимальная скорость еды бананов" и подобные.
// Ищем минимальное x такое, что check(x) == true
func minValid(lo, hi int, check func(int) bool) int {
    for lo < hi {
        mid := lo + (hi-lo)/2
        if check(mid) {
            hi = mid
        } else {
            lo = mid + 1
        }
    }
    return lo
}
```

**Грабли:**
- `mid := (l + r) / 2` может переполниться для больших `int`. Пиши `l + (r-l)/2`.
- Выбор `l <= r` vs `l < r` и `r = mid-1` vs `r = mid` — зависит от того, включительная правая граница или нет. Лучше запомнить один шаблон и держать его.

---

## 5. Sliding Window

**Когда:** подмассив/подстрока с условием, «не более K различных», максимум суммы окна.

**Шаблон — окно переменного размера:**

```go
func variableWindow(arr []int) int {
    l := 0
    best := 0
    // состояние окна, например:
    count := make(map[int]int)

    for r := 0; r < len(arr); r++ {
        // РАСШИРЕНИЕ: добавляем arr[r] в окно
        count[arr[r]]++

        // СОКРАЩЕНИЕ: пока окно невалидно
        for !valid(count) {
            count[arr[l]]--
            if count[arr[l]] == 0 {
                delete(count, arr[l])
            }
            l++
        }

        // ОБНОВЛЕНИЕ ответа
        if r-l+1 > best {
            best = r - l + 1
        }
    }
    return best
}
```

**Классика — самая длинная подстрока без повторов:**

```go
func lengthOfLongestSubstring(s string) int {
    lastSeen := make(map[byte]int)
    l, best := 0, 0
    for r := 0; r < len(s); r++ {
        if idx, ok := lastSeen[s[r]]; ok && idx >= l {
            l = idx + 1
        }
        lastSeen[s[r]] = r
        if r-l+1 > best {
            best = r - l + 1
        }
    }
    return best
}
```

**Шаблон — окно фиксированного размера k:**

```go
func maxSumSizeK(arr []int, k int) int {
    sum := 0
    for i := 0; i < k; i++ {
        sum += arr[i]
    }
    best := sum
    for r := k; r < len(arr); r++ {
        sum += arr[r] - arr[r-k]
        if sum > best {
            best = sum
        }
    }
    return best
}
```

**Грабли:**
- В Go `range` по строке даёт `rune` (UTF-8 символы), а индексация `s[i]` даёт `byte`. Для ASCII-задач используй `s[i]`; для юникода — предварительно `runes := []rune(s)`.
- Не забывай обновить ответ **после** сокращения окна, когда оно валидно.

---

## 6. Linked List

**Когда:** reverse, cycle detection, merge, midpoint, удаление N-го с конца.

**Базовый тип:**

```go
type ListNode struct {
    Val  int
    Next *ListNode
}
```

**Шаблон — reverse:**

```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    for curr != nil {
        next := curr.Next
        curr.Next = prev
        prev = curr
        curr = next
    }
    return prev
}
```

**Шаблон — быстрый и медленный указатель (cycle detection):**

```go
func hasCycle(head *ListNode) bool {
    slow, fast := head, head
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return true
        }
    }
    return false
}
```

**Шаблон — dummy head (упрощает удаление/вставку):**

```go
func mergeTwoLists(l1, l2 *ListNode) *ListNode {
    dummy := &ListNode{}
    tail := dummy
    for l1 != nil && l2 != nil {
        if l1.Val <= l2.Val {
            tail.Next = l1
            l1 = l1.Next
        } else {
            tail.Next = l2
            l2 = l2.Next
        }
        tail = tail.Next
    }
    if l1 != nil {
        tail.Next = l1
    } else {
        tail.Next = l2
    }
    return dummy.Next
}
```

**Грабли:**
- Всегда проверяй `node != nil` перед обращением к `node.Next` — иначе nil pointer dereference panic.
- Dummy node спасает от специальной обработки головы списка.
- Для fast pointer: условие `fast != nil && fast.Next != nil`, иначе `fast.Next.Next` упадёт.

---

## 7. Trees

**Когда:** BST, высота, LCA, обходы, проверка валидности дерева.

**Базовый тип:**

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}
```

**Шаблон — рекурсивный DFS (in-order):**

```go
func inorder(root *TreeNode) []int {
    var result []int
    var dfs func(*TreeNode)
    dfs = func(node *TreeNode) {
        if node == nil {
            return
        }
        dfs(node.Left)
        result = append(result, node.Val)
        dfs(node.Right)
    }
    dfs(root)
    return result
}
```

**Шаблон — BFS по уровням:**

```go
func levelOrder(root *TreeNode) [][]int {
    if root == nil {
        return nil
    }
    var result [][]int
    queue := []*TreeNode{root}
    for len(queue) > 0 {
        size := len(queue)
        level := make([]int, 0, size)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        result = append(result, level)
    }
    return result
}
```

**Классика — максимальная глубина:**

```go
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    l, r := maxDepth(root.Left), maxDepth(root.Right)
    if l > r {
        return l + 1
    }
    return r + 1
}
```

**Шаблон — «возвращаем информацию снизу вверх» (post-order):**

```go
// LCA двух узлов
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil || root == p || root == q {
        return root
    }
    left := lowestCommonAncestor(root.Left, p, q)
    right := lowestCommonAncestor(root.Right, p, q)
    if left != nil && right != nil {
        return root
    }
    if left != nil {
        return left
    }
    return right
}
```

**Грабли:**
- Всегда база рекурсии: `if node == nil { return ... }`.
- В Go нет встроенного `math.Max` для int до Go 1.21. В Go 1.21+ — есть `max()` и `min()` как встроенные функции.

---

## 8. Tries

**Когда:** словарь, автодополнение, префиксный поиск, задачи на XOR с префиксами.

**Шаблон:**

```go
type Trie struct {
    children [26]*Trie
    end      bool
}

func NewTrie() *Trie {
    return &Trie{}
}

func (t *Trie) Insert(word string) {
    node := t
    for i := 0; i < len(word); i++ {
        idx := word[i] - 'a'
        if node.children[idx] == nil {
            node.children[idx] = &Trie{}
        }
        node = node.children[idx]
    }
    node.end = true
}

func (t *Trie) Search(word string) bool {
    node := t.find(word)
    return node != nil && node.end
}

func (t *Trie) StartsWith(prefix string) bool {
    return t.find(prefix) != nil
}

func (t *Trie) find(s string) *Trie {
    node := t
    for i := 0; i < len(s); i++ {
        idx := s[i] - 'a'
        if node.children[idx] == nil {
            return nil
        }
        node = node.children[idx]
    }
    return node
}
```

**Вариант с map (когда алфавит большой или символы произвольные):**

```go
type Trie struct {
    children map[rune]*Trie
    end      bool
}

func NewTrie() *Trie {
    return &Trie{children: make(map[rune]*Trie)}
}
```

**Грабли:**
- Массив `[26]*Trie` быстрее и экономнее памяти, чем map, но требует фиксированного алфавита.
- Для подсчёта количества слов с префиксом добавь в узел поле `count`, инкрементируй при insert.

---

## 9. Backtracking

**Когда:** все перестановки/комбинации/подмножества, N-queens, sudoku, word search.

**Универсальный шаблон:**

```go
func backtrack(path []int, choices []int) {
    if isComplete(path) {
        // сохранить копию path в результат
        return
    }
    for _, c := range choices {
        if !isValid(path, c) {
            continue
        }
        path = append(path, c) // выбор
        backtrack(path, nextChoices(choices, c))
        path = path[:len(path)-1] // откат
    }
}
```

**Классика — подмножества:**

```go
func subsets(nums []int) [][]int {
    var result [][]int
    var path []int
    var backtrack func(start int)
    backtrack = func(start int) {
        // Копируем path — нельзя сохранить сам срез, он мутабелен
        cp := make([]int, len(path))
        copy(cp, path)
        result = append(result, cp)

        for i := start; i < len(nums); i++ {
            path = append(path, nums[i])
            backtrack(i + 1)
            path = path[:len(path)-1]
        }
    }
    backtrack(0)
    return result
}
```

**Классика — перестановки:**

```go
func permute(nums []int) [][]int {
    var result [][]int
    used := make([]bool, len(nums))
    var path []int

    var backtrack func()
    backtrack = func() {
        if len(path) == len(nums) {
            cp := make([]int, len(path))
            copy(cp, path)
            result = append(result, cp)
            return
        }
        for i := 0; i < len(nums); i++ {
            if used[i] {
                continue
            }
            used[i] = true
            path = append(path, nums[i])
            backtrack()
            path = path[:len(path)-1]
            used[i] = false
        }
    }
    backtrack()
    return result
}
```

**Грабли:**
- **Всегда копируй** `path` при сохранении в результат. Срез — это вьюшка на массив, все «сохранённые» будут указывать на один общий буфер.
- Откат должен быть зеркальным выбору: что добавил — то удали, что пометил — то размети.

---

## 10. Heap / Priority Queue

**Когда:** top-K, k-й по величине, медиана потока, merge-K отсортированных, Дейкстра.

**В Go куча реализуется через `container/heap` — нужно реализовать интерфейс:**

```go
import "container/heap"

// Min-heap для int
type IntHeap []int

func (h IntHeap) Len() int            { return len(h) }
func (h IntHeap) Less(i, j int) bool  { return h[i] < h[j] } // < для min, > для max
func (h IntHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *IntHeap) Push(x any)         { *h = append(*h, x.(int)) }
func (h *IntHeap) Pop() any {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[:n-1]
    return x
}
```

**Использование:**

```go
h := &IntHeap{}
heap.Init(h)          // если уже есть элементы
heap.Push(h, 5)
heap.Push(h, 2)
heap.Push(h, 8)
min := (*h)[0]        // peek
min = heap.Pop(h).(int) // 2
```

**Классика — k-й наибольший элемент (через min-heap размера k):**

```go
func findKthLargest(nums []int, k int) int {
    h := &IntHeap{}
    for _, x := range nums {
        heap.Push(h, x)
        if h.Len() > k {
            heap.Pop(h) // выбрасываем минимум
        }
    }
    return (*h)[0] // в куче остались k наибольших, минимум из них — k-й наибольший
}
```

**Шаблон — куча пар (например, для Дейкстры):**

```go
type Item struct {
    dist, node int
}
type PQ []Item

func (pq PQ) Len() int            { return len(pq) }
func (pq PQ) Less(i, j int) bool  { return pq[i].dist < pq[j].dist }
func (pq PQ) Swap(i, j int)       { pq[i], pq[j] = pq[j], pq[i] }
func (pq *PQ) Push(x any)         { *pq = append(*pq, x.(Item)) }
func (pq *PQ) Pop() any {
    old := *pq
    n := len(old)
    x := old[n-1]
    *pq = old[:n-1]
    return x
}
```

**Грабли:**
- **Не вызывай** `h.Pop()` напрямую — это метод интерфейса. Вызывай `heap.Pop(h)`.
- `heap.Push/Pop` возвращают `any`, нужна приведение типа: `.(int)` или `.(Item)`.
- Для max-heap поменяй `<` на `>` в `Less`.

---

## 11. Graphs

**Когда:** связность, компоненты, кратчайший путь в невзвешенном графе, обход.

**Представление — список смежности:**

```go
// Неориентированный граф
graph := make(map[int][]int)
graph[u] = append(graph[u], v)
graph[v] = append(graph[v], u)
```

**Шаблон — BFS:**

```go
func bfs(graph map[int][]int, start int) []int {
    var order []int
    visited := map[int]bool{start: true}
    queue := []int{start}
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        order = append(order, node)
        for _, next := range graph[node] {
            if !visited[next] {
                visited[next] = true
                queue = append(queue, next)
            }
        }
    }
    return order
}
```

**Шаблон — DFS (рекурсивно):**

```go
func dfs(graph map[int][]int, start int) []int {
    var order []int
    visited := make(map[int]bool)
    var rec func(int)
    rec = func(node int) {
        if visited[node] {
            return
        }
        visited[node] = true
        order = append(order, node)
        for _, next := range graph[node] {
            rec(next)
        }
    }
    rec(start)
    return order
}
```

**Классика — количество островов (grid DFS):**

```go
func numIslands(grid [][]byte) int {
    if len(grid) == 0 {
        return 0
    }
    m, n := len(grid), len(grid[0])
    count := 0
    var dfs func(i, j int)
    dfs = func(i, j int) {
        if i < 0 || i >= m || j < 0 || j >= n || grid[i][j] != '1' {
            return
        }
        grid[i][j] = '0' // помечаем как посещённую
        dfs(i+1, j)
        dfs(i-1, j)
        dfs(i, j+1)
        dfs(i, j-1)
    }
    for i := 0; i < m; i++ {
        for j := 0; j < n; j++ {
            if grid[i][j] == '1' {
                count++
                dfs(i, j)
            }
        }
    }
    return count
}
```

**Шаблон — Union-Find (DSU):**

```go
type DSU struct {
    parent, rank []int
}

func NewDSU(n int) *DSU {
    d := &DSU{parent: make([]int, n), rank: make([]int, n)}
    for i := range d.parent {
        d.parent[i] = i
    }
    return d
}

func (d *DSU) Find(x int) int {
    if d.parent[x] != x {
        d.parent[x] = d.Find(d.parent[x]) // path compression
    }
    return d.parent[x]
}

func (d *DSU) Union(x, y int) bool {
    px, py := d.Find(x), d.Find(y)
    if px == py {
        return false
    }
    if d.rank[px] < d.rank[py] {
        px, py = py, px
    }
    d.parent[py] = px
    if d.rank[px] == d.rank[py] {
        d.rank[px]++
    }
    return true
}
```

**Грабли:**
- В неориентированном графе добавляй ребро в обе стороны.
- `visited` проверяй **перед** добавлением в очередь, а не при обработке — иначе элемент попадёт в очередь несколько раз.

---

## 12. 1-D DP

**Когда:** «количество способов», «минимум/максимум до позиции i», лестницы, house robber, LIS.

**Шаблон:**

```go
// 1. Определить состояние: dp[i] = что-то, зависящее только от i
// 2. Найти переход: dp[i] = f(dp[i-1], dp[i-2], ...)
// 3. Задать базу: dp[0], dp[1]
// 4. Итерировать в правильном порядке

func climbStairs(n int) int {
    if n <= 2 {
        return n
    }
    dp := make([]int, n+1)
    dp[1], dp[2] = 1, 2
    for i := 3; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}

// Оптимизация памяти до O(1):
func climbStairsOpt(n int) int {
    if n <= 2 {
        return n
    }
    a, b := 1, 2
    for i := 3; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}
```

**Классика — House Robber:**

```go
func rob(nums []int) int {
    prev2, prev1 := 0, 0
    for _, x := range nums {
        prev2, prev1 = prev1, max(prev1, prev2+x)
    }
    return prev1
}
```

**Классика — Longest Increasing Subsequence, O(n²):**

```go
func lengthOfLIS(nums []int) int {
    n := len(nums)
    dp := make([]int, n)
    best := 0
    for i := 0; i < n; i++ {
        dp[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] && dp[j]+1 > dp[i] {
                dp[i] = dp[j] + 1
            }
        }
        if dp[i] > best {
            best = dp[i]
        }
    }
    return best
}
```

**Грабли:**
- Правильный порядок обхода: `dp[i]` должен обращаться только к уже посчитанному.
- Если `dp[i]` зависит только от нескольких предыдущих — оптимизируй память до `O(1)`.

---

## 13. Intervals

**Когда:** merge, overlap, meeting rooms, расписания, insert interval.

**Шаблон — сортировка + проход:**

```go
import "sort"

func merge(intervals [][]int) [][]int {
    if len(intervals) == 0 {
        return nil
    }
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    result := [][]int{intervals[0]}
    for i := 1; i < len(intervals); i++ {
        last := result[len(result)-1]
        if intervals[i][0] <= last[1] {
            // пересечение — расширяем
            if intervals[i][1] > last[1] {
                last[1] = intervals[i][1]
            }
        } else {
            result = append(result, intervals[i])
        }
    }
    return result
}
```

**Классика — сколько нужно переговорок (meeting rooms II):**

```go
import (
    "container/heap"
    "sort"
)

func minMeetingRooms(intervals [][]int) int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    h := &IntHeap{} // min-heap времён окончания
    for _, iv := range intervals {
        if h.Len() > 0 && (*h)[0] <= iv[0] {
            heap.Pop(h) // переговорка освободилась
        }
        heap.Push(h, iv[1])
    }
    return h.Len()
}
```

**Грабли:**
- Решай заранее: `[a, b]` — отрезок закрытый или полуоткрытый? От этого зависит `<=` или `<` при сравнении.
- Сортировка по **началу** или **концу** — разные задачи требуют разного.

---

## 14. Greedy

**Когда:** jump game, gas station, задача о расписании с дедлайнами, минимум монет канонической системы.

**Шаблон:** на каждом шаге делаем локально лучший выбор, не откатываемся.

**Классика — Jump Game (можно ли дойти до конца):**

```go
func canJump(nums []int) bool {
    reach := 0
    for i, x := range nums {
        if i > reach {
            return false // не дошли
        }
        if i+x > reach {
            reach = i + x
        }
    }
    return true
}
```

**Классика — Gas Station:**

```go
func canCompleteCircuit(gas, cost []int) int {
    total, tank, start := 0, 0, 0
    for i := 0; i < len(gas); i++ {
        diff := gas[i] - cost[i]
        total += diff
        tank += diff
        if tank < 0 {
            start = i + 1
            tank = 0
        }
    }
    if total < 0 {
        return -1
    }
    return start
}
```

**Грабли:**
- **Не всякая задача жадная.** Coin Change для произвольного набора монет — нет. Для канонического (1, 5, 10, 25) — да.
- Обоснуй корректность: почему локальный выбор приводит к глобальному оптимуму. Если не можешь — скорее всего, нужна DP.

---

## 15. Advanced Graphs

**Когда:** взвешенный shortest path, MST, topological sort, Bellman-Ford.

**Dijkstra:**

```go
import "container/heap"

type Edge struct {
    to, weight int
}

type Item struct {
    dist, node int
}
type PQ []Item

func (pq PQ) Len() int            { return len(pq) }
func (pq PQ) Less(i, j int) bool  { return pq[i].dist < pq[j].dist }
func (pq PQ) Swap(i, j int)       { pq[i], pq[j] = pq[j], pq[i] }
func (pq *PQ) Push(x any)         { *pq = append(*pq, x.(Item)) }
func (pq *PQ) Pop() any {
    old := *pq
    n := len(old)
    x := old[n-1]
    *pq = old[:n-1]
    return x
}

func dijkstra(graph map[int][]Edge, start, n int) []int {
    const INF = int(1e18)
    dist := make([]int, n)
    for i := range dist {
        dist[i] = INF
    }
    dist[start] = 0

    pq := &PQ{{0, start}}
    for pq.Len() > 0 {
        cur := heap.Pop(pq).(Item)
        if cur.dist > dist[cur.node] {
            continue // устаревшая запись
        }
        for _, e := range graph[cur.node] {
            nd := cur.dist + e.weight
            if nd < dist[e.to] {
                dist[e.to] = nd
                heap.Push(pq, Item{nd, e.to})
            }
        }
    }
    return dist
}
```

**Топологическая сортировка (алгоритм Кана):**

```go
func topoSort(n int, edges [][]int) []int {
    graph := make([][]int, n)
    indeg := make([]int, n)
    for _, e := range edges {
        graph[e[0]] = append(graph[e[0]], e[1])
        indeg[e[1]]++
    }
    var queue []int
    for i := 0; i < n; i++ {
        if indeg[i] == 0 {
            queue = append(queue, i)
        }
    }
    var order []int
    for len(queue) > 0 {
        node := queue[0]
        queue = queue[1:]
        order = append(order, node)
        for _, next := range graph[node] {
            indeg[next]--
            if indeg[next] == 0 {
                queue = append(queue, next)
            }
        }
    }
    if len(order) != n {
        return nil // цикл
    }
    return order
}
```

**Грабли:**
- Дейкстра **не работает** с отрицательными весами — используй Bellman-Ford.
- В Дейкстре нельзя помечать узел как финально посещённый до извлечения из кучи.
- Пропускай устаревшие записи в куче: `if cur.dist > dist[cur.node] { continue }`.

---

## 16. 2-D DP

**Когда:** grid paths, LCS, edit distance, knapsack, палиндромы.

**Шаблон:**

```go
// dp[i][j] = ответ для подзадачи, определяемой двумя индексами

func template(s1, s2 string) int {
    m, n := len(s1), len(s2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    // база dp[0][*] и dp[*][0]
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if s1[i-1] == s2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[m][n]
}
```

**Классика — Longest Common Subsequence:**

```go
func longestCommonSubsequence(text1, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    return dp[m][n]
}
```

**Классика — unique paths в сетке:**

```go
func uniquePaths(m, n int) int {
    dp := make([][]int, m)
    for i := range dp {
        dp[i] = make([]int, n)
        dp[i][0] = 1
    }
    for j := 0; j < n; j++ {
        dp[0][j] = 1
    }
    for i := 1; i < m; i++ {
        for j := 1; j < n; j++ {
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
        }
    }
    return dp[m-1][n-1]
}
```

**Оптимизация памяти до O(n):**

Когда `dp[i][j]` зависит только от `dp[i-1][*]` и `dp[i][j-1]` — достаточно двух строк или одной.

```go
func uniquePathsOpt(m, n int) int {
    dp := make([]int, n)
    for j := range dp {
        dp[j] = 1
    }
    for i := 1; i < m; i++ {
        for j := 1; j < n; j++ {
            dp[j] += dp[j-1] // dp[j] = старое dp[i-1][j] + свежее dp[i][j-1]
        }
    }
    return dp[n-1]
}
```

**Грабли:**
- Индексы `+1` в таблице (строки/массивы 1-индексированы в DP, 0-индексированы в исходных данных) — легко запутаться.
- При оптимизации памяти следи за тем, в каком порядке перезаписываешь клетки.

---

## 17. Bit Manipulation

**Когда:** single number, подсчёт битов, subsets через маски, power of two.

**Базовые операции:**

```go
x & y       // AND
x | y       // OR
x ^ y       // XOR
^x          // NOT
x << k      // сдвиг влево
x >> k      // сдвиг вправо

x & (1 << k)       // проверить k-й бит
x | (1 << k)       // поставить k-й бит
x &^ (1 << k)      // сбросить k-й бит (в Go — оператор AND NOT)
x ^ (1 << k)       // переключить k-й бит

x & (x - 1)        // сбросить самый младший единичный бит
x & -x             // выделить самый младший единичный бит
```

**Классика — single number через XOR:**

```go
func singleNumber(nums []int) int {
    result := 0
    for _, x := range nums {
        result ^= x
    }
    return result
}
```

**Классика — количество установленных битов (Brian Kernighan):**

```go
func hammingWeight(n uint32) int {
    count := 0
    for n != 0 {
        n &= n - 1 // сбрасываем младший установленный бит
        count++
    }
    return count
}
```

**Шаблон — перебор всех подмножеств через битовые маски:**

```go
func allSubsets(nums []int) [][]int {
    n := len(nums)
    var result [][]int
    for mask := 0; mask < (1 << n); mask++ {
        var subset []int
        for i := 0; i < n; i++ {
            if mask&(1<<i) != 0 {
                subset = append(subset, nums[i])
            }
        }
        result = append(result, subset)
    }
    return result
}
```

**Грабли:**
- В Go `&^` — это оператор AND NOT (не `& ~` как в C).
- `1 << k` — если `k` большое, `int` может не хватить. Явно пиши `int64(1) << k` при необходимости.
- Приоритет операций: `a & b == c` парсится как `a & (b == c)`. **Ставь скобки:** `(a & b) == c`.

---

## Общие советы по Go в алгоритмических задачах

**Вход/выход:**

```go
// Быстрое чтение для больших входов:
import (
    "bufio"
    "fmt"
    "os"
)

var reader = bufio.NewReader(os.Stdin)
var writer = bufio.NewWriter(os.Stdout)
defer writer.Flush()
```

**Полезные идиомы:**

```go
// max/min для int (встроены в Go 1.21+)
a := max(x, y)
b := min(x, y)

// Сортировка срезов
sort.Ints(arr)
sort.Slice(arr, func(i, j int) bool { return arr[i] < arr[j] })

// Обратная сортировка
sort.Sort(sort.Reverse(sort.IntSlice(arr)))

// Копия среза (важно для backtracking!)
cp := make([]int, len(src))
copy(cp, src)

// Очистка map
for k := range m {
    delete(m, k)
}
// или просто: m = make(map[K]V)

// Срез как стек
stack = append(stack, x)          // push
x := stack[len(stack)-1]          // peek
stack = stack[:len(stack)-1]      // pop

// Срез как очередь (медленно из-за shift — для огромных входов используй deque через container/list)
x := queue[0]
queue = queue[1:]
```

**Какие могут быть проблемы при решении на Go:**
- Срез — это вьюшка на массив. При сохранении среза в результат — копируй.
- Map возвращает нулевое значение для отсутствующего ключа. Используй `, ok` для проверки.
- Go не имеет generic структур данных в стандартной библиотеке (кроме `container/heap`, `container/list`) — многое пишется руками. Это утомляет, но помогает понять, что происходит внутри.

---

Если захочешь углубиться в конкретный паттерн — скажи какой, разберу задачи LeetCode с этой темой подробнее.
