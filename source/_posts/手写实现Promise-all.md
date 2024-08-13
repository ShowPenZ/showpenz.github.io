---
title: 手写实现Promise.all
date: 2020-07-09 15:12:55
tags: Promise
top:
categories: js
---

```javascript
const PromiseAll = (array)=>{
    let tempValue = [];
    let i = 0;
    
    return new Promise((resolve,reject)=>{
       const next = () => {
            array[i].then((res)=>{
                i++;
                tempValue.push(res)
                
                return i === array.length ? resolve(tempValue):next()
            }).catch((res)=>{
                reject(`reject ${res}`)
            })
        }
        next();
    })
}

let p1 = Promise.resolve(1);
let p2 = Promise.resolve(2);
let p3 = Promise.resolve(3);

PromiseAll([p1,p2,p3]).then(res=>{
    console.log(res) // [1,2,3]
})

let p4 = Promise.resolve(2);
let p5 = Promise.resolve(2);
let p6 = Promise.reject(1);

PromiseAll([p4,p5,p6]).then(res=>{
    console.log(res) // reject  1
})


```