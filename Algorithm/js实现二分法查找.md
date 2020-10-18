```
const arr1 = [1,4,5,8,12,16,18]

function binarySearch(arr, num) {
    let len = arr.length
    let leftIndex = 0
    let rightIndex = len - 1

    while(leftIndex <= rightIndex) {
        let mid = Math.floor((leftIndex + rightIndex)/2)
        if(num === arr[mid]) { // 找到返回mid
            return mid
        } else if (num > arr[mid]) { // 比中间值大，说明在 mid 到 rightIndex 之间；否则就在 mid 到 leftIndex 之间
            leftIndex = mid + 1
        } else {
            rightIndex = mid - 1
        }
    }
    return -1 // 没找到返回-1
}

console.log(binarySearch(arr1, 12))
```
