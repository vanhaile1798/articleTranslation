# CALLBACKS, PROMISES AND ASYNC/AWAIT


Đây là sự giải thích của tôi về cách sử dụng code bất đồng bộ (Asynchronous) trong Javascript ở bậc cao.


Note: Tôi sẽ sử dụng arrow function trong bài viết này. Cơ bản là tôi sẽ thay thế các anonymous function như thế này:

```
function() {}
function(a) {}      
function(a,b) {}

```
bằng các arrow function như thế này:

```
() => {}          
a => {}            
(a,b) => {}
```

Bạn có một function sẽ in ra 1 string sau một khoảng thời gian ngẫu nhiên:

```
function printString(string){
  setTimeout(
    () => {
      console.log(string)
    }, 
    Math.floor(Math.random() * 100) + 1
  )
}
```

Hãy thử in ra các chữ cái A, B, C:

```
function printAll(){
  printString("A")
  printString("B")
  printString("C")
}
printAll()
```

Bạn sẽ nhận ra rằng A, B, C sẽ được in ra theo một thứ tự ngẫu nhiên và các thứ tự này sẽ khác nhau mỗi lần bạn gọi hàm printAll.

Đó là bởi vì các function này là function bất đồng bộ. Mỗi function được thực thi theo đúng thứ tự, nhưng mỗi function lại có một setTimeout riêng. (Nghĩa là khi được gọi thì theo thứ tự như trên, nhưng khi in ra kết quả thì không theo tứ tự đó vì thứ tự của kết quả in ra phụ thuộc vào setTimeout). Các function này sẽ không đợi đến khi nào function trước hoàn thành mới chạy.

Điều này gây khó khăn cho chúng ta, vì vậy hãy sửa lại bằng callback.

## CALLBACK

CALLBACK LÀ MỘT FUNCTION ĐƯỢC TRUYỀN VÀO FUNCTION KHÁC. Khi mà function đầu chạy xong, thì function thứ 2 sẽ chạy.

```
function printString(string, callback){
  setTimeout(
    () => {
      console.log(string)
      callback()
    }, 
    Math.floor(Math.random() * 100) + 1
  )
}
```

Bạn có thể nhận thấy là sửa function ban đầu để dùng nó với callback quá EZ.

Hãy thử in ra A, B, C một lần nữa:

```
function printAll(){
  printString("A", () => {
    printString("B", () => {
      printString("C", () => {})
    })
  })
}
printAll()
```

Code bây giờ trở lên "xấu xí", nhưng mà ít nhất là nó chạy. Mỗi lần bạn gọi printAll, bạn sẽ nhận được 1 kết quả giống nhau.

Vấn đề với callback là nó tạo ra một cái gọi là "Callback Hell". Cơ bản là nếu bạn lồng các function vào nhau thì code trở nên khó hiểu.

## PROMISES

Promise cố gắng giải quyết vấn đề này (lồng nhiều function). Hãy thay đổi code để có thể sử dụng Promise:

```
function printString(string){
  return new Promise((resolve, reject) => {
    setTimeout(
      () => {
       console.log(string)
       resolve()
      }, 
     Math.floor(Math.random() * 100) + 1
    )
  })
}
```

Bạn có thể thấy là code mới trông vẫn quen thuộc. Bạn gói cả function trong 1 Promise, và thay vì gọi callback, bạn có thể resolve Promise (hoặc reject nếu có lỗi). Function sẽ trả về object Promise này.

Hãy thử lại in ra A, B, C:

```
function printAll(){
  printString("A")
  .then(() => {
    return printString("B")
  })
  .then(() => {
    return printString("C")
  })
}
printAll()
```

Cái này gọi là Promise Chain. Bạn có thể code trả về một kết quả (là Promise), và nó sẽ gửi kết quả mới trả về này cho function tiếp theo.

Code không bao gồm các function lồng nhau nữa nhưng trông vẫn lộn xộn!

Bằng cách dùng arrow function, chúng ta có thể bỏ "wrapper" function ( bỏ { }). Code bây giờ trông rõ ràng hơn nhưng vẫn có nhiều ngoặc thừa:

```
function printAll(){
  printString("A")
  .then(() => printString("B"))
  .then(() => printString("C"))
}
printAll()
```

## AWAIT

Await cơ bản là giúp cú pháp của Promise dễ hiểu hơn. Nó giúp cho code bất đồng bộ trông giống như code đồng bộ, giúp chúng ta dễ hiểu hơn.

Function printString vẫn giữ nguyên giống như ở trên mục Promise.

Ta lại in ra A, B, C:

```
async function printAll(){
  await printString("A")
  await printString("B")
  await printString("C")
}
printAll()
```

Yeah ... trông đẹp hơn rồi.

Bạn có thể nhận ra chúng ta dùng keyword async cho function printAll. Keyword này báo cho Javascript biết chúng ta đang dùng cú pháp async/await, và keyword này là cần thiết nếu muôn dùng Await. Nghĩa là chúng ta không thể dùng Await ngoài function. Hầu hết code trong Javascript chạy trong function nên điều này không quan trọng.

NHƯNG CÒN THÊM NỮA

Function printAll không trả về cái gì và không phụ thuộc vào cái gì, và cái chúng ta quan tâm là thứ tự chạy của function này. Nhưng nếu bạn muốn lấy kết quả của function đầu, truyền vào function thứ 2, và lại truyền kết quả của function thứ 2 sang function thứ 3?

Thay vì in ra string mỗi lần, hãy viết 1 function nối các string và truyền nó vào function tiếp theo.

### CALLBAKCS

Đây là code dùng callback:

```
function addString(previous, current, callback){
  setTimeout(
    () => {
      callback((previous + ' ' + current))
    }, 
    Math.floor(Math.random() * 100) + 1
  )
}
```

Và đây là cách gọi nó: 

```
function addAll(){
  addString('', 'A', result => {
    addString(result, 'B', result => {
      addString(result, 'C', result => {
       console.log(result) // Prints out " A B C"
      })
    })
  })
}
addAll()
```

Code không được đẹp lắm.

Đây là code khi dùng Promise:

```
function addString(previous, current){
  return new Promise((resolve, reject) => {
    setTimeout(
      () => {
        resolve(previous + ' ' + current)
      }, 
      Math.floor(Math.random() * 100) + 1
    )
  })
}
```

Còn đây là cách gọi nó:

```
function addAll(){  
  addString('', 'A')
  .then(result => {
    return addString(result, 'B')
  })
  .then(result => {
    return addString(result, 'C')
  })
  .then(result => {
    console.log(result) // Prints out " A B C"
  })
}
addAll()
```

Dùng arrow function thì code gọn hơn: (do bỏ {})

```
function addAll(){  
  addString('', 'A')
  .then(result => addString(result, 'B'))
  .then(result => addString(result, 'C'))
  .then(result => {
    console.log(result) // Prints out " A B C"
  })
}
addAll()
```

Thế này thì dễ hiểu hơn, đặc biệt là bạn muốn thêm nhiều then. Nhưng vẫn có quá nhiều dấu ngoặc.

Còn đây là cách dùng với Await:

```
async function addAll(){
  let toPrint = ''
  toPrint = await addString(toPrint, 'A')
  toPrint = await addString(toPrint, 'B')
  toPrint = await addString(toPrint, 'C')
  console.log(toPrint) // Prints out " A B C"
}
addAll()
```

Đây là bản dịch tiếng Việt của bài viết: https://medium.com/front-end-hacking/callbacks-promises-and-async-await-ad4756e01d90