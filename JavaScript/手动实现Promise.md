```JavaScript
// 实现的Promise需要符合Promise A plus规范

const PROMISE_STATUS_PENDING = "pending";
const PROMISE_STATUS_FULFILLED = "fulfilled";
const PROMISE_STATUS_REJECT = "reject";

// 封装工具函数
function execTryCatch(execFn, value, resolve, reject) {
  try {
    const result = execFn(value);
    resolve(result);
  } catch (error) {
    reject(error);
  }
}
// 手写promise
class qbPromise {
  constructor(executor) {
    this.status = PROMISE_STATUS_PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledFns = [];
    this.onRejectFns = [];
    const resolve = (value) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_FULFILLED;
          this.value = value;
          this.onFulfilledFns.forEach((fn) => fn());
        });
      }
    };
    const reject = (reason) => {
      if (this.status === PROMISE_STATUS_PENDING) {
        queueMicrotask(() => {
          if (this.status !== PROMISE_STATUS_PENDING) return;
          this.status = PROMISE_STATUS_REJECT;
          this.reason = reason;
          this.onRejectFns.forEach((fn) => fn());
        });
      }
    };
    executor(resolve, reject);
  }

  then(onFulfilled, onReject) {
    const defaultOnRejected = (err) => {
      throw err;
    };
    onReject = onReject || defaultOnRejected;

    const defaultOnFulfilled = (value) => {
      return value;
    };
    onFulfilled = onFulfilled || defaultOnFulfilled;

    return new qbPromise((resolve, reject) => {
      // 状态已经确定后的then方法调用
      if (this.status === PROMISE_STATUS_FULFILLED && onFulfilled) {
        execTryCatch(onFulfilled, this.value, resolve, reject);
      }

      if (this.status === PROMISE_STATUS_REJECT && onReject) {
        execTryCatch(onReject, this.reason, resolve, reject);
      }
      // 将回调函数放到数组中
      if (this.status === PROMISE_STATUS_PENDING) {
        if (onFulfilled)
          this.onFulfilledFns.push(() => {
            execTryCatch(onFulfilled, this.value, resolve, reject);
          });
        if (onReject)
          this.onRejectFns.push(() => {
            execTryCatch(onReject, this.reason, resolve, reject);
          });
      }
    });
  }

  catch(onReject) {
    return this.then(undefined, onReject);
  }
  finally(onFinally) {
    this.then(
      () => {
        onFinally();
      },
      () => {
        onFinally();
      }
    );
  }
  // 一些静态方法 resolve, reject, all, allSettled, race, any
  // resolve
  static resolve(value) {
    return new qbPromise((resolve) => resolve(value));
  }
  // reject
  static reject(reason) {
    return new qbPromise((resolve, reject) => reject(reason));
  }
  // all
  static all(promises) {
    const values = [];
    return new qbPromise((resolve, reject) => {
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            values.push(res);
            // 关键在判断什么时候resolve
            if (values.length === promises.length) {
              resolve(values);
            }
          },
          (err) => {
            reject(err);
          }
        );
      });
    });
  }
  // allSettled
  static allSettled(promises) {
    const values = [];
    return new qbPromise(resolve => {
      promises.forEach((promise) => {
        promise.then(
          (res) => {
            values.push({status: "fulfilled", value: res});
            if (values.length === promises.length) {
              resolve(values);
            }
          },
          (err) => {
            values.push({status: "rejected", reason: err})
            if (values.length === promises.length) {
              resolve(values);
            }
          }
        );
      })
    });
  }
  // race
  static race(promises) {
    return new qbPromise((resolve, reject) => {
      promises.forEach(promise => {
        promise.then(
          res => {
            resolve(res)
       }, err => {
            reject(err)
        })
      })
    })
  }
  // any
  static any(promises) {
    return new qbPromise(resolve => {
      promises.forEach(promise => {
        promise.then(res => {
          resolve(res)
        })
      })
    })
  }
}
const p1 = new qbPromise((resolve, reject) => {
  setTimeout(() => {
    resolve("123");
  }, 1000);
});
const p2 = new qbPromise((resolve, reject) => {
  setTimeout(() => {
    reject("456");
  }, 200);
});
const p3 = new qbPromise((resolve, reject) => {
  setTimeout(() => {
    // resolve("789");
    reject(111)
  }, 3000);
});

qbPromise.any([p1, p2, p3]).then((res) => {
  console.log(res)
}).catch(err => console.log("err:" + err))
```
