# Flutter Provider

- Provider là một trong những công cụ quản lý state trong flutter.
- Để sử dụng các package của provider, thêm phần sau vào thư mục __pubspec.yaml__ của project:

```yaml
dependencies:
  provider: ^6.1.2 // Lastest version
```

## Provider

- __Provider__ là package cơ bản nhất trong các loại package của provider. Nó được sử dụng để cung cấp giá trị (thường cho data model) cho bất kỳ vị trí nào trong widget tree khi giá trị của nó thay đổi. Tuy nhiên, nó sẽ chỉ thay đổi dữ liệu chứ không thay đổi UI.

Ví dụ có một class data model:

```dart
class MyModel { 
  String someValue = 'Hello';
  void doSomething() {
    someValue = 'Goodbye';
    print(someValue);
  }
}

```

Để có thể cung cấp data model đó cho widget tree thì cần bọc phần widget tree bằng một __Provider__, tham số truyền vào là data model vừa tạo. Trong widget tree, sử dụng __Consumer__ để tham chiếu tới data model vừa truyền.

Ví dụ về __Provider__:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider<MyModel>(
      //                                <--- Provider
      create: (context) => MyModel(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('My App')),
          body: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              Container(
                  padding: const EdgeInsets.all(20),
                  color: Colors.green[200],
                  child: Consumer<MyModel>(
                    //                    <--- Consumer
                    builder: (context, myModel, child) {
                      return ElevatedButton(
                        child: const Text('Do something'),
                        onPressed: () {
                          // We have access to the model.
                          myModel.doSomething();
                        },
                      );
                    },
                  )),
              Container(
                padding: const EdgeInsets.all(35),
                color: Colors.blue[200],
                child: Consumer<MyModel>(
                  //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return Text(myModel.someValue);
                  },
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

Kết quả thu được:

- Giao diện hiển thị button và text có nội dung "Hello", data từ trong data model.
- Khi nhấn button, data model thay đổi. Tuy nhiên, UI không có sự thay đổi dù data model thay đổi.

## ChangeNotifierProvider

- Không giống như __Provider__, __ChangeNotifierProvider__ lắng nghe các thay đổi trong data model. Khi có thay đổi, nó sẽ xây dựng lại bất kỳ widget nào trong Consumer.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<MyModel>( //      <--- ChangeNotifierProvider
      create: (context) => MyModel(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('My App')),
          body: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[

              Container(
                  padding: const EdgeInsets.all(20),
                  color: Colors.green[200],
                  child: Consumer<MyModel>( //                  <--- Consumer
                    builder: (context, myModel, child) {
                      return ElevatedButton(
                        child: Text('Do something'),
                        onPressed: (){
                          myModel.doSomething();
                        },
                      );
                    },
                  )
              ),

              Container(
                padding: const EdgeInsets.all(35),
                color: Colors.blue[200],
                child: Consumer<MyModel>( //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return Text(myModel.someValue);
                  },
                ),
              ),

            ],
          ),
        ),
      ),
    );
  }
}

class MyModel with ChangeNotifier { //                          <--- MyModel
  String someValue = 'Hello';

  void doSomething() {
    someValue = 'Goodbye';
    print(someValue);
    notifyListeners();
  }
}
```

- Sử dụng __ChangeNotifierProvider__ trong hàm build thay cho __Provider__, đồng thời, class MyModel cần được __extend ChangeNotifier__ hoặc __with ChangeNotifier__. Điều này sẽ cung cấp quyền truy cập vào __notifyListeners()__ và bất cứ khi nào gọi tới __notifyListeners()__, __ChangeNotifierProvider__ sẽ được thông báo và tất cả các widget bên trong __Consumers__ sẽ được rebuild.

## FutureProvider

- Về cơ bản, __FutureProvider__ chỉ là một wrapper với bên trong là một __FutureBuilder__. Ta cung cấp cho nó một số dữ liệu ban đầu để hiển thị trong giao diện người dùng và cũng có thể cung cấp cho nó một hoạt động bất đồng bộ __Future__ của giá trị mà ta muốn cung cấp. __FutureProvider__ sẽ lắng nghe khi __Future__ hoàn thành và sau đó thông báo cho __Consumer__ để xây dựng lại các widget của nó.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return FutureProvider<MyModel>( //                      <--- FutureProvider
      initialData: MyModel(someValue: 'default value'),
      create: (context) => someAsyncFunctionToGetMyModel(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('My App')),
          body: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[

              Container(
                padding: const EdgeInsets.all(20),
                color: Colors.green[200],
                child: Consumer<MyModel>( //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return ElevatedButton(
                      child: Text('Do something'),
                      onPressed: (){
                        myModel.doSomething();
                      },
                    );
                  },
                )
              ),

              Container(
                padding: const EdgeInsets.all(35),
                color: Colors.blue[200],
                child: Consumer<MyModel>( //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return Text(myModel.someValue);
                  },
                ),
              ),

            ],
          ),
        ),
      ),
    );
    
  }
}

Future<MyModel> someAsyncFunctionToGetMyModel() async { //  <--- async function
  await Future.delayed(Duration(seconds: 3));
  return MyModel(someValue: 'new data');
}

class MyModel { //                                               <--- MyModel
  MyModel({required this.someValue});
  String someValue = 'Hello';
  Future<void> doSomething() async {
    await Future.delayed(Duration(seconds: 2));
    someValue = 'Goodbye';
    print(someValue);
  }
}
```

- Trong đoạn code trên, ta sử dụng một Model để cung cấp dữ liệu ban đầu cho giao diện người dùng, sau đó chương trình sẽ trả về một Model với data mới sau 3 giây thông qua __someAsyncFunctionToGetMyModel()__.
- Giống như __Provider__, __FutureProvider__ không lắng nghe các thay đổi của Model. Action của button trong ví dụ sẽ chỉ thay đổi dữ liệu của model sau 2 giây và không có sự ảnh hưởng tới giao diện.

## StreamProvider

- Về cơ bản, StreamProvider chỉ là một wrapper với bên trong là một __StreamBuilder__. Ta cung cấp một __Stream__ và sau đó __Consumer__ được xây dựng lại khi có sự kiện trong __Stream__.
- Các giá trị được phát ra từ luồng là bất biến. Tức là __StreamProvider__ không lắng nghe những thay đổi của model mà chỉ lắng nghe các sự kiện mới trong __Stream__.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamProvider<MyModel>( //                       <--- StreamProvider
      initialData: MyModel(someValue: 'default value'),
      create: (context) => getStreamOfMyModel(),
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: Text('My App')),
          body: Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[

              Container(
                padding: const EdgeInsets.all(20),
                color: Colors.green[200],
                child: Consumer<MyModel>( //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return ElevatedButton(
                      child: Text('Do something'),
                      onPressed: (){
                        myModel.doSomething();
                      },
                    );
                  },
                )
              ),

              Container(
                padding: const EdgeInsets.all(35),
                color: Colors.blue[200],
                child: Consumer<MyModel>( //                    <--- Consumer
                  builder: (context, myModel, child) {
                    return Text(myModel.someValue);
                  },
                ),
              ),

            ],
          ),
        ),
      ),
    );
    
  }
}

Stream<MyModel> getStreamOfMyModel() { //                        <--- Stream
  return Stream<MyModel>.periodic(Duration(seconds: 1),
          (x) => MyModel(someValue: '$x'))
      .take(10);
}

class MyModel { //                                               <--- MyModel
  MyModel({required this.someValue});
  String someValue = 'Hello';
  void doSomething() {
    someValue = 'Goodbye';
    print(someValue);
  }
}
```

- Khi nhấn vào button để thay đổi dữ liệu của model, giao diện sẽ không được cập nhật. Giá trị hiển thị trên màn hình là giá trị được phát qua __Stream<MyModel>__.
- __StreamProvider__ yêu cầu __Consumer__ xây dựng lại giao diện sau khi có sự kiện mới được phát ra.

## ValueListenableProvider

- Nó giống như __ChangeNotifierProvider__ nhưng phức tạp hơn và không có sự gia tăng giá trị rõ ràng.
- Nếu ta có một __ValueNotifier__ như sau:

```dart
 class MyModel {
  ValueNotifier<String> someValue = ValueNotifier('Hello');
  void doSomething() {
    someValue.value = 'Goodbye';
  }
}
```

sau đó ta có thể lắng nghe những thay đổi trong đó với __ValueListenableProvider__. Tuy nhiên, nếu ta muốn gọi tới phương thức trên model từ giao diện, ta cần cung cấp một model. Do đó, ta có thể thấy __Provider__ cung cấp __MyModel__ cho __Consumer__, cái mà sẽ đưa __ValueNotifier__ trong __MyModel__ cho __ValueListenableProvider__.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return Provider<MyModel>(
      //                              <--- Provider
      create: (context) => MyModel(),
      child:
          Consumer<MyModel>(//                           <--- MyModel Consumer
              builder: (context, myModel, child) {
        return ValueListenableProvider<String>.value(
          // <--- ValueListenableProvider
          value: myModel.someValue,
          child: MaterialApp(
            home: Scaffold(
              appBar: AppBar(title: const Text('My App')),
              body: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Container(
                      padding: const EdgeInsets.all(20),
                      color: Colors.green[200],
                      child: Consumer<MyModel>(
                        //       <--- Consumer
                        builder: (context, myModel, child) {
                          return ElevatedButton(
                            child: const Text('Do something'),
                            onPressed: () {
                              myModel.doSomething();
                            },
                          );
                        },
                      )),
                  Container(
                    padding: const EdgeInsets.all(35),
                    color: Colors.blue[200],
                    child: Consumer<String>(
                      //           <--- String Consumer
                      builder: (context, myValue, child) {
                        return Text(myValue);
                      },
                    ),
                  ),
                ],
              ),
            ),
          ),
        );
      }),
    );
  }
}

class MyModel {
  //                                             <--- MyModel
  ValueNotifier<String> someValue =
      ValueNotifier('Hello'); // <--- ValueNotifier
  void doSomething() {
    someValue.value = 'Goodbye';
    print(someValue.value);
  }
}
```

- Khi nhấn button, text của button sẽ thay đổi vì __ValueListenableProvider__ lắng nghe sự thay đổi của data ở model.

## MultiProvider

- Khi cần cung cấp một model thứ hai, ta có thể lồng các provider với nhau, hoặc một cách ngắn gọn hơn là sử dụng __MultiProvider__.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      //                                     <--- MultiProvider
      providers: [
        ChangeNotifierProvider<MyModel>(create: (context) => MyModel()),
        ChangeNotifierProvider<AnotherModel>(
            create: (context) => AnotherModel()),
      ],
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('My App')),
          body: Column(
            children: <Widget>[
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Container(
                      padding: const EdgeInsets.all(20),
                      color: Colors.green[200],
                      child: Consumer<MyModel>(
                        //            <--- MyModel Consumer
                        builder: (context, myModel, child) {
                          return ElevatedButton(
                            child: const Text('Do something'),
                            onPressed: () {
                              // We have access to the model.
                              myModel.doSomething();
                            },
                          );
                        },
                      )),
                  Container(
                    padding: const EdgeInsets.all(35),
                    color: Colors.blue[200],
                    child: Consumer<MyModel>(
                      //              <--- MyModel Consumer
                      builder: (context, myModel, child) {
                        return Text(myModel.someValue);
                      },
                    ),
                  ),
                ],
              ),

              // SizedBox(height: 5),

              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Container(
                      padding: const EdgeInsets.all(20),
                      color: Colors.red[200],
                      child: Consumer<AnotherModel>(
                        //      <--- AnotherModel Consumer
                        builder: (context, myModel, child) {
                          return ElevatedButton(
                            child: const Text('Do something'),
                            onPressed: () {
                              myModel.doSomething();
                            },
                          );
                        },
                      )),
                  Container(
                    padding: const EdgeInsets.all(35),
                    color: Colors.yellow[200],
                    child: Consumer<AnotherModel>(
                      //        <--- AnotherModel Consumer
                      builder: (context, anotherModel, child) {
                        return Text('${anotherModel.someValue}');
                      },
                    ),
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class MyModel with ChangeNotifier {
  //                        <--- MyModel
  String someValue = 'Hello';
  void doSomething() {
    someValue = 'Goodbye';
    print(someValue);
    notifyListeners();
  }
}

class AnotherModel with ChangeNotifier {
  //                   <--- AnotherModel
  int someValue = 0;
  void doSomething() {
    someValue = 5;
    print(someValue);
    notifyListeners();
  }
}

```

- Khi nhấn vào các button, dữ liệu của model cũng như giao diện sẽ thay đổi vì sử dụng __ChangeNotifierProvider__.

## ProxyProvider

- Khi ta muốn cung cấp 2 model, tuy nhiên model này lại phụ thuộc vào model kia, ta có thể sử dụng __ProxyProvider__. __ProxyProvider__ lấy giá trị từ 1 provider và cho phép nó sử dụng trong provider kia.
- Ví dụ sử dụng:

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider<MyModel>(
      create: (context) => MyModel(),
    ),
    ProxyProvider<MyModel, AnotherModel>(
      update: (context, myModel, anotherModel) => AnotherModel(myModel),
    ),
  ],
)
```

- Một __ProxyProvider__ sẽ có hai loại model trong dấu ngoặc nhọn (<>), model thứ hai sẽ phụ thuộc vào model đầu tiên.

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      //                              <--- MultiProvider
      providers: [
        ChangeNotifierProvider<MyModel>(
          //               <--- ChangeNotifierProvider
          create: (context) => MyModel(),
        ),
        ProxyProvider<MyModel, AnotherModel>(
          //          <--- ProxyProvider
          update: (context, myModel, anotherModel) => AnotherModel(myModel),
        ),
      ],
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('My App')),
          body: Column(
            children: <Widget>[
              Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  Container(
                      padding: const EdgeInsets.all(20),
                      color: Colors.green[200],
                      child: Consumer<MyModel>(
                        //          <--- MyModel Consumer
                        builder: (context, myModel, child) {
                          return ElevatedButton(
                            child: const Text('Do something'),
                            onPressed: () {
                              myModel.doSomething('Goodbye');
                            },
                          );
                        },
                      )),
                  Container(
                    padding: const EdgeInsets.all(35),
                    color: Colors.blue[200],
                    child: Consumer<MyModel>(
                      //            <--- MyModel Consumer
                      builder: (context, myModel, child) {
                        return Text(myModel.someValue);
                      },
                    ),
                  ),
                ],
              ),
              Container(
                  padding: const EdgeInsets.all(20),
                  color: Colors.red[200],
                  child: Consumer<AnotherModel>(
                    //          <--- AnotherModel Consumer
                    builder: (context, anotherModel, child) {
                      return ElevatedButton(
                        child: const Text('Do something else'),
                        onPressed: () {
                          anotherModel.doSomethingElse();
                        },
                      );
                    },
                  )),
            ],
          ),
        ),
      ),
    );
  }
}

class MyModel with ChangeNotifier {
  //                       <--- MyModel
  String someValue = 'Hello';
  void doSomething(String value) {
    someValue = value;
    print(someValue);
    notifyListeners();
  }
}

class AnotherModel {
  //                                      <--- AnotherModel
  final MyModel _myModel;
  AnotherModel(this._myModel);
  void doSomethingElse() {
    _myModel.doSomething('See you later');
    print('doing something else');
  }
}

```

- Ở đây, khi nhấn button "Do something", text sẽ được thay đổi theo giá trị truyền vào vì __MyModel__ thông báo cho thành phần lắng nghe nó (__ChangeNotifierProvider__) và giao diện được rebuild.
- Còn khi ta nhấn nút "Do something else", __AnotherModel__ sẽ lấy __MyModel__ (đã được thêm bởi __ProxyProvider__) và thay đổi text theo giá trị truyền vào vì __MyModel__ thông báo cho thành phần lắng nghe của nó về thay đổi, nên giao diện được thay đổi. Nếu __AnotherModel__ có dữ liệu riêng thay đổi, giao diện sẽ không được cập nhật vì __ProxyProvider__ không lắng nghe các thay đổi. Nếu cần, ta sẽ phải sử dụng __ChangeNotifierProxyProvider__ cho việc đó.
