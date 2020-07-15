# Binary Indexed Tree
給定一個array，長度為n，我們對這個array的操作有兩種:

1. 計算從0到i的sum
2. 修改array[i]的值

要達到這兩個operation，有兩種做法:

1. 計算sum時每次都從0加到i(需要O(n))。修改時直接修改array[i]的值。(需要O(1))
2. 先把所有0加到i的值算好存在另一個array裡面叫做sum。query sum時直接取用sum[i]的值(需要O(1))。修改則要從算一次sum(需要O(n))

此兩種做法都各有優缺，但仍不夠好。需要一種方法能夠兩個operation都bound在O(log n)內才行。
因此使用BIT的方法。

## 定義

> BITree[y] is the parent of BITree[x], if and only if y can be obtained by removing the last set bit from the binary representation of x, that is y = x – (x & (-x)).
> 
根據上述定義，我們的BIT長相都是固定的，如下圖:

![](https://i.imgur.com/Pv9P2Qa.png)

## Operation 1

有一點要稍微注意，BITree是一個array，而且是1-base的array。Node 0是root不代表任何值。

知道我們的tree會長這樣之後，再追加一些條件給他。
- Node x是Node y的child，則y = x - (x & (-x))，而且，BITree[x]的值存的是array[y]\(include)到array[x]\(exclude)的sum。

那我們計算sum的方法就可以由child沿著parent加回到root就可以了，pesudocode如下:
```
getSum(x): Returns the sum of the sub-array arr[0,...,x]
// Returns the sum of the sub-array arr[0,...,x] using BITree[0..n], which is constructed from arr[0..n-1]
1) Initialize the output sum as 0, the current index as x+1.
2) Do following while the current index is greater than 0.
...a) Add BITree[index] to sum
...b) Go to the parent of BITree[index].  The parent can be obtained by removing
     the last set bit from the current index, i.e., index = index - (index & (-index))
3) Return sum.
```

Time Complexity: O(log n)

## Operation 2

當要修改值的時候，必須要考慮到這次的修改會在BIT上的哪些位置受到影響。

例如:

- 當array[8]的值被修改時，會受到影響的是BIT的9, 10, 12這三個位置。
- 當array[4]的值被修改時，會受到影響的是BIT的5, 6, 8這三個位置。

可以得到一個結論: 受影響的i在BIT上Node i+1以及此Node的右邊的node會受到影響，右邊的Node的index一定比i+1來得大，而且除此之外，他們都有共通的parent，所以這個被修改的值一定也被包含在他們之中。

在bit上的表示以第一個例子為例，9 = 1001, 10 = 1010, 12 = 1100, 而他們的parent 8 = 1000。觀察得知，parent最後面有幾個連續的0他就會有幾個child(根據定義)。而要從9到10的方法很簡單，<mark>只要將last set bit + 1即可</mark>。

再來看到第二個例子，5 = 101, 6 = 110, parent 4 = 100，5, 6都會受到影響沒有錯，但8不是4的child阿。

8的parent是0，確實沒錯，對4修改會影響到BIT的8這個位置，也就是說，parent的nighbor也會受到影響。回顧我們剛剛得到的結論，會發現8 = 1000，當6在last set bit加上1之後，確實就會進位成8了，所以結論其實沒有問題!

那麼，可以把tree想成這樣:
![](https://i.imgur.com/bje1JzF.png)

而update的operation，只要按照上面所說，在會被影響的地方加上修改的變化量，也就是加完現在這個BIT的entry就將index的last set bit + 1到下一個被影響的BIT entry直到index超過n為止。

詳細的pesudocode如下:
```
update(x, val): Updates the Binary Indexed Tree (BIT) by performing arr[index] += val
// Note that the update(x, val) operation will not change arr[].  It only makes changes to BITree[]
1) Initialize the current index as x+1.
2) Do the following while the current index is smaller than or equal to n.
...a) Add the val to BITree[index]
...b) Go to parent of BITree[index].  The parent can be obtained by incrementing
     the last set bit of the current index, i.e., index = index + (index & (-index))
```