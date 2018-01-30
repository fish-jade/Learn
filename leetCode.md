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