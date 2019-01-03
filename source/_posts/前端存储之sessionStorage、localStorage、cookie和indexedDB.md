---
title: 前端存储之sessionStorage、localStorage、cookie和indexedDB
date: 2019-1-3
tags: [JS, 浏览器, 本地存储] 
---


## 前端存储 localStorage、sessionStorage、Cookie

### 一、相同点

localStorage、sessionStorage、Cookie共同点：都是保存在浏览器端，且同源的。

### 二、不同点： 

1》传递方式不同 

- cookie数据始终在同源的http请求中携带(即使不需要)，即cookie在浏览器和服务器间来回传递。 
- sessionStorage和loaclStorage不会自动把数据发给服务器，仅在本地保存。 

2》数据大小不同 

- cookie存放数据大小为4K左右 。有个数限制（各浏览器不同），一般不能超过20个。 因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。 
- sessionStorage和localStorage存放数据大小为一般为5MB,而且它仅在客户端（即浏览器）中保存，不参与和服务器的通信。 

3》数据有效期不同 

- sessionStorage仅在当前会话下有效，关闭页面或浏览器后被清除。
- localStorage生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除localStorage信息，否则这些信息将永远存在。
- cookie只在设置cookie过期时间之前一直有效，即使窗口或浏览器关闭。 

4》作用域不同 

- sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面； 
- localStorage在所有同源窗口中都是共享的； 
- cookie也是在所有同源窗口中都是共享的。 
- Web Storage支持事件通知机制，可以将数据更新的通知发送给监听者。 
- Web Storage的api接口使用更方便。 

### 三、localStorage和sessionStorage使用时使用相同的API：

```javascript
    localStorage.setItem("key","value");//以“key”为名称存储一个值“value”

    localStorage.getItem("key");//获取名称为“key”的值

    localStorage.removeItem("key");//删除名称为“key”的信息。

    localStorage.clear();​//清空localStorage中所有信息
```


---

## HTML5本地存储 浏览器数据库——IndexedDB


indexedDB 整体结构

![indexedDB 整体结构](https://segmentfault.com/img/bV4R5H)

类比sql型数据库，IndexedDB中的DB（数据库）就是sql中的DB，而Object Store(存储空间)则是数据表,Item则等于表中的一条记录。

### 一、IndexedDB 具有以下特点

（1）键值对储存。

（2）异步。

（3）支持事务。

（4）同源限制 IndexedDB 受到同源限制，每一个数据库对应创建它的域名。

（5）储存空间大 IndexedDB 的储存空间比 LocalStorage 大得多，一般来说不少于 250MB，甚至没有上限。

（6）支持二进制储存。 IndexedDB 不仅可以储存字符串，还可以储存二进制数据（ArrayBuffer 对象和 Blob 对象）。


### 二、基本概念

- 数据库：IDBDatabase 对象，数据库是一系列相关数据的容器。每个域名（严格的说，是协议 + 域名 + 端口）都可以新建任意多个数据库。

- 对象仓库：IDBObjectStore 对象，每个数据库包含若干个对象仓库（object store）。它类似于关系型数据库的表格。

- 索引： IDBIndex 对象，为了加速数据的检索，可以在对象仓库里面，为不同的属性建立索引。

- 事务： IDBTransaction 对象，事务对象提供error、abort和complete三个事件，用来监听操作结果。

- 操作请求：IDBRequest 对象

- 指针： IDBCursor 对象

- 主键集合：IDBKeyRange 对象


### 三、操作流程

#### 3.1 打开数据库

使用 IndexedDB 的第一步是打开数据库，使用indexedDB.open()方法。

    var request = window.indexedDB.open(databaseName, version);
这个方法接受两个参数，第一个参数是字符串，表示数据库的名字。如果指定的数据库不存在，就会新建数据库。第二个参数是整数，表示数据库的版本。如果省略，打开已有数据库时，默认为当前版本；新建数据库时，默认为1。

indexedDB.open()方法返回一个 IDBRequest 对象。这个对象通过三种事件error、success、upgradeneeded，处理打开数据库的操作结果。

**（1）error 事件**

error事件表示打开数据库失败。


    request.onerror = function (event) {
      console.log('数据库打开报错');
    };

**（2）success 事件**

success事件表示成功打开数据库。

    var db;
    
    request.onsuccess = function (event) {
      db = request.result;
      console.log('数据库打开成功');
    };
这时，通过request对象的result属性拿到数据库对象。

**（3）upgradeneeded 事件**

如果指定的版本号，大于数据库的实际版本号，就会发生数据库升级事件upgradeneeded。


    var db;
    
    request.onupgradeneeded = function (event) {
      db = event.target.result;
    }
这时通过事件对象的target.result属性，拿到数据库实例。


#### 3.2 新建数据库
新建数据库与打开数据库是同一个操作。如果指定的数据库不存在，就会新建。不同之处在于，后续的操作主要在upgradeneeded事件的监听函数里面完成，因为这时版本从无到有，所以会触发这个事件。

通常，新建数据库以后，第一件事是新建对象仓库（即新建表）。


    request.onupgradeneeded = function(event) {
      db = event.target.result;
      var objectStore = db.createObjectStore('person', { keyPath: 'id' });
    }
上面代码中，数据库新建成功以后，新增一张叫做person的表格，主键是id。

更好的写法是先判断一下，这张表格是否存在，如果不存在再新建。


    request.onupgradeneeded = function (event) {
      db = event.target.result;
      var objectStore;
      if (!db.objectStoreNames.contains('person')) {
        objectStore = db.createObjectStore('person', { keyPath: 'id' });
      }
    }
主键（key）是默认建立索引的属性。比如，数据记录是{ id: 1, name: '张三' }，那么id属性可以作为主键。主键也可以指定为下一层对象的属性，比如{ foo: { bar: 'baz' } }的foo.bar也可以指定为主键。

如果数据记录里面没有合适作为主键的属性，那么可以让 IndexedDB 自动生成主键。


    var objectStore = db.createObjectStore(
      'person',
      { autoIncrement: true }
    );
上面代码中，指定主键为一个递增的整数。

新建对象仓库以后，下一步可以新建索引。


    request.onupgradeneeded = function(event) {
      db = event.target.result;
      var objectStore = db.createObjectStore('person', { keyPath: 'id' });
      objectStore.createIndex('name', 'name', { unique: false });
      objectStore.createIndex('email', 'email', { unique: true });
    }
上面代码中，IDBObject.createIndex()的三个参数分别为索引名称、索引所在的属性、配置对象（说明该属性是否包含重复的值）。

#### 3.3 新增数据

新增数据指的是向对象仓库写入数据记录。这需要通过事务完成。


    function add() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' });
    
      request.onsuccess = function (event) {
        console.log('数据写入成功');
      };
    
      request.onerror = function (event) {
        console.log('数据写入失败');
      }
    }
    
    add();
上面代码中，写入数据需要新建一个事务。新建时必须指定表格名称和操作模式（"只读"或"读写"）。新建事务以后，通过IDBTransaction.objectStore(name)方法，拿到 IDBObjectStore 对象，再通过表格对象的add()方法，向表格写入一条记录。

写入操作是一个异步操作，通过监听连接对象的success事件和error事件，了解是否写入成功。

3.4 读取数据
读取数据也是通过事务完成。

    
    function read() {
       var transaction = db.transaction(['person']);
       var objectStore = transaction.objectStore('person');
       var request = objectStore.get(1);
    
       request.onerror = function(event) {
         console.log('事务失败');
       };
    
       request.onsuccess = function( event) {
          if (request.result) {
            console.log('Name: ' + request.result.name);
            console.log('Age: ' + request.result.age);
            console.log('Email: ' + request.result.email);
          } else {
            console.log('未获得数据记录');
          }
       };
    }
    
    read();
上面代码中，objectStore.get()方法用于读取数据，参数是主键的值。

#### 3.5 遍历数据
遍历数据表格的所有记录，要使用指针对象 IDBCursor。

    
    function readAll() {
      var objectStore = db.transaction('person').objectStore('person');
    
       objectStore.openCursor().onsuccess = function (event) {
         var cursor = event.target.result;
    
         if (cursor) {
           console.log('Id: ' + cursor.key);
           console.log('Name: ' + cursor.value.name);
           console.log('Age: ' + cursor.value.age);
           console.log('Email: ' + cursor.value.email);
           cursor.continue();
        } else {
          console.log('没有更多数据了！');
        }
      };
    }

    readAll();
上面代码中，新建指针对象的openCursor()方法是一个异步操作，所以要监听success事件。

3.6 更新数据
更新数据要使用IDBObject.put()方法。

    
    function update() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });
    
      request.onsuccess = function (event) {
        console.log('数据更新成功');
      };
    
      request.onerror = function (event) {
        console.log('数据更新失败');
      }
    }
    
    update();
上面代码中，put()方法自动更新了主键为1的记录。

3.7 删除数据
IDBObjectStore.delete()方法用于删除记录。

    
    function remove() {
      var request = db.transaction(['person'], 'readwrite')
        .objectStore('person')
        .delete(1);
    
      request.onsuccess = function (event) {
        console.log('数据删除成功');
      };
    }
    
    remove();
3.8 使用索引
索引的意义在于，可以让你搜索任意字段，也就是说从任意字段拿到数据记录。如果不建立索引，默认只能搜索主键（即从主键取值）。

假定新建表格的时候，对name字段建立了索引。


    objectStore.createIndex('name', 'name', { unique: false });
现在，就可以从name找到对应的数据记录了。


    var transaction = db.transaction(['person'], 'readonly');
    var store = transaction.objectStore('person');
    var index = store.index('name');
    var request = index.get('李四');
    
    request.onsuccess = function (e) {
      var result = e.target.result;
      if (result) {
        // ...
      } else {
        // ...
      }
    }


---
## 参考

[1、localStorage、sessionStorage、Cookie的区别及用法](https://segmentfault.com/a/1190000012057010)

[2、前端存储之sessionStorage、localStorage、cookie和indexedDB](https://blog.csdn.net/qq_29132907/article/details/80389398)

[3、浏览器数据库 IndexedDB 入门教程](http://www.ruanyifeng.com/blog/2018/07/indexeddb.html)

[4、IndexedDB API](https://wangdoc.com/javascript/bom/indexeddb.html#indexeddb-%E5%AF%B9%E8%B1%A1)

[5、IndexedDB使用与出坑指南](https://segmentfault.com/a/1190000006924681)








