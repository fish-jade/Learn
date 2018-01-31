### 数组中的两个数的和等于目标数
![](http://ww1.sinaimg.cn/large/9da83df8ly1fnxrqx8nhfj20ku0dvdg4.jpg)
```
var twoSum = function(nums, target) {
    let arr = []
    nums.forEach((val,index) => {
        if(val<target){
            arr[index]=val
        }
    })
    for(let i=0;i<arr.length-1;i++){
        for(let j = i+1;j<arr.length;j++){
            if(arr[i]+arr[j]==target){
                return[i,j]
            }
        }
    }
};
```

### 快排
```
let quickSort = function (arr) {
    if(arr.length<=1){return arr}
    let pivotIndex = Math.floor(arr.length / 2);

　　let pivot = arr.splice(pivotIndex, 1)[0]; 
　　let left = [];

　　let right = [];
　　arr.forEach((val,index)=>{
　　    if (arr[index] < pivot) {

　　　　　　left.push(arr[index]);

　　　　} else {

　　　　　　right.push(arr[index]);

　　　　}
　　})
　　return [...quickSort(left),pivot,...quickSort(right)]
}
```
==注意==
```
　　若要使用，let pivot = arr[pivotIndex]，则return [...quickSort(left),...quickSort(right)]，不能将pivot加进去
　　还是不能使用let pivot = arr[pivotIndex]的方式
　　这样当数组个数多时，会报递归次数过多的错误
　　具体原因不清楚
　　
```
### 字符串中最长不重复字串
![](http://ww1.sinaimg.cn/large/9da83df8ly1fnykungfhgj20ok0bxt94.jpg)
```
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    let strArr = s.split('')
    let arr = []
    let returnArr = []
    strArr.forEach((val,index) => {
      let arrIndex = arr.findIndex(value=>{
        return val === value
      })
      if(arrIndex === -1){
        arr.push(val)
      }else{
        returnArr.push(arr)
        //console.log(index,arr)
        //console.log(index,returnArr)
        
        arr = arr.filter(function(v,i){
          return i>arrIndex 
        })
        arr.push(val)
      }
    })
	  returnArr.push(arr)
    let point = returnArr[0].length
    let returnStr = returnArr[0].join('')
    returnArr.forEach((val,index)=>{
      if(point < val.length){
        point = val.length
        returnStr = val.join('')
      }
    })
    return returnStr.length
};
```
### 两数组的中位数
![](http://ww1.sinaimg.cn/large/9da83df8ly1fnzl3onxdnj20oc0a93yj.jpg)
```
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number}
 */
var findMedianSortedArrays = function(nums1, nums2) {
    function quickSort(arr){
      if(arr.length<=1){return arr}
      let pointIndex = Math.floor(arr.length/2)
      let point = arr.splice(pointIndex, 1)[0]
      let left = []
      let right = []
      arr.forEach((val,index)=>{
        if(val<point){
          left.push(val)
        }else{
          right.push(val)
        }
      })
      return [...quickSort(left),point,...quickSort(right)]
    }
    let nums = [...nums1,...nums2]
    nums = quickSort(nums)
    console.log(nums)
    let num = nums.length%2
    console.log(num)
    let index = Math.floor(nums.length/2)
    console.log(index)
    if(num === 1){
      return nums[index]
    }else{
      return (nums[index]+nums[index-1])/2
    }
};
```