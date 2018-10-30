Chú thích: (...)*: Ký hiệu giải thích của người dịch

		        (...)**: Câu/Đoạn dịch cần được review

# CHƯƠNG 7

*"Câu hỏi Máy móc có thể suy nghĩ hay không cũng giống như là câu hỏi Tàu ngầm có thể lặn hay không."*

- Edsger Dijkstra, *Mối đe dọa đối với ngành khoa học tính toán*

![](https://eloquentjavascript.net/img/chapter_picture_7.jpg)

Trong các chương "Dự án", tôi sẽ dừng bạn với lý thuyết mới một lúc, và thay vào đó chúng ta sẽ cùng nhau giải quyết một chương trình máy tính. Lý thuyết là quan trọng để học lập trình, nhưng đọc và hiểu một chương trình thực tế cũng quan trọng.

Dự án trong chương này là xây dựng một cỗ máy tự động, một chương trình nhỏ thực hiện tác vụ trong một thế giới ảo. Cỗ máy tự động này sẽ là một con robot đưa mail nhặt và thả các gói hàng.

## MEADOWFIELD

Ngôi làng Meadowfield không quá lớn. Nó bao gồm 11 địa điểm với 14 con đường giữa các địa điểm này. Nó được mô tả bởi mảng các con đường này:

```
const roads = [
  "Alice's House-Bob's House",   "Alice's House-Cabin",
  "Alice's House-Post Office",   "Bob's House-Town Hall",
  "Daria's House-Ernie's House", "Daria's House-Town Hall",
  "Ernie's House-Grete's House", "Grete's House-Farm",
  "Grete's House-Shop",          "Marketplace-Farm",
  "Marketplace-Post Office",     "Marketplace-Shop",
  "Marketplace-Town Hall",       "Shop-Town Hall"
];
```

![](https://eloquentjavascript.net/img/village2x.png)

Mạng lưới các con đường của ngôi làng này tạo thành một đồ thị. Một đồ thị là nhiều điểm (các địa điểm trong ngôi làng) với các đường nối giữa chúng (là các con đường). Đồ thị này sẽ là thế giới mà robot sẽ di chuyển trong đó.

Một mảng các ký tự không dễ để xử lý. Thứ chúng ta quan tâm là các địa điểm mà chúng ta có thể đến được từ một địa điểm. Hãy cùng nhau chuyển đổi danh sách các con đường này thành một kiểu dữ liệu mà, với mỗi địa điểm, cho chúng ta biết các địa điểm nào có thể đi đến từ điểm đó.

```
function buildGraph(edges) {
  let graph = Object.create(null);
  function addEdge(from, to) {
    if (graph[from] == null) {
      graph[from] = [to];
    } else {
      graph[from].push(to);
    }
  }
  for (let [from, to] of edges.map(r => r.split("-"))) {
    addEdge(from, to);
    addEdge(to, from);
  }
  return graph;
}

const roadGraph = buildGraph(roads);
```

Với một mảng các cạnh, buildGraph tạo một map object, với mỗi đỉnh, chứa một mảng các đỉnh liên kết với đỉnh đó.

Nó (buildGraph)* sử dụng method split để biến các string là các con đường, có dạng là "Điểm bắt đầu - Điểm kết thúc", thành các mảng hai phần tử bao gồm điểm bắt đầu và điểm kết thúc dưới dạng các string tách rời nhau.

## NHIỆM VỤ

Robot sẽ di chuyển vòng quanh làng. Có các gói hàng ở nhiều địa điểm khác nhau, và mỗi gói sẽ có địa chỉ tới là các địa điểm tới là các địa điểm khác với địa điểm hiện tại của gói hàng. Robot sẽ nhặt các gói hàng khi nó đi qua các nơi có gói hàng đó và sẽ đưa các gói hàng tới đích đến nếu nó đến địa điểm đó.

Cỗ máy tự động này phải quyết định rằng, tại mỗi vị trí, sẽ đi đâu tiếp. Nó sẽ hoàn thành nhiệm vụ khi tất cả các gói hàng đã được chuyển phát.

Để có thể giả lập được quá trình này, chúng ta phải định nghĩ một thế giới ảo mà có thể mô tả quá trình này. Thế giới này cho chúng ta biết robot ở đâu và các gói hàng ở đâu. Khi robot quyết định đi đến địa điểm nào đó, chúng ta phải cập nhật thế giới này để thể hiện tình trạng mới của nó.

Nếu bạn đang nghĩ về lập trình hướng đối tượng, ý định ban đầu của bạn có thể là định nghĩ các object cho các vật trong thế giới đó: một class cho robot, một cho gói hàng, và có thể là một cho các địa điểm. Các class này có thể có các thuộc tính miêu tả trạng thái hiện tải, như là một chồng các gói hàng tại một địa điểm, cái mà chúng ta thay đổi khi cập nhật thế giới này. (nghĩa là thay đổi trạng thái của thế giới ảo).

Điều đó là sai lầm.

At least (Ít nhất)**, nó thường là thế. Sự thật là cái gì nghe có vẻ là object không tự có nghĩa là nó sẽ là một object trong chương trình. Tự động viết class cho tất cả các khái niệm trong ứng dụng của bạn có thường cho bạn nhiều object liên kết chặt chẽ với nhau mà mỗi object sẽ có trạng thái thay đổi bên trong nó. Các chương trình như vậy rất khó hiểu và thường dễ bị lỗi.

Thay vào đó, hãy thu gọn trạng thái của ngôi làng thành một tập các giá trị. Trong đó có vị trí hiện tại của robot và các gói hàng chưa được chuyển phát, mỗi gói hàng đó sẽ có vị trí hiện tại của nó và địa chỉ điểm đến. Như vậy đó.

Hãy tạo tập giá trị này sao cho chúng ta không thay đổi trạng thái khi robot di chuyển mà thay vào đó tạo một trạng thái mới sau bước di chuyển của robot.

```
class VillageState {
  constructor(place, parcels) {
    this.place = place;
    this.parcels = parcels;
  }

  move(destination) {
    if (!roadGraph[this.place].includes(destination)) {
      return this;
    } else {
      let parcels = this.parcels.map(p => {
        if (p.place != this.place) return p;
        return {place: destination, address: p.address};
      }).filter(p => p.place != p.address);
      return new VillageState(destination, parcels);
    }
  }
}
```

Move method là nơi mà hành động robot di chuyển diễn ra. Đầu tiên nó kiểm tra xem có con đường nào nối địa điểm hiện tại của robot đến điểm đến không, nó trả về trạng thái cũ vì bước di chuyển là không hợp lệ (không thể di chuyển đến một địa điểm mà không có con đường nào giữa điểm hiện tại và điểm đó)*.

Sau đó nó tạo một trạng thái mới trong đó điểm đến trở thành vị trí mới của robot. Nhưng nó cũng cần tạo một tập các gói hàng mới - các gói hàng mà robot đang mang theo (các gói mà tại vị trí hiện tại của robot) cần được di chuyển đến một địa điểm mới. Và các gói hàng được gán cho địa điểm mới này cần được chuyển phát - và chúng cần phải được loại bỏ khỏi tập các gói hàng chưa được chuyển phát. Lời gọi tới map xử lý phần di chuyển, còn lời gọi tới filter xử lý phần chuyển phát. (Mỗi lần gói hàng được di chuyển thì nó được cập nhật vị trị hiện tại, đến khi nào vị trí hiện tại của nó mà trùng với điểm đến của gói hàng, thì hàm filter sẽ loại bỏ nó ra khỏi danh sách các gói hàng chưa được chuyển phát)*

Các object gói hàng không bị thay đổi khi chúng bị di chuyển mà được tạo lại. Move method cho chúng ta một trạng thái mới của ngôi làng và trạng thái cũ vẫn được giữ nguyên.

```
let first = new VillageState(
  "Post Office",
  [{place: "Post Office", address: "Alice's House"}]
);
let next = first.move("Alice's House");

console.log(next.place);
// → Alice's House
console.log(next.parcels);
// → []
console.log(first.place);
// → Post Office
```

Bước di chuyển khiến cho gói hàng được chuyển phát, và điều này được thể hiện trong trạng thái mới của ngôi làng. Nhưng trạng thái ban đầu vẫn mô tả trạng thái mà trong đó robot ở bưu điện (Post Office) và gói hàng chưa được chuyển phát.

## DỮ LIỆU KHÔNG THAY ĐỔI


Cấu trúc dữ liệu mà không thay đổi được gọi là immutable hoặc persistent. Chúng giống như string và số ở điểm là nó luôn giữ nguyên, thay vì thay đổi tại các thời điểm khác nhau.

Trong Javascript, hầu hết mọi thứ đều có thể bị thay đổi, vì vậy làm việc với các giá trị mà được cho là không đổi yêu cầu một vài giới hạn. Có một function gọi là Object.freeze cái mà thay đổi object sao cho sự ghi đè lên các thuộc tính của nó bị bỏ qua. Bạn có thể sử dụng nó để đảm bảo rằng các objects của bạn không bị thay đổi, nếu bạn muốn cẩn thận.Freezing does require the computer to do some extra work, and having updates ignored is just about as likely to confuse someone as having them do the wrong thing. (Sự đóng băng object này yêu cầu máy tính thực hiện thêm công việc, và sự bỏ qua việc ghi đè lên thuộc tính của object này dễ dàng khiến cho ai đó khó hiểu giống như là khiến họ làm gì đó sai)**. Vì vậy tôi thường muốn cho mọi người thấy rằng một object không nên đụng đến và hi vọng là họ nhớ nó.

```
let object = Object.freeze({value: 5});
object.value = 10;
console.log(object.value);
// → 5
```

Tại sao tôi lại không định thay đổi object khi mà ngôn ngữ lập trình mong muốn tôi làm như vậy?

Vì nó giúp tôi hiểu chương trình của mình. Nó liên quan tới quản lý độ phức tạp. Khi mà object trong hệ thống của tôi được cố định, I can consider operations on them in isolation (tôi có thể xử lý chúng một cách tách biệt)** - di chuyển tới nhà của Alice từ một vị trí nào đó luôn trả về một trạng thái mới giống nhau (functional programming)*. Khi mà các object thay đổi theo thời gian, thì nó tăng độ phức tạp của chương trình. (When objects change over time, that adds a whole new dimension of complexity to this kind of reasoning). (CÂU NÀY CHƯA DỊCH ĐƯỢC)

Với một hệ thống nhỏ như cái mà chúng ta xây dựng trong chương này, chúng ta có thể xử lý độ phức tạp đó. But the most important limit on what kind of systems we can build is how much we can understand (Nhưng một hạn chế quan trọng liên quan tới loại hệ thống nào chúng ta có thể xây dựng là chúng ta có thể hiểu nó đến mức nào)**. Bất cứ thứ gì khiến cho code của bạn dễ hiểu hơn thì khiến xây dựng một hệ thống phức tạp dễ dàng hơn.

Thật không may là, mặc dù hiểu một hệ thống xây dựng trên cấu trúc dữ liệu không đổi là dễ dàng, nhưng thiết kế một cấu trúc dữ liệu không đổi như vậy, đặc biệt là khi ngôn ngữ lập trình bạn dùng không hỗ trợ điều đó, có thể hơi khó. Chúng ta sẽ tìm cơ hội để sử dụng cấu trúc dữ liệu không đỏi trong quyển sách này, nhưng chúng ta cũng sẽ sử dụng cấu trúc dữ liệu có thay đổi.

## SỰ GIẢ LẬP

Một robot chuyển phát sẽ quan sát thế giới (ngôi làng) và quyết định xem nó muốn di chuyển theo hướng nào. Như vậy, chúng ta có thể nói rằng một robot là một function nhận vào một VillageState object và trả về tên của một điểm gần đó.

Bởi vì chúng ta muốn robot có khả năng ghi nhớ các thứ (ghi nhớ các địa điểm cần di chuyển tới), để nó lên kế hoạch và thực hiện kế hoạch, ta cũng truyền cho nó bộ nhớ và cho phép nó trả về một bộ nhớ mới (nghĩa là sau mỗi bước di chuyển thì bộ nhớ của nó sẽ được cập nhật). Do đó, cái mà robot trả về sẽ là một object chứa hướng di chuyển mà nó muốn di chuyển và một bộ nhớ cái mà được truyền lại cho nó vào lần tiếp theo nó được gọi.

```
function runRobot(state, robot, memory) {
  for (let turn = 0;; turn++) {
    if (state.parcels.length == 0) {
      console.log(`Done in ${turn} turns`);
      break;
    }
    let action = robot(state, memory);
    state = state.move(action.direction);
    memory = action.memory;
    console.log(`Moved to ${action.direction}`);
  }
}
```

Xem xét những gì một robot phải làm để giải quyết một trạng thái cho trước. Nó phải nhặt tất cả các gói hàng bằng cách tới tất cả các địa điểm có gói hàng và chuyển phát chúng bằng cách tới các địa điểm là địa chỉ nhận của các gói hàng đó, và việc chuyển phát này chỉ được thực hiện sau khi robot đã nhặt gói hàng.

Chiến lược ngu ngốc nhất nào có thể làm được việc chuyển phát này? Robot có thể di chuyển theo hướng tự do tại mỗi địa điểm. Nghĩa là, rất có thể, nó dần dần nhặt tất cả các gói hàng và tại một điểm nào đó tới được địa điểm nơi mà các gói hàng được chuyển phát tới (địa chỉ nhận của các gói hàng).

Nó giống như thế này:

```
function randomPick(array) {
  let choice = Math.floor(Math.random() * array.length);
  return array[choice];
}
```

```
function randomRobot(state) {
  return {direction: randomPick(roadGraph[state.place])};
}
```

Nhớ rằng Math.random() trả về một số có giá trị trong khoảng giữa 0 và 1 - nhưng luôn luôn dưới 1. Nhân số đó với độ dài của một mảng mà xử lý giá trị vừa nhân xong đó bằng Math.floor() sẽ cho chúng ta một chỉ số ngẫu nhiên cho mảng đó.

Bởi vì robot này không cần nhớ gì cả, nó sẽ bỏ qua tham số thứ 2 (nhớ rằng function trong Javascript có thể được gọi với số lượng tham số nhiều hơn số lượng function đó cần mà không gây ảnh hưởng) và bỏ thuộc tính memory trong object trả về của function này.

Để robot phức tạp này hoạt động, trước tiên chúng ta cần tạo một trạng thái mới (của ngôi làng) với vài gói hàng. Chúng ta tạo trạng thái mới này sử dụng static method (được viết bằng cách thêm một thuộc tính trực tiếp cho constructor).

```
VillageState.random = function(parcelCount = 5) {
  let parcels = [];
  for (let i = 0; i < parcelCount; i++) {
    let address = randomPick(Object.keys(roadGraph));
    let place;
    do {
      place = randomPick(Object.keys(roadGraph));
    } while (place == address);
    parcels.push({place, address});
  }
  return new VillageState("Post Office", parcels);
};
```

(Tạo một state mới cho ngôi làng, trong đó robot sẽ có vị trí ban đầu ở Post Office và có 5 gói hàng nằm ở các địa điểm bất kỳ và địa chỉ nhận bất kỳ)*

Chúng ta không muốn bất kỳ gói hàng nào được gửi từ một địa điểm giống với địa chỉ nhận của gói hàng đó. Vì lý do này, vòng lặp do tiếp tục việc chọn địa điểm đặt gói hàng khi mà địa điểm đặt gói hàng giống với địa chỉ nhận của gói hàng.

Hãy thiết lập một thế giới ảo.

```
runRobot(VillageState.random(), randomRobot);
// → Moved to Marketplace
// → Moved to Town Hall
// → …
// → Done in 63 turns
```

Robot mất nhiều lần để chuyển phát các gói hàng bởi vì nó không được lên kế hoạch trước (các hướng di chuyển là ngẫu nhiên). Chúng ta sẽ giải quyết vấn đề này sớm.

Để có thể có một cái rõ ràng hơn về sự giả lập này, bạn có thể dùng function runRobotAnimation có sẵn trong môi trường lập trình của chương này. Nó chạy chương trình robot này, nhưng thay vì in ra văn bản, nó cho bạn thấy hình ảnh của robot di chuyển quanh bản đồ ngôi làng. (Cho mọi người dễ hình dung cách di chuyển của robot).

runRobotAnimation(VillageState.random(), randomRobot);

Cách mà function runRbotAnimation này được thực thi vẫn là một điều bí ẩn với chúng ta, nhưng sau khi bạn đọc xong chương cuối cùng của cuốn sách này, chương thảo luận về sự tích hợp của Javascript vào trình duyệt web, bạn sẽ có thể biết được cách mà nó hoạt động.

## THE MAIL TRUCK'S ROUTE

Chúng ta nên làm tốt hơn robot di chuyển ngẫu nhiên. Một cách cải thiện robot dễ dàng là dùng gợi ý từ cách mà mail được chuyển phát trong thế giới thưc. Nếu chúng ta tìm được một con đường mà đi qua tất cả các địa điểm trong ngôi làng, robot có thể đi theo con đường đó 2 lần. Đây là một con đường như thế (bắt dầu từ bưu điện (Post Office)).

```
const mailRoute = [
  "Alice's House", "Cabin", "Alice's House", "Bob's House",
  "Town Hall", "Daria's House", "Ernie's House",
  "Grete's House", "Shop", "Grete's House", "Farm",
  "Marketplace", "Post Office"
];
```

Để thực thi một robot đi theo một con đường được xác định trước, chúng ta cần tận dụng bộ nhớ của robot. Robot sẽ lưu phần còn lại của con đường trong bộ nhớ và sẽ loại bỏ địa điểm đầu tiên trong bộ nhớ sau mỗi bước di chuyển.

```
function routeRobot(state, memory) {
  if (memory.length == 0) {
    memory = mailRoute;
  }
  return {direction: memory[0], memory: memory.slice(1)};
}
```

Robot này đã nhanh hơn nhiều rồi. Nó sẽ mất số bước tối đa là 26 (2 lần con đường được xác định trước, con đường được xác định trước gồm 13 địa điểm, mất 13 bước để đi hết con đường này).

```
runRobotAnimation(VillageState.random(), routeRobot, []);
```

## TÌM ĐƯỜNG

Tuy nhiên, tôi sẽ không gọi việc đi theo một con đường một cách không suy nghĩ là một hành vi thông minh. 
Robot có thể hoạt động hiệu quả hơn nếu nó thay đổi hành vi sao cho phù hợp với các công việc cần được hoàn thành.

Để làm điều đó, nó cần phải có khả năng có chủ đích tới một gói hàng cho trước hoặc tới địa điểm mà gói hàng cần được chuyển phát tới. Làm như vậy, even when the goal is more than one move away, sẽ yêu cầu một function tìm đường.

Tìm đường trên đồ thị là một vấn đề tìm kiếm hay gặp phải. Chúng ta có thể cho rằng một giải pháp (một con đường) là hợp lệ hay không, nhưng chúng ta không thể tính toán giải pháp đó như là cách chúng ta tính toán 2 + 2. Thay vào đó, chúng ta phải tiếp tục tạo ra giải pháp tiềm năng mới cho tới khi chúng ta tìm được giải pháp phù hợp nhất.

Số lượng các đường đi trong một đồ thị là vô hạn. Nhưng khi tìm kiếm một đường đi từ A tới B, chúng ta chỉ quan tâm tới các đường đi xuất phát từ A. Chúng ta cũng không quan tâm các đường đi mà đi tới các địa điểm hai lần - những đường đi đó hoàn toàn không phải là đường đi hiệu quả. Vì vậy những điều trên làm giảm đi số lượng các đường đi mà function tìm đường phải xem xét.

Thật ra chúng ta quan tâm nhất đến đường đi ngắn nhất. Vì vậy chúng ta muốn chắc chắn rằng chúng ta kiểm tra các đường đi ngắn trước khi kiểm tra các đường đi dài hơn. Một cách tiếp cận tốt là xây dựng đường đi từ điểm bắt đầu, khám phá ra tất cả các địa điểm có thể đi đến được mà vẫn chưa tới, cho tới khi đường đi đén nơi cần đến. Theo cách đó, chúng ta sẽ chỉ khám phá các đường đi chúng ta quan tâm, và chúng ta sẽ tìm đường đi ngắn nhất (hoặc một trong những đường đi ngắn nhất, nếu có nhiều đường đi hơn một đường) tới điểm cần đến.

Đây là một function tìm đường:

```
function findRoute(graph, from, to) {
  let work = [{at: from, route: []}];
  for (let i = 0; i < work.length; i++) {
    let {at, route} = work[i];
    for (let place of graph[at]) {
      if (place == to) return route.concat(place);
      if (!work.some(w => w.at == place)) {
        work.push({at: place, route: route.concat(place)});
      }
    }
  }
}
```

Sự khám phá đường đi phải được hoàn thành theo đúng thứ tự - các địa điểm được đến đầu tiên thì phải được khám phá đầu tiên. Chúng ta không thể khám phá một địa điểm ngay khi chúng ta tới địa điểm đó vì điều đó nghĩa là các địa điểm từ địa điểm ta vừa khám phá đó sẽ được khám phá ngay lập tức, và cứ tiếp tục như vậy, mặc dù có thể có những đường đi khác ngắn hơn chưa được khám phá.

Do đó, function này lưu một danh sách công việc. Đó là một mảng các object bao gồm địa điểm mà sẽ được khám phá tiếp theo, cùng với đường đi đưa robot tới địa điểm sẽ được khám phá tiếp này (ví dụ về mảng này, 1 phần tử có dạng: {at: "Alice's House", route: ["Post Office", "Marketplace", "Town Hall", "Bob's House", "Alice's House"]}, và đích đến là một địa điểm ví dụ là Farm thì địa điểm được khám phá tiếp theo sẽ là Alice's House và thuộc tính route là các địa điểm đã đi qua để đến được địa điểm Alice's House)*. Nó bắt đầu với vị trí ban đầu và một đường đi trống, không bao gồm địa điểm nào.

Sự tìm kiếm được thực thi bằng cách lấy giá trị tiếp theo trong danh sách công việc và xét nó, (nghĩa là các con đường đi từ địa điểm ở trong giá trị đó sẽ được xét)*. Nếu một trong các con đường đó là con đường cần tìm (nghĩa là địa điểm cuối cùng của con đường đó là điểm đến)*, một con đường hoàn chỉnh sẽ được trả về. Nếu không thì nếu chúng ta chưa từng xét một địa điểm thì một item sẽ được thêm vào trong danh sách (item này là một object, bao gồm 2 thuộc tính là at và route, khi được thêm vào thì at sẽ là địa điểm chưa từng được xét và route là cách mà robot đến được địa điểm đó)*. Nếu một địa điểm đã được xét trước đó, vì chúng ta xét các đường đi ngắn trước, nên khi xét tiếp địa điểm này chúng ta sẽ tìm ra một con đường dài hơn tới địa điểm đó hoặc một con đường dài như con đường đã tồn tại, và chúng ta không cần phải khám phá nó.

Bạn có thể tưởng tượng nó như là một mạng lưới các con đường đi từ điểm bắt đầu và tỏa ra các địa điểm khác (nhưng không bao giờ quay ngược trở lại điểm bắt đầu). Ngay khi đường đi đầu tiên đi đến địa điểm cần tìm, nó sẽ cho chúng ta đường đi cần tìm.

Code của chúng ta không xử lý tình huống mà không còn item nào trong danh sách công việc bởi vì chúng ta biết rằng đồ thị của chúng ta là liên kết, nghĩa là mọi điểm đều có thể đi tới từ tất cả các điểm khác. Chúng ta luôn luôn có thể tìm được một đường đi giữa 2 điểm, và sự tìm kiếm không bao giờ thất bại.

```
function goalOrientedRobot({place, parcels}, route) {
  if (route.length == 0) {
    let parcel = parcels[0];
    if (parcel.place != place) {
      route = findRoute(roadGraph, place, parcel.place);
    } else {
      route = findRoute(roadGraph, place, parcel.address);
    }
  }
  return {direction: route[0], memory: route.slice(1)};
}
```


Robot này dùng bộ nhớ là một danh sách các hướng để di chuyển, giống như robot theo một đường đi xác định trước. Bất cứ khi nào danh sách hướng đi này rỗng, nó cần phải chỉ ra làm cái gì tiếp theo. Nó lấy gói hàng chưa được chuyển phát đầu tiên trong danh sách các gói hàng, và nếu mà gói hàng này chưa được nhặt, thì nó sẽ tìm một đường tới gói hàng đó. Thay vào đó nếu gói hàng đã được nhặt, thì gói hàng cần được chuyển phát, vì vậy robot tạo một đường đi tới địa chỉ đến của gói hàng.

Hãy xem cách nó làm việc.

```
runRobotAnimation(VillageState.random(), goalOrientedRobot, []);
```                  

Robot này thường hoàn thành tác vụ chuyển phát 5 gói hàng trong khoảng 16 lượt di chuyển. Nó chỉ hơn routeRobot (robot đi theo con đường xác định trước) một chút nhưng chắc chắn chưa phải là tối ưu.

## BÀI TẬP

### Measuring a robot

Thật khó để so sánh khách quan các robot chỉ bằng cách cho chúng giải quyết một vài tình huống. Có thể một robot nhận được công việc dễ dàng hơn hoặc công việc mà nó làm tốt, trong khi robot còn lại không được như vậy.

Viết một function compareRobots truyền vào 2 robot (và bộ nhớ ban đầu của chúng). Nó sẽ sinh ra 100 công việc và bắt mỗi robot làm các công việc này. Khi xong, nó sẽ in ra số lần mỗi robot di chuyển cho một công viêc.

Để công bằng, đảm bảo rằng bạn cho mỗi robot một công việc giống, thay vì tạo ra các công việc khác nhau cho các robot.

```
function compareRobots(robot1, memory1, robot2, memory2) {
  // Your code here
}

compareRobots(routeRobot, [], goalOrientedRobot, []);
```

*Gợi ý:
Bạn phải có một vài thay đổi cho function runRobot, thay vì in ra các sự kiện ra console, trả về số bước robot di chuyển để hoàn thành công việc.

Function đo số bước này có thể, trong vòng lặp, sinh ra các trạng thái khác nhau và đém số bước mỗi robot di chuyển. Khi nó tạo ra đủ số lần đo, nó có thể dùng console.log để in ra số bước trung bình cho mỗi robot, là tổng số lượng của các bước di chuyển chia cho số lần đo.

### Hiệu suất của robot

Bạn có thể viết một robot hoàn thành công việc chuyển phát nhanh hơn goalOrientedRobot? Nếu bạn quan sát hành vi của robot đó, điều ngu ngốc dễ dàng nhận thấy nó làm là gì? Những thứ đó có thể được cải thiện như thế nào?

Nếu bạn đã giải quyết bài tập trước, bạn có thể muốn sử dụng function compareRobots để kiểm tra bạn đã cải tiến robot hay chưa.

```
// Your code here

runRobotAnimation(VillageState.random(), yourRobot, memory);
```

* Gợi ý:
Hạn chế của goalOrientedRobot là nó chỉ xét một gói hàng tại một thời điểm. Nó sẽ thường xuyên đi qua đi lại toàn bộ ngôi làng bởi vì gói hàng nó đang xét ở phía bên kia của ngôi làng, mặc dù có các gói hàng ở gần nó hơn.

Một giải pháp khả thi là tính toán đường đi cho tất cả các gói hàng và sau đó đi theo đường đi ngắn nhất. Chúng ta có thể có được kết quả tốt hơn, nếu có nhiều đường đi ngắn nhất, bằng cách chọn đường đi nhặt gói hàng thay vì chọn đường đi để chuyển phát nó. (Nếu các bước chuyển phát gói hàng bằng số bước đi nhặt gói hàng thì robot sẽ ưu tiên đi nhặt gói hàng).


### Persistent Group

Hầu hết cấu trúc dữ liệu có sẵn trong Javascript không phù hợp cho việc dùng nó mà không thay đổi nó (persistent use). Mảng có method slice và concat, chúng cho phép chúng ta tạo mảng mới mà không làm thay đổi mảng cũ. Nhưng Set không có method nào để tạo một tập dữ liệu mới với các item được thêm vào hay xóa bỏ.

Viết một class mới là PGroup, giống với class Group từ chương 6, chứa một tập các giá trị. Giống như Group, nó có các method add, delete, và has.

Tuy nhiên method add sẽ trả về một PGroup instance(1) với các item được thêm vào và không thay đổi instance cũ. Giống như method add, delete tạo một instance mới và bỏ đi phần tử đã xóa.

Class này có thể được dùng với bất kỳ kiểu dữ liệu nào, không chỉ là string. Nó không cần có hiệu năng tốt khi được dùng với số lượng lớn các giá trị.

Constructor sẽ không là một phần của class interface (nghĩa là không gọi trực tiếp nó khi khởi tạo một instance)* (nhưng bạn chắc chắn sẽ dùng nó bên trong class). Thay vào đó, chúng ta có một instance rỗng là PGroup.empty, được sử dụng là giá trị ban đầu.

Tại sao chúng ta chỉ cần một giá trị PGroup.empty, thay vì có một function tạo một map rỗng mới? (rather than having a function that creates a new, empty map every time?)
(Nghĩa là sao chỉ cần tạo một lần giá trị ban đầu cho tất cả các group sau này mà không phải là khi tạo một group mới thì cần tạo giá trị ban đầu mới cho group đó.
Ví dụ:
```
let group1 = new PGroup([]);
let group2 = new PGroup([]);
let group3 ...
```
)*

```
class PGroup {
  // Your code here
}

let a = PGroup.empty.add("a");
let ab = a.add("b");
let b = ab.delete("a");

console.log(b.has("b"));
// → true
console.log(a.has("b"));
// → false
console.log(b.has("a"));
// → false
```

* Gợi ý:

Các thuận tiện nhất để lưu trữ một tập các giá trị vẫn là mảng vì mảng dễ sao chép.

Khi một giá trị được thêm vào group, bạn có thể tạo một nhóm mới là một mảng là một bản sao của mảng cũ cùng với giá trị được thêm vào (ví dụ: dùng concat). Khi xóa một giá trị thì bạn lọc giá trị đó khỏi mảng.

Constructor của class có thể nhận một mảng là tham số và lưu mảng đó thành thuộc tính duy nhất của instance. Mảng này sẽ không bao giờ được cập nhật, luôn luôn không đổi.

Để thêm một thuộc tính (empty) vào constructor mà thuộc tính này không phải là method, bạn cần phải thêm nó vào constructor sau khi định nghĩa class, như một thuộc tính thông thường.

(
```
	class PGroup {
		constructor(arr) {
			...
		}
		add(value) {
			...
		}
	}

	PGroup.empty = ...
	// Thêm thuộc tính empty như gợi ý ở trên.
```
)*

Bạn chỉ cần một empty instance vì mọi group rỗng đều giống nhau và các instance của class là không đổi. Bạn có thể tạo nhiều group khác nhau từ một group rỗng mà không làm thay đổi group rỗng đó. (Vì PGroup là kiểu dữ liệu không thay đổi tự tạo, nên mỗi lần gọi add, delete trên group rỗng đó các method này đều tạo ra một group mới và không làm thay đỗi group cũ. Vì vậy group rỗng ban đầu sẽ vẫn giữ nguyên giá trị rỗng.)*



## GIẢI THÍCH NGHĨA CỦA TỪ:

(1) instance: ví dụ: ta có một class là Car như sau:

```
class Car {
	constructor(type, color) {
		this.type = type;
		this.color = color;
	}
}

let maserati = new Car('sport', 'blue');
```

Vậy trong ví dụ này maserati là một instance của class Car.

Khi gọi constructor function hoặc constructor của class với keyword new thì object mới được tạo ra sẽ được gọi là một instance của constructor function hoặc của class đó.

Đây là bản dịch của Chapter 7: Project: A Robot - Eloquent Javascript (https://eloquentjavascript.net/07_robot.html)