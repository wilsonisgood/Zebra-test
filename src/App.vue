
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  name: 'App',
  mounted () {
  },
  methods: {
    aboutFunctionConstructor() {
      // 建立物件的三種方式
        // 1.先前我們有說明使用 object literal，也就是大括號的方式來建立物件，
        // 2.或者是使用 new Object( ) 的方式，
        // 3.我們還可以使用**函式建構式（function constructor）**的方式來建立物件。
      // 為何物件要用 function 建立？
      // 為何物件的方法要放在 .prototype
      function Person(firstName, lastName) {
        this.firstName = firstName
        this.lastName = lastName
      }
      Person.prototype.getFullName = function() {
        return this.firstName + ' ' + this.lastName
      }
      const john = new Person('John', 'Doe')
      console.log(john)
      console.log(john.getFullName())
      Person.prototype.getFormalName = function() {
        return this.lastName + ' ' + this.lastName
      }
      const jane = new Person('Jane', 'Doe')
      console.log(jane)
      console.log(jane.getFormalName())
    },
    specifyingArgumentsWithFunctionConstructor() {
      // 示範 argument ，但是functionConstructor 是哪個部分？
      var adder = new Function('a', 'b', 'return a + b');
      adder(2, 6);
      // > 8
    },
    difBetweenFuncConstructAndDecla() {
      let x = 10;
      function createFunction1() {
        let x = 20;
        return new Function('return x;'); // this |x| refers global |x|
      }
      function createFunction2() {
        let x = 20;
        function f() {
          return x; // this |x| refers local |x| above
        }
        return f;
      }
      let f1 = createFunction1();
      console.log(f1());          // 10
      let f2 = createFunction2();
      console.log(f2());          // 20
    }
  },
  aboutPromise() {

    var PENDING = 0;
    var FULFILLED = 1;
    var REJECTED = 2;

    function Promise(fn) {
      // store state which can be PENDING, FULFILLED or REJECTED
      var state = PENDING;

      // store value once FULFILLED or REJECTED
      var value = null;

      // store sucess & failure handlers
      var handlers = [];

      // A_狀態機
          function fulfill(result) {
            state = FULFILLED;
            value = result;
            // 將 handlers 裡面的東西 都做一遍 handle()
            handlers.forEach(handle);
            // 清空
            handlers = null;
          }
          function reject(error) {
            state = REJECTED;
            value = error;
            // 將 handlers 裡面的東西 都做一遍 handle()
            handlers.forEach(handle);
            // 清空
            handlers = null;
          }

      // try 執行 doResolce
          function resolve(result) {
            try {
              // 從 result 裡面拿出 then 
              var then = getThen(result);
              if (then) { // 如果有 then
              // doResolve吃三個參數，fn(要議決的內容), onFulfilled(成功時的callback), onRejected(失敗時的callback)，因此我們可以把要fn訂成該promise的then，也就是doResolve(then.bind(result), resolve, reject)。
              // bind會把執行then時的this綁定到該promise上，因此看起來就像呼叫了該promise的then，如果成功的話就繼續議決(resolve)，如果失敗的話就否決(reject)
                doResolve(then.bind(result), resolve, reject)
                return
              }
              fulfill(result);
            } catch (e) {
              reject(e);
            }
          }

      // 
          function handle(handler) {
            if (state === PENDING) {
              handlers.push(handler);
            } else {
              if (state === FULFILLED &&
                typeof handler.onFulfilled === 'function') {
                handler.onFulfilled(value);
              }
              if (state === REJECTED &&
                typeof handler.onRejected === 'function') {
                handler.onRejected(value);
              }
            }
          }

      // 異步執行
          this.done = function (onFulfilled, onRejected) {
            // ensure we are always asynchronous
            setTimeout(function () {
              handle({
                onFulfilled: onFulfilled,
                onRejected: onRejected
              });
            }, 0);
          }

      doResolve(fn, resolve, reject);
    }


        /**
         * 如果輸入是 promise 就將輸入的 then 取出來回傳
         * Check if a value is a Promise and, if it is,
         * return the `then` method of that promise.
         * @param {Promise|Any} value
         * @return {Function|Null}
         */
        function getThen(value) {
          var t = typeof value;
          // 將 輸入的參數 拿來判斷是 object 或 function，
          if (value && (t === 'object' || t === 'function')) {
            // 取出 輸入參數 的 then 回傳
            var then = value.then;
            if (typeof then === 'function') {
              return then;
            }
          }
          return null;
        }

        /**
         * 確認是否有 潛在危險的 resolver，並且確保 ful 和 rej 只能被執行一次
         * Take a potentially misbehaving resolver function and make sure
         * onFulfilled and onRejected are only called once.
         * 不保證異步
         * Makes no guarantees about asynchrony.
         * @param {Function} fn A resolver function that may not be trusted
         * @param {Function} onFulfilled
         * @param {Function} onRejected
         */
        function doResolve(fn, onFulfilled, onRejected) {
          var done = false;
          try {
            // fn 輸入兩個匿名函式 當參數，
            // 
            fn(
              function (value) {
                if (done) return
                done = true
                onFulfilled(value)
              }, 
              function (reason) {
                if (done) return
                done = true
                onRejected(reason)
              }
            )
          } catch (ex) {
            if (done) return
            done = true
            onRejected(ex)
          }
        }


    this.then = function (onFulfilled, onRejected) {
      // 使 promise 可成為執行鏈
      var self = this;
      return new Promise(function (resolve, reject) {
        return self.done(function (result) {
          if (typeof onFulfilled === 'function') {
            try {
              return resolve(onFulfilled(result));
            } catch (ex) {
              return reject(ex);
            }
          } else {
            return resolve(result);
          }
        }, function (error) {
          if (typeof onRejected === 'function') {
            try {
              return resolve(onRejected(error));
            } catch (ex) {
              return reject(ex);
            }
          } else {
            return reject(error);
          }
        });
      });
    }


  },
};
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
