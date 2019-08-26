---
layout:     post
title:      JavaScript 异步编程 - Automation
subtitle:   
date:       2019-08-26
author:     Sam
header-img: 
catalog: true
tags:
    - 技术
    - JavaScript
---
```Javascript
function automation (gen) {
    function continuer(verb, arg) {
        var result;
        try {
            result = generator[verb](arg);
        } catch (err) {
            return Promise.reject(err);
        }
        if (result.done) {
            return result.value;
        } else {
            return Promise.resolve(result.value).then(onFulfilled, onRejected);
        }
    }
    var generator = gen();
    var onFulfilled = continuer.bind(continuer, 'next');
    var onRejected = continuer.bind(continuer, 'throw');
    return onFulfilled();
};

function red() {
  console.log('red');
}

function green() {
  console.log('green');
}

function yellow() {
  console.log('yellow');
}

function light (fn, timer){
    return new Promise(resolve => {
      setTimeout(function() {
        fn()
        resolve()
      }, timer)
    })
}

function *start() {
  
    yield light(red, 3000)
    yield light(green, 1000)
    yield light(yellow, 2000)

}

automation(start)
```