# vue-todo

> Vue TodoMVC demo with database and REST api  

## Database Setup

``` bash
$ git clone https://github.com/thanhdat21293/rest_api_todo_vue

$ cd rest_api_todo_vue

$ npm install
```

 - Bật PostgreSQL, tạo 1 cơ sở dữ liệu

 - Chỉnh sửa cấu hình theo cơ sở dữ liệu vừa tạo ở file `config/config.json`
 
 - Bật REST server:
 
``` bash
$ npm test 
$ node index.js
```

## Build Setup

 - Chỉnh sửa tất cả đường dẫn đến REST server đang chạy, mặc định là `http://localhost:3000` trong file `/src/components/Todo.vue`, ví dụ: 
 
 ```js
 ... 
 axios.get(`http://localhost:3000/api/todo-mvc/all)
 ... 
 ```

``` bash

$ cd Vue-TodoMVC

$ npm install

$ npm run dev

```

For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

# REST api with ExpressJS

> Project rest_api_todo_vue có mục đích cung cấp một server chứa cơ sở dữ liệu (CSDL) với bảng todolist... và các REST api làm việc với chúng 

### Cấu trúc chính 
```bash
 rest_api_todo_vue/
 --| config
 -----| config.json          // nơi khai báo các thông số để kết nối đến CSDL  
 --| migrations
 -----| 001.do.Todo.sql      // file sql chứa các câu lệnh tương tác với CSDL 
 --| test
 -----| 01_CreateTable.js    // file test để kiểm tra có kết nối được với CSDL được không
 --| index.js                // file khởi tạo server cùng các REST api 
 --| pgp.js                  // file khởi tạo pg-promise để kết nối ứng dụng node với PostgreSQL server 
 --| package.json            // file khai báo các thông tin cơ bản của ứng dụng, node module cần cài đặt...
 --|....

```

### pg-promise 

> Để tương tác được ứng dụng nodejs với CSDL PostgreSQL, ta cần một thư viện trung gian kết nối. pg-promise là một trong số đó.
 
Thiết lập kết nối đến CSDL
 
 - file `config/config.json`
```js 
{ //Ba môi trường làm việc
  "development": { 
    "username": "postgres",  // username và password để đăng nhập vào PostgreSQL 
    "password": "abc",
    "database": "vue_rest",  // tên CSDL 
    "host"    : "localhost", // địa chỉ ip, localhost tương ứng với 127.0.0.1 
    "port"    : 5432,        // cổng kết nối 
    "dialect" : "postgres",
    "schema"  : "public",    // tên schema chứa các bảng 
    "logging" : false
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host"    : "payroll",
    "dialect" : "postgres"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host"    : "127.0.0.1",
    "dialect" : "postgres"
  }
}
```

 - `pgp.js` 
 Sử dụng module pg-promise, require các thông số từ `config/config.json` truyền vào biến cn và export ra để sử dụng 
 
 ```javascript
...
const config = require(path.join(__dirname, 'config', 'config.json'))[env];
... 
const cn = {
  host: config.host,
  port: config.port,
  database: config.database,
  user: config.username,
  password: config.password
};

... 
const pgp = require('pg-promise')(options);

module.exports.db = pgp(cn);
module.exports.config = config;
```
 
### EpxressJS 

- file `index.js`

Đây là file chạy chính cho phép ta bật một nodejs server, có khả năng kết nối đến CSDL nhờ sử dụng đến pg-promise ở file `pgp.js`

```javascript

const {db, } = require('./pgp');
const express = require('express');
... 

// sử dụng middleware bodyParser để xử lí dữ liệu thô, JSON, text, mã hóa URL... 
app.use (bodyParser.urlencoded ({
	extended: true
}));

```

Có 6 REST api tương ứng với các hành động đọc, thêm, sửa, xóa dữ liệu.

Dữ liệu trả về ở dạng json, nhằm phục vụ cho việc client-side rendering.

```javascript
app.use (bodyParser.json());
```

Cú pháp cho từng api có dạng tổng quát như sau, có thể kiểm tra riêng không cần giao diện với công cụ Postman:  
 ```bash 
    http://<địa chỉ ip>:<cổng kết nối>//<đường dẫn api>
 ```
```javascript
// Dùng phương thức GET 
app.get('/api/todo-mvc/all', (req, res) => {
    // gọi đến biến db làm việc với CSDL, thực thi câu lệnh sql 
    db.any('SELECT * FROM todolist ORDER BY id DESC')
    // đợi có kết quả
    .then((data) => {
        // trả lại dữ liệu dạng json 
        res.json(data)
    })
    // bắt lỗi nếu có 
    .catch(error => {
        console.log(error)
    });
});

// Dùng phương thức POST 
app.post('/api/todo-mvc/addTodo', (req, res) => {
    // truyền vào các tham số cần thiết cho câu lệnh sql 
    let title1 = req.body.title;
    let completed1 = req.body.completed;
    // method task được dùng để thực thi nhiều câu lệnh lồng nhau 
    db.task(t => {
        // "thêm mới dữ liệu vào bảng todolist", truyền vào tham số phía trên
        return t.any("INSERT INTO todolist(title, completed) VALUES($1, $2)", [title1, completed1])
            // sau khi thêm mới, liệt kê lại toàn bộ dữ liệu 
            .then(() => {
                return t.any('SELECT * FROM todolist ORDER BY id DESC');
            })
    })
    // trả về kiểu json 
    .then((data) => {
        res.json(data)
    })
    .catch(error => {
        console.log(error)
    });
});

// các hàm dưới cũng tương tự 
app.post('/api/todo-mvc/removeTodo', (req, res) => {
    ...
});

app.post('/api/todo-mvc/editTodo', (req, res) => {
    ...
});

app.post('/api/todo-mvc/completeTodo', (req, res) => {
    ... 
});

app.post('/api/todo-mvc/removeCompleted', (req, res) => {
    ... 
});

// khai báo cổng chạy server, hiện lên trên màn hình console 
const port = 3000;
app.listen(port, () => {
    console.log('Ready for GET requests on http://localhost:' + port);
});

```

# Đổi Vue TodoMVC sang cấu trúc của Vuecli
* Tách thành `Todo` và `Footer` components
### Todo Component
* Route: `/todo-mvc/:status` với status là `all`, `active`, `completed`. Đổi từ thẻ `<a>` sang `<router-link>`

```js
// src/router/index.js
// khai báo route vào todo-mvc
import Todo from '@/components/Todo'
Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'Hello',
      component: Hello
    },
    {
      path: '/todo-mvc/:status',
      name: 'Todo',
      component: Todo
    }
  ]
})
```

```html
// src/components/Todo.vue
<li><router-link :to="{name: 'Todo', params: {status: 'all'}}" :class="{ selected: visibility == 'all' }">All</router-link></li>
<li><router-link :to="{name: 'Todo', params: {status: 'active'}}" :class="{ selected: visibility == 'active' }">Active</router-link></li>
<li><router-link :to="{name: 'Todo', params: {status: 'completed'}}" :class="{ selected: visibility == 'completed' }">Completed</router-link></li>
```

* Vì tất cả các routes `/todo-mvc/:status` đều dùng chung components nên khi routes thay đổi vì component này cũng không đc render lại. Để theo dõi routes và hiển thị dữ liệu chính xác, ta có `watch` biến `$route`

