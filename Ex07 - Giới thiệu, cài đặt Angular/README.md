# Ex07: Giới thiệu và cài đặt Angular
Create by: TungBT - Bui Thanh Tung <<10/05/2021>>

![N|Solid](https://cdn.worldvectorlogo.com/logos/angular-3.svg)


# Giới thiệu
Angular là một Javascript Framework được phát triển bởi google để xây dựng các Single Page Application (SPA) bằng JavaScript , HTML và TypeScript. Angular cung cấp các tính năng tích hợp cho animation , http service, materials và có các tính năng như auto-complete , navigation , toolbar , menus ,… Code được viết bằng TypeScript , biên dịch thành JavaScript và hiển thị giống nhau trong trình duyệt. Các bài hướng dẫn về angular này sẽ giúp bạn tìm hiểu và sử dụng Angular 8.

# Cài đặt môi trường

Bạn cần phải cài đặt các công cụ sau:

- Nodejs: là một nền tảng Server side được xây dựng dựa trên Javascript Engine (V8 Engine). Node.js = Môi trường Runtime + Các thư viện Javascript
- Npm: NPM viết tắt của Node package manager là một công cụ tạo và quản lý các thư viện lập trình Javascript cho Node.js.
- Angular CLI: Cung cấp câu lệnh giúp tạo nhanh các components, services và modules.
- Công cụ lập trình IDE: Visual Studio Code hoặc Sublime Text để lập trình trên đó. Nhưng tôi khuyên bạn nên sử dụng Visual Studio Code bởi nó nhẹ và nhiều tính năng hữu ích.

## 1.Cài đặt Nodejs và npm

Download phiên bản mới nhất của Node.js phù hợp với hệ điều hành của bạn từ trang: https://nodejs.org/en/download/.

Trang chủ của Nodejs như sau:

![N|CSDL](https://viettuts.vn/images/angular/angular7/cai-dat-moi-truong-angular7-1.png)

Dựa trên hệ điều hành của bạn, cài đặt gói tương ứng. Khi nodejs được cài đặt, npm cũng sẽ được cài đặt cùng với nó. Để kiểm tra xem npm có được cài đặt hay không, gõ lệnh npm -v trong terminal. Nó sẽ hiển thị phiên bản của npm.

![N|CSDL](https://viettuts.vn/images/angular/angular7/cai-dat-moi-truong-angular7-2.png)

## 2. Angular CLI

Cài đặt Angular rất đơn giản với sự trợ giúp của Angular CLI. Truy cập trang chủ https://cli.angular.io/ của Angular sẽ cài được bản angular mới nhất.

Để cài đặt một phiên bản Angular bất kỳ ví dụ v7.3.2, bạn gõ lệnh sau:

```shell script
npm install -g @angular/cli
```

Sau khi nhập lệnh xong các bạn chờ một lúc để máy cài đặt sau khi xong bạn có thể gõ lệnh ng --version để kiểm tra phiên bản của Angular:


Chúng ta đã hoàn tất việc cài đặt môi trường để phát triển Angular

# Tạo project đầu tiên
Bài này sẽ hướng dẫn bạn làm thế nào để tạo dự án đầu tiên trong Angular và giải thích về cấu trúc của dự án Angular. Hãy đảm bảo rằng bạn đã hoàn thành cài đặt môi trường cho Angular.

Về tài liệu học Angular, bạn có thể tìm thấy tại trang chủ https://cli.angular.io/

Bạn sẽ thấy các lệnh sau trên trang web:

```shell script
ng new my-dream-app // tạo một dự án angular có tên my-dream-app
cd my-dream-app // đi đến thư mực my-dream-app
ng serve // run dự án
```

![N|CSDL](https://viettuts.vn/images/angular/angular7/tao-du-an-dau-tien-trong-angular7-1.png)


Đây là thư mục dự án C:\Angular\Angular7\my-app:

![N|CSDL](https://viettuts.vn/images/angular/angular7/tao-du-an-dau-tien-trong-angular7-2.png)


Run dự án Angular
Thay đổi thư mục trong dòng lệnh bằng cách gõ lệnh sau:

```shell script
cd my-app
```

Run dự án bằng cách gõ lệnh sau:

```shell script
ng serve
```

![N|CSDL](https://viettuts.vn/images/angular/angular7/tao-du-an-dau-tien-trong-angular7-4.png)


Đến đây chúng ta đã run thành công dự án đầu tiên. Như bạn thấy trong ô màu đỏ của ảnh trên là link của ứng dụng. Mặc định angular sẽ lắng nghe port 4200.

Mở trình duyệt lên và gõ http://localhost:4200 để xem kết quả:


![N|CSDL](https://viettuts.vn/images/angular/angular7/tao-du-an-dau-tien-trong-angular7-5.png)


Cấu trúc một dự án angular

Cấu trúc thư mục của một angular bao gồm:

- e2e: Thư mục này dùng để chứa các tập tin dành cho mục đích testing.
- node_modules: Gói npm được cài đặt là node_modules. Bạn có thể mở thư mục và xem các gói có sẵn.
- src: Đây là thư mục sẽ chứa toàn bộ source code của ứng dụng Angular các bạn nhé.
- .editorconfig: Chứa các cấu hình liên quan đến phần Editor để chỉnh sửa source code như: indent_size, max_line_length,…
- .gitignore: Đây là tập tin metadata của Git, chứa thông tin những tập tin hoặc thư mục sẽ bị ignore không được commit lên Git Repository.
- angular-cli.json: Đây là tập tin chứa cấu hình cho Angular CLI, giúp chúng ta có thể build ứng dụng Angular.
- karma.conf.js: Tập tin cấu hình cho Karma, liên quan nhiều đến phần testing.
- package-lock.json: Dùng để lock version cho các Node.js module dependencies
- package.json: Tập tin cấu hình cho Node.js module dependencies
- protractor.conf.js: Tập tin cấu hình cho Protractor, liên quan nhiều đến phần testing
- README.md: Tập tin này thường được sử dụng để cho các hệ thống Git hiển thị thông tin về Git Repository của chúng ta.
- tslint.json: Tập tin cấu hình để kiểm tra lỗi cho các tập tin .ts (TypeScript) trong Angular project.

Trong thư mục src có nhưng thư mục sau:

- app: Đây là thư mục sẽ chứa toàn bộ code của ứng dụng Angular.
- assets: Thư mục này sẽ chứa các file ảnh, CSS, custom JavaScript của ứng dụng Angular
- environments: Chúng ta có thể viết ứng dụng chạy trên nhiều môi trường khác nhau, đây chính là thư mục giúp chúng ta làm định nghĩa các tập tin cấu hình cho những môi trường khác nhau đó.
- favicon.ico: Icon của ứng dụng Angular.
- index.html: Trang chủ của ứng dụng Angular.
- main.ts: Chứa code bootstrapping cho ứng dụng Angular.
- polyfill.ts: Dùng để định nghĩa các chuẩn để ứng dụng của chúng ta có thể chạy được trên mọi trình duyệt.
- style.css: Định nghĩa style CSS cho ứng dụng Angular.
- test.ts: Code để chạy test.
- tsconfig.json: Tập tin định nghĩa việc compile cho TypeScript.




Thannks,
TungBT.
